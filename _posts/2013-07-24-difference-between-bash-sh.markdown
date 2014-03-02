---
layout: post
title: "Deep-understand-Bash"
date: 2013-07-24 22:55
comments: true
categories: 
---

### Bash and sh

*** Basic concepts ***

Bash is Bourne again shell, ```ls -l /bin/bash```,then ```ls -l /bin/sh```,
then you will find, ```/bin/sh -> bash```, ```sh``` equals to ```bash --posix```,
now in Ubuntu System you will find ```/bin/sh -> dash```, ```dash``` is different
with ```bash```, it's for script executing, not for interaction, it's fast but
less functions than ```bash```, and it comply with POSIX.

<!-- more -->

*** Specify an interpreter for script ***

You write a script, you may write ```#!/bin/bash``` in first line,
the you run ```chmod +x script```, then run it, it will use bash 
interpreter to run it, or write ```/bin/bash script``` to specify a 
interpreter and ```#!/bin/bash``` will be no use.

*** Differences ***

要理解Bash和sh乃至Dash的区别,关键是要理解
Bash的POSIX Mode,我也不是很明白这个模式，因为我一般写shell脚本
直接声明的```#!/bin/bash```,等以后实践实践再补充补充使用体会吧.

### Type of shell

*** Interactive Shell***

```echo $-```,如果含有i,就是interactive shell,interactive shell都含有```PS1```
变量,所以也可以通过是否存在```PS1```这个变量来判断是否微interactive shell.

*** Non-Interactive Shell***

即与上面的解释向反,不含有i这个参数选项

*** Login ***

远程登录或者tty1-6登录的shell都是Logoin Shell.

*** Non-Login ***

像模拟终端的shell就是Non-Login Shell,比如terminal终端.

### Startup files of different type of shell

*** 登录的shell ***

*Bash*

当你登录一个交互的shell或者shell脚本执行时加上--login选项,也就是说是一个```login shell```,
配置读取```/etc/profile```这个配置文件,然后按下面的顺序再寻找配置文件
    1. ~/.bash_profile
    2. ~/.bash_login
    3. ~/.profile
寻找到第一个存在且可读的配置就不继续后面的寻找了.
其实可以看一下```/etc/profile```的内容,很有可能这个文件调用了```/etc/bash.bashrc```这个配置文件,
或者也调用了```/etc/profile.d/```下面的所有配置文件,再看看```~/.bash_profile```这个
文件,很有可能里面又调用了```~/.bashrc```这个配置文件.这些需要清晰一点，不然很容易混淆.

*sh*

首先读取并执行```/etc/profile```和```~/.profile```


*** 交互的非登录shell ***

*Bash* 

比如你在你的ubuntu系统里用terminal打开的终端shell就是交互的非登录shell,这种shell
会从```~/.bashrc```读取信息执行.

*sh*

寻找ENV环境变量的值命名的文件,读取并执行.

*** 非交互的非登录shell ***

*Bash*

也就是你常用的shell脚本,它会把环境变量```BASH_ENV```的值当作文件名,如果文件存在且可读就进行读取执行.

*sh*

不会读取任何startup文件.


** End **

At last Recommend a web address: [About Bash](http://www.gnu.org/software/bash/manual/bashref.html)
