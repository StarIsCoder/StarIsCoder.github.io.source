---
title: Delete half of files randomly
date: 2019/02/01
categories: Shell
---

Marvel中的灭霸可以打个响指就灭掉一半的人，然后在网上看到有个开源库，是执行脚本随机删除一半的文件。
- https://github.com/hotvulcan/Thanos.sh

不过该库中用的是PowerShell Script，由于之前学过一点shell，这次就用shell试试，就当练习下shell的语法了。


用find命令列出路径下的所有文件，这个脚本是这样

```sh
#!/bin/sh
find . -type f
```

其中type属性可以传递以下的参数，表示需要找到什么类型的文件

-type c

       File is of type c:

       b      block (buffered) special

       c      character (unbuffered) special

       d      directory

       p      named pipe (FIFO)

       f      regular file

       l      symbolic link; this is never true if the -L option or the  -follow  option  is  in  effect,
              unless the symbolic link is broken.  If you want to search for symbolic links when -L is in
              effect, use -xtype.

       s      socket

       D      door (Solaris)

由于是随机删除一半，因此我们需要把`find`输出的内容砍掉一半并且打乱。打乱需要用到`shuf`指令，mac上这个命令默认不支持需要执行`brew install coreutils`安装下。

`shuf`可以传递`-n`的参数，表示最多输出的行数。

这时候指令变成了`find . -type f | shuf -n  $[总文件数/2]`。

还需要获取到总文件数，可以用`wc(Word count)`来统计。由于文件名是按行输出，因此我们需要使用`-l`参数来统计行数。

这时候脚本变成了
```sh
#!/bin/sh
count=`find . -type f | wc -l`
echo $[count/2]
find . -type f | shuf -n  $[count/2]
```

接下来只需要把`find`并且用`shuf`打乱后的结果放在`rm -rf`后面，需要用到`xargs`指令。

`xargs`表示可以把输出传递给`xargs`之后的指令。最后生成的脚本就是这样的，功能就是随机删除path路径下一半的文件。

```sh
#!/bin/sh
count=`find . -type f | wc -l`
echo $[count/2]
find [path] -type f | shuf -n  $[count/2] | xargs rm -rf
```


