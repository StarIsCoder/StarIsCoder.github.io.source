---
title: Serializable
date: 2018/9/8
categories: Java
---

## 原因
由于对象在程序结束的时候会被销毁，因此如果希望对象能够在程序不运行的时候仍然能保存信息，那么就需要用到对象的序列化操作，简单来说就是将对象转换成字节序列，等需要的时候再反序列化即可读取其中的信息。又或者在Android开发中通过intent传递对象的时候，需要将对象序列化然后才能传递。

## 使用方法
```java
//写入文件
ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("test.obj"));
objectOutputStream.writeObject(new Data(1));
objectOutputStream.close();
//读取文件
ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("test.obj"));
Data data = (Data) objectInputStream.readObject();
System.out.println(data);
```
## transient关键字
如果某个类的属性不希望被序列化则可以加上transient关键字。
```java
transient  private String str;
```
当在反序列化读取的时候就会为null或者是类型的默认值。

## 序列化中的static
如果在序列化之后修改了static类型的变量，那么打印出来会如何呢
```java
class Data implements Serializable {
    private int n;
    transient public static int i = 5;

    public Data(int n) {
        this.n = n;
    }
    @Override
    public String toString() {
        return "Data{" +
                "n=" + n +
                '}';
    }
}
public static void main(String[] args) {
    try {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("test.obj"));
        objectOutputStream.writeObject(new Data(1));
        objectOutputStream.close();

        Data.i = 10;
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("test.obj"));
        Data data = (Data) objectInputStream.readObject();
        System.out.println(data);

    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
结果是10，原因是static是针对类的属性，而不是针对对象的属性。因为静态变量可以直接使用，所以序列化并不保存静态变量。

## Externalizable
Externalizable是一个继承Serializable的接口，并增加了两个方法
```java
public interface Externalizable extends Serializable {
    void writeExternal(ObjectOutput var1) throws IOException;
    void readExternal(ObjectInput var1) throws IOException, ClassNotFoundException;
}
```
这两个放在分别会在write和read的时候调用，我们可以实现这个接口，如果有其他操作可以放在这两个方法中，并且他会readExternal之前调用该类的无参构造方法。
```java
class Data implements Externalizable {
    private int n;

    public Data(int n) {
        this.n = n;
    }

    public Data() {
        System.out.println("no pm constructor");
    }

    @Override
    public String toString() {
        return "Data{" +
                "n=" + n +
                '}';
    }

    @Override
    public void writeExternal(ObjectOutput objectOutput) throws IOException {
        System.out.println("writeExternal()");
    }

    @Override
    public void readExternal(ObjectInput objectInput) throws IOException, ClassNotFoundException {
        System.out.println("readExternal()");
    }
}
public static void main(String[] args) {
    try {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("test.obj"));
        objectOutputStream.writeObject(new Data(1));
        objectOutputStream.close();

        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("test.obj"));
        Data data = (Data) objectInputStream.readObject();
        System.out.println(data);

    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
```
Result:
writeExternal()
no pm constructor
readExternal()
Data{n=0}
```
奇怪的是打印出的n居然是零，但是我们写进去的是一个n=1的对象。原因是Externalizable调用了无参的构造方法重新赋值了一遍。

这也是Serializable和Externalizable最大的区别：Serializable是完全根据二级制文件来构造对象，但是Externalizable是根据构造方法和readExternal方法来构造的。

因此正确的做法需要在writeExternal和readExternal的时候写入和赋值。这样的好处是更容易控制变量的存亡。

```java
@Override
public void writeExternal(ObjectOutput objectOutput) throws IOException {
    System.out.println("writeExternal()");
    objectOutput.writeInt(n);
}

@Override
public void readExternal(ObjectInput objectInput) throws IOException, ClassNotFoundException {
    System.out.println("readExternal()");
    n = objectInput.readInt();
}
```
## Externalizable的替代方法
如果觉得Externalizable这样太麻烦的话，我们依然可以使用Serializable，不过需要添加两个方法。并遵循它们的特征签名
```java
private void writeObject(ObjectOutputStream stream) {
        System.out.println(Thread.currentThread().getStackTrace()[1].getMethodName());
}

private void readObject(ObjectInputStream stream) {
        System.out.println(Thread.currentThread().getStackTrace()[1].getMethodName());
}
```
查看下源码就知道如果我们实现了这两个方法（虽然只是添加，就当他是实现吧），那么就不会走正常的序列化流程，而转为使用我们自己实现的。
```java
void invokeWriteObject(Object var1, ObjectOutputStream var2) throws IOException, UnsupportedOperationException {
    this.requireInitialized();
    if (this.writeObjectMethod != null) {
        try {
            this.writeObjectMethod.invoke(var1, var2);
        } catch (InvocationTargetException var5) {
            Throwable var4 = var5.getTargetException();
            if (var4 instanceof IOException) {
                throw (IOException)var4;
            }

            throwMiscException(var4);
        } catch (IllegalAccessException var6) {
            throw new InternalError(var6);
        }

    } else {
        throw new UnsupportedOperationException();
    }
}
```
如果我们自己实现的话实现方式和Externalizable一样
```java
private void writeObject(ObjectOutputStream stream) throws IOException {
    stream.writeInt(n);
}

    private void readObject(ObjectInputStream stream) throws IOException {
    n = stream.readInt();
}
```



序列化个人认为没有特别深入的必要，只要知道使用方法即可。