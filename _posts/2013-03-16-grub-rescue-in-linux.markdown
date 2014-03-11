---
layout: post
title: "grub-rescue-in-linux"
date: 2013-03-16 16:42
comments: true
categories: 
---

### What is grub?

Of course you should know it if you have ever used linux operating system.
I will directly to describe you one of my experience to use grub rescue prompt to rescue my linux.

### What is happening...

One day I start my linux , it prompt

```
grub rescue>
```

Then I don't know what to do, then google it, then do as the following:
    
```
grub rescue> ls
grub rescue> ls (hd0,3)/boot
```

Then I know where my vmlinuz and initrd file locate.
Then I do the following:

```
grub rescue> set prefix=(hd0,2)/boot/grub
grub rescue> insmod(hd0,2)/boot/grub/linux.mod
grub rescue> insmod part_msdos (maybe not need)
grub rescue> insmod ext2
grub rescue> insmod gzip
grub rescue> set root=(hd0,2)
grub rescue> linux /boot/vmlinuz-3.0.0-*** root=/dev/sda3 ro
grub rescue> initrd /boot/initrd.img-3.0.0-***
```

Then boot system again:

```
grub rescue> boot
```

OK, that's all, I can start my linux system, into the system, I fix my grub
confis `/boot/grub/grub.cfg`.

因为平时喜欢瞎折腾，改改配置啥的，很有可能会改错导致启动不了，记得又一次改了XWindow的
什么配置，然后启动不了了，难道要重装？当然不必要，只要你有一个linux的Desktop系统
就行，也就是一个微型版的linux系统啦，当然你可以自己制作一个，也可以自己刻录一个，
我是随身携带一个gentoo的u盘系统，一个是喜欢折腾gentoo随时需要装gentoo玩玩，还有就是
linux系统一被我改错文件了可以用u启动起来然后再挂载磁盘然后再将改错的内容改回去,哈哈,一起来折腾*inux吧~
