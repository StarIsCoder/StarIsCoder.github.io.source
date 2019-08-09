---
title: Python ImportError
date: 2019/8/9
categories: Python
---

在执行python项目的时候，出现了这样的报错

```
ImportError: No module named user_util
```

去check了一下代码，发现user_util确实在目录中，令人不解。

在google了之后才知道需要设置PYTHONPATH或者sys.path，执行如下指令就可以解决ImportError

```shell
export PYTHONPATH=`pwd`
```

在看了doc之后，原来是这样的，`sys.path`的类型是一个list，而编译器是在这个list中寻找import的module，如果这个list中找不到就会报出ImportError。而这个变量是从`PYTHONPATH`初始化来的，当然还有一些系统默认自带的路径。需要注意的是在执行某个文件的时候，会将这个文件的目录加入`sys.path`中，比如这个文件

```
/Users/star/PycharmProjects/Practice/osinfo.py
```

如果在`osinfo.py`中打印一下`sys.path`会得到如下输出

```
['/Users/star/PycharmProjects/Practice', '/Users/star/PycharmProjects/Practice', '/usr/local/Cellar/python/3.7.0/Frameworks/Python.framework/Versions/3.7/lib/python37.zip', '/usr/local/Cellar/python/3.7.0/Frameworks/Python.framework/Versions/3.7/lib/python3.7', '/usr/local/Cellar/python/3.7.0/Frameworks/Python.framework/Versions/3.7/lib/python3.7/lib-dynload', '/Users/star/PycharmProjects/Practice/venv/lib/python3.7/site-packages', '/Users/star/PycharmProjects/Practice/venv/lib/python3.7/site-packages/setuptools-39.1.0-py3.7.egg', '/Users/star/PycharmProjects/Practice/venv/lib/python3.7/site-packages/pip-10.0.1-py3.7.egg']

```

可以看到`sys.path[0]`和`sys.path[1]`表示的是该文件的当前目录，而其余都是python自带的目录。

因此不仅仅需要check module是否存在，还需要check`PYTHONPATH`变量。