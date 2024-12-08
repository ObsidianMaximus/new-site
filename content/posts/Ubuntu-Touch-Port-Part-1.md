+++
date = '2024-04-25T13:38:00+05:30'
title = 'Ubuntu Touch Port Part 1'
tags = [ "android", "ubuntu" ]
+++

## Getting Started

So I recently decided to get more involved in the Android world and came about a 'new' type of OS, known as Ubuntu Touch. I thought a bit on it and decided to start the porting process for it for my device Realme GT Neo 2 [codenamed:bitra].

I started with the [official documentation of UBports](https://docs.ubports.com/en/latest/porting/introduction/index.html) where in they gave some good amount of introduction on how and why certain things needed to be done.

The guide mentions the use of GSI for devices with Android 8+ and, as my device is indeed treble supported, I think I will head to using GSI only.

[Standalone Kernel Method (Halium 9 and newer)](https://docs.ubports.com/en/latest/porting/build_and_boot/standalone_kernel_build.html) is very interesting as it seems to be the least tiresome way for porting among all of the 3 mentioned methods. So I started with this and did the steps as mentioned in the guide.

The defconfig that i had to use is the [sm8250_defconfig](https://gitlab.com/Krishna-Yadav/android_kernel_realme_sm8250/-/blob/master/arch/arm64/configs/vendor/sm8250_defconfig?ref_type=heads). Next i made a halium.config file as prompted.


Next, i had to fill in the deviceinfo file. For this purpose, I used the [Poco X3 Pro device branch](https://gitlab.com/ubports/porting/community-ports/android11/xiaomi-poco-x3-pro/xiaomi-vayu) as a dummy template and started filling stuff according to it and the guide.

But I got stuck on the *deviceinfo_bootimg_prebuilt_dtb* step as i was not sure where to find my dtb. So i asked this question in the telegram group of Ubports and was recommended to run a script on the [boot.img which i had extracted](/bootimageextractionguide/index.html).

The script was by LOS. It can be found [here](https://raw.githubusercontent.com/LineageOS/android_system_tools_mkbootimg/lineage-18.1/unpack_bootimg.py) [use it through curl -O options]. And it worked! I was able to get the output from my boot.img using the script by running this command : **`python unpack_bootimg.py --boot_img boot.img`**

{{< figure align=center src="/img/pyScriptLOS18.png" caption="The script's output in normal format.">}}

But then, i was recommended to add the option **`--format mkbootimg`** along with the above command to get the outputed text in terms of mkbootimg arguments, as it would output the values that were expected for the **deviceinfo** file.

Unfortunately for me though, adding that option to the command gave me an error : **`error: unrecognized arguments: --format mkbootimg`** 

I then thought and checked the latest LOS branch from where i had got that previous script. And i did find an updated script from LOS21 that did have those options that i needed for mkbootimg format. The new script can be found [here](https://raw.githubusercontent.com/LineageOS/android_system_tools_mkbootimg/lineage-21.0/unpack_bootimg.py)

After this, i ran the command again, now with the additional parameters to it and was indeed able to generate the output in mkbootimg format without any errors! The updated command : **`python unpack_bootimg.py --boot_img boot.img --format mkbootimg`**

{{< figure align=center src="/img/pyScriptLOS21.png" caption="The script's output in mkbootimg format.">}}

This has given me alot to work with now. I will be looking forward to progress even further and complete the deviceinfo file.