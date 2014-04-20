---
layout: post
title: "root-my-nexus5"
date: 2014-04-19 20:15:47
comments: true
categories: 
---

Today I prepare to root my nexus5 smartphone, so I search google and get a lot of usefull information from
[xda-forum](http://forum.xda-developers.com/) eventually.

After you root your android smartphone, you can easily remove the system applications, you can install
custom ROMS which means various android versions, and you can do almost everything on your android
smartphone, because you have gained the entire control of your phone.(But必须要说明的是如果购买的是国行
的android手机,root后系统将不会自动更新,也不会联保了,因为控制权已经完全到自己手上了,当然一般情况下我不买国行手机^-^)

Several days ago, my nexus5 come to me, and after experience it for a few days, I decide to root it. I find some ways
in the Internet, I do not like fool-method, I want to do it step by step, So after asking G-Teacher, I do as follows:

Install Official Android SDK from Google, Run SDK Manager(`./android-sdk/tools/android`) to install required components, based on you
network status maybe you need a proxy, then connect my nexus5 to my mac and I should open ***Developer options***, and it is in ***Settings*** above
***About Phone*** menu, if you do not see it, just enter ***About Phone*** and find ***Build number*** menu just touch it for 6 times, then it will
open ***Developer options*** menu, then enter it find ***USE debugging*** menu and touch it.

OK, after do all of the preparation above, you can enter below commands step by step:

1. `/path/to/your/sdk/android-sdk/platform-tools/adb devices`

    After execute it, as normal you will see:

    ```
    List of devices attached
    030fb19e0911b2b4    device
    ```

2. `/path/to/your/sdk/android-sdk/platform-tools/adb reboot bootloader`

    Reboot phone into bootloader, and your will see it on the phone.

3. `/path/to/your/sdk/android-sdk/platform-tools/fastboot oem unlock`

    This will asked to confirm the unlock on the phone's screen by select **YES** with volume buttons and
    confirm with the power button. And **attention** *unlock the bootloader will factory reset the phone*.
    Next I need to flash the recovery and root.You can use *TWRP* or *CWM* recovery, I choose *TWRP*, and download
    [TWRP recovery](http://techerrata.com/browse/twrp2/hammerhead) to my platform-tools directory.

4. `/path/to/your/sdk/android-sdk/platform-tools/fastboot flash recovery openrecovery-twrp-2.7.0.0-hammerhead.img`

    This command will send the custom recovery to your phone. After sending finished, just use volume toggles to select
    into **Recovery Mode**. Tap on **Reboot** then **System**, if normal, TWRP will ask to install root packages and go ahead
    and after rebooting, phone will prompt to install the SuperSU app, after that, you'are rooted.

    But(maybe you do not want see it at this moment), I meet a error when reboot the system. The error is about *unable to mount /data*,
    Maybe it's permissions problems, I enter the **Recovery Mode** again and select *fix permissions*, after fix the problem also exists.
    Then I use *Power* and *Volume -* buttons to reboot bootloader, and use `adb shell` to enter system shell, then use
    `e2fsck -fv /dev/block/platform/msm_sdcc.1/by-name/userdata`, but report *Bad magick number in super-block...*. Then I will download
    [Philz recovery](http://d-h.st/users/rootsu/?fld_id=34576#files) and use it to wipe my /data partition and remount it.

5. `/path/to/your/sdk/android-sdk/platform-tools/fastboot flash recovery philz_touch_6.26.6-hammerhead-recovery.img`

    This will flash philz recovery to the phone, and then boot to Philz recovery and wipe your error partition(mine is /data).
    After that flash TWRP recovery back and boot to it to continue.

6. `/path/to/your/sdk/android-sdk/platform-tools/fastboot flash recovery openrecovery-twrp-2.7.0.0-hammerhead.img`
   
    After flash TWRP recovery back, continue **step 4** to finish your root process.

OK, my nexus5 have been rooted, and my phone also been factory reset, but I use Google Account to sync all my data, so I resotre my phone data
quickly(based on your network speed), that's why I like **Google**, at last I recommend a very good android software for *geek*, 
see [here](http://rarnu.7thgen.info/), ofcourse after you root your device, install it and then enjoy your android tour.
