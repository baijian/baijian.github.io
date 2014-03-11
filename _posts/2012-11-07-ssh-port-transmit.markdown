---
layout: post
title: "SSH_Port_Forwarding"
date: 2012-11-07 22:16
comments: true
categories: 
---

SSH Port Forwarding is also called SSH Tunneling

### 笔记本反qiang方法

$ ssh -DNf 1000 user@myvps.com

D -> 动态端口转发

N -> 不需要shell

f -> 后台执行

然后浏览器设置一个SOCKS带离(可以使用chrome-SwitchySharp)，做个过滤就可以上你想上的网站了.

### 手机反qiang方法

以前我在vps上架了个vpn服务，然后手机连上去就行了，后来换了服务器，也就懒得架
vpn了，然后上网找了个android的应用程序，可以通过通过上面类似的原理来反qiang.

<s>下载地址:<a href="/sshtunnel-1.8.1beta.apk">sshtunnel-1.8.1beta.apk</a></s>

* 原理下面这两个链接讲的很清楚^-^

<a target="_blank" href="https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/">https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/<a>

<a target="_blank" href="http://blog.jianingy.com/2009/09/ssh隧道技术简介/">SSH**技术简介</a>


