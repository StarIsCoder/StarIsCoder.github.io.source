---
title: Qt first&last note
date: 2018/11/16
categories: C++
---

## 起因
为了验证底层库的api是否可用需要在各个平台写DemoApp，但是如果每个平台各写一个有点重复工作了，因此需要用Qt写跨平台的应用，所以这可能是第一篇也是最后一篇和Qt相关的note。

## 概述
Qt是一种跨平台的应用框架和工具集，常见的Android、ios、windows和mac都可以支持。也就是可以一次编码多次编译生成各个不同的可执行文件，当然肯定是有一些约束或者未兼容好的地方。

Qt业务逻辑用的C++，界面用的是QML（Qt Modeling Language），有点类似于JSON和CSS，并且支持JavaScript。

## QML
```QML
Button {
        id: dialButton
        objectName: "dialButtonObject"
        signal dial(string address)
        onClicked: dialButton.dial(dialNumberTextEdit.text);
        x: 23
        y: 26
        text: qsTr("Dial")
    }

    TextEdit {
        id: dialNumberTextEdit
        x: 141
        y: 36
        width: 80
        height: 20
        font.pixelSize: 12
        text: qsTr("")
    }

    Button {
        id: answerButton
        objectName: "answerButtonObject"
        signal answer()
        onClicked: answerButton.answer();
        x: 23
        y: 128
        text: qsTr("Answer")
    }
```
在QML中大致是用如上格式来设计，当然Qt也支持可视化的控件拖动工具。
- id:可以理解在qml中一个控件的句柄，例如我们希望获得TextEdit的值，那就可以用`dialNumberTextEdit.text`
- objectName:在C++中找到该控件的一个类似于id的东西。
- signal：定义一个信号，可用它的id来发送
```QML
dialButton.dial(dialNumberTextEdit.text);
```

## 交互
Qt中很重要的概念就是信号（signal）和槽（slot）。信号是发送方，槽是接收方，由于信号和槽才使得前后端交互。假设我们在界面上有个button，在c++需要监听这个button的点击事件。

1.在QML中控件的onClicked事件发送一个信号
```QML
Button {
        id: dialButton
        objectName: "dialButtonObject"
        signal dial(string address)
        //发送信号dial，参数为dialNumberTextEdit的text
        onClicked: dialButton.dial(dialNumberTextEdit.text);
        x: 23
        y: 26
        text: qsTr("Dial")
    }
``` 
1. 新建一个类集成QObject，并定义一个slot，名字不用和信号一样。
```c++
class DialButtonSlot:public QObject {
    Q_OBJECT
public slots:
    void dial(QString message) {
        qDebug() << message;
    }
public:
    DialButtonSlot();
};
```
3.将信号和槽连接起来

```c++
//使用component加载qml文件
QQmlComponent component(&engine,QUrl(QStringLiteral("qrc:/main.qml")));
//实例化ViewLoader
QObject *viewLoader = component.create();
//从ViewLoader中找到objectName为"dialButtonObject"的QObject
QObject *dialButton = viewLoader->findChild<QObject*>("dialButtonObject");
//实例化一个Slot
DialButtonSlot dialButtonSlot;
//sender:dialButton
//receiver:dialButtonSlot
QObject::connect(dialButton, SIGNAL(dial(QString)), &dialButtonSlot, SLOT(dial(QString)));
```
这样当button点击之后被定义为slot的dial方法会被调用，我们就可以对拿到的数据进行处理。

那么如果我们数据处理完了想更新ui怎么通知呢，同样用信号和槽的形式。

1. 头文件中定义一个signal
```c++
signals:
  void setTextField(QVariant text);
```
2. QML中定义一个函数签名一样的方法
```QML
function setTextField(text){
    console.log("setTextField: " + text)
}
```
3. 同样将信号和槽连接起来
```
QObject::connect(&handleTextField, SIGNAL(setTextField(QVariant)),window, SLOT(setTextField(QVariant)));
```
4. 在我们需要通知QML的地方发送信号即可
```c++
emit setTextField("666666");
```

## QQuickImageProvider
QQuickImageProvider是某种用来存取图片的类。具体使用
1. 新建一个类继承QQuickImageProvider，覆写requestImage方法。
```c++
class SnapshotImageProvider:public QQuickImageProvider
{
public:
    SnapshotImageProvider();
    QImage requestImage(const QString &id, QSize *size, const QSize& requestedSize) override;
    void setSnapshotObject(HandleSnapshotId *handle){
        snapshotObject = handle;
    }

private:
    QImage screenshot;
    HandleSnapshotId *snapshotObject;
    QQuickWindow *m_window;
};
```
```c++
QImage SnapshotImageProvider::requestImage(const QString &id, QSize *size, const QSize& requestedSize){
    if (id == "fetch") {
        qDebug() << "id :"<< snapshotObject->id;
        return screenshot;
    } else {
        screenshot = ........
    }

    return screenshot;
```
2. QML中给Imaged的source属性赋值为QQuickImageProvider。
```QML
Image {
    id: snapshotImage
    objectName: "imageObject"
    cache: false;
    x: 23
    y: 259
    width: 325
    height: 445
    source: "image://snapshotimageprovider/presentation"
}
```
3. 把QQuickImageProvider添加到QQmlApplicationEngine中，在添加的时候传入的第一个参数表示这个QQuickImageProvider的名字，在用的时候也必须用这个名字，不过不区分大小写。
```c++
engine.addImageProvider("snapshotimageprovider",imageProvider);
```
QML在加载这个source的时候会主动去调用requestImage方法，并且他的第一个参数就是source属性中表示QQuickImageProvider名字后面的子串，如上代码，那么在requestImage方法中传入的第一个参数就是`"presentation"`，可以根据这个id进行不同的图片处理。

如果想主动刷新那么只需要重新赋值source即可，但是需要先赋为空
```QML
snapshotImage.source = ""
snapshotImage.source = "image://snapshotimageprovider/presentation"
```

## In the end
当然这边只是Qt的冰山一角，不过用来做Demo的app是足矣了，不知道后续还有没有机会再用Qt开发，其实Qt也只是界面可以大部分跨平台开发，很多其他功能并不是很适用，例如Qt原生的AudioRecorder很多参数不支持set，像bitrate等等，到最后还是导入了mac自带的lib然后用宏区分平台来实现录音功能。因此如果只是简单界面的测试app用qt是可以的，如果涉及的模块较多就不太合适了。

参考资料：https://andrew-jones.com/blog/qml2-to-c---and-back-again-with-signals-and-slots/