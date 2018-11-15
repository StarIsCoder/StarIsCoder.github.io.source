---
title: 脚本
date: 2018/11/15
categories: OS
---

之前还在ODM的时候用的是Linux系统，关于环境配置有两个脚本文件经常修改，一个是bash_rc和/etc/profile，bash_rc是针对当前用户的，而/etc/profile是针对所有用户的。

不过这次的topic是关于macOS环境下的。相对于Linux，mac下并没有bash_rc，取而代之的是bash_profile。

bash_profile是在login shell(在填写username和password的时候启动的shell)启动的。至于bash_rc在mac上是没有的，如果习惯使用这个的话就需要在bash_profile中source ~/.bash_rc

```sh
if [ -r ~/.bashrc ]; then
   source ~/.bashrc
fi
```
或者这样写更短
```sh
[ -r ~/.bashrc ] && . ~/.bashrc
```

和Linux一样，mac下也有/etc/profile。
```sh
File /etc/profile

# System-wide .profile for sh(1)

if [ -x /usr/libexec/path_helper ]; then
        eval `/usr/libexec/path_helper -s`
fi

if [ "${BASH-no}" != "no" ]; then
        [ -r /etc/bashrc ] && . /etc/bashrc
fi
```
可以看到是去执行了/etc/bashrc
```sh
File /etc/bashrc

if [ -z "$PS1" ]; then
   return
fi

PS1='\h:\W \u\$ '
# Make bash check its window size after a process completes
shopt -s checkwinsize

[ -r "/etc/bashrc_$TERM_PROGRAM" ] && . "/etc/bashrc_$TERM_PROGRAM"
```
又去执行了bashrc_Apple_Terminal文件，而在这个文件中执行了bash_profile。

因此具体的执行顺序是
- /etc/profile
  - /etc/bashrc
    - /etc/bashrc_Apple_Terminal
- if it exists: ~/.bash_profile
  - when ~/.bash_profile does not exists, ~/.bash_login
  - when neither ~/.bash_profile nor ~/.bash_login exist, ~/.profile
- ~/bash_profile can optionally source ~/.bashrc

Tips:Oh-My-Zsh是mac中非常好用的针对Z-shell的框架，但是他不会自动执行bash_profile，需要在~/.zshrc中添加`[ -r ~/.bash_profile ] && . ~/.bash_profile`
