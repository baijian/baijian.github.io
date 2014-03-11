---
layout: post
title: "First-day-work-on-SNDA"
date: 2013-07-15 18:30
comments: true
categories: 
---

Today is the first day to work in SNDA, and ofcourse the first thing is to reinstall linux 
system instead of windows which is always egg ache.

Ok, begin my reinstall process, I insert my Ubuntu LiveCD, and reboot then enter BIOS, modify 
boot order, then save and boot.


After a while, I will get a install view, but I just got a black screen. I have realized that it must
be a graphics problem, then I go to ubuntu for help,Then I find the solution:

When startup just press e, then you will be able to edit your boot options, then change 
```quiet splash``` to ```nomodeset``` , then you can install your ubuntu as normal.

Ok, after a while, my installation is done, another problem is appear, my screen has a lower resolution 
than normal, I feel egg ache, Then I view what graphics card my computer have, It is nvidia, Then I guess
it must be no nvidia driver, then I input ```apt-cache search nvidia``` , then it list a lot of softwares about nvidia,
then I input ```sudo apt-get install nvidia-current nvidia-settings``` , then reboot system, It now show normally.

After I have installed my linux system, then I should prepare my linux environment as I do it again and again before, 
So I create [Personal-Linux-Configs](https://github.com/baijian/config-files) repository to help me.

Some small points to remember:

1. ssh connect slow:
        
    change ```/etc/ssh_config``` to add ```GSSAPIAuthentication no```

2. run my tunnel.sh shell every 3 miniutes

    */3 * * * * /bin/bash /path/to/shell/script

3.  open ubuntu update center settings and close update prompt, just update system manually input

    ```$ sudo apt-get update```

    ```$ sudo apt-get upgrade```

With every thing aboove is done, then I install some application softwares and then begin my work on SNDA,
Hope a great begin.

