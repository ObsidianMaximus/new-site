+++
title = "Whatsapp Without Phone"
date = "2024-05-29T20:43:23+05:30"
tags = [ "VM", "guide", "android", "linux" ]
showtoc = true
+++

I had stopped using whatsapp because I didn't want to have it's proprietary app on my de-googled device.

### The Issue
But you see, this application is used by most of my academic professors and other colleages, so I had to somehow use it (without the app on my phone, of course).

So I thought of running it through some android emulator, but didn't want to natively run it on my linux machine. [No waydroid as I run X11]

Thus I installed Windows 10 through virtual box and then installed bluestacks in it. But bluestacks was not starting properly (issues with virtualization), which I solved via virtualbox settings.

{{< figure align=center src="/img/enable_virt.png" caption="Had to enable the above 2 options to get virtualization working.">}}

_If this still doesn't work for you, I think you should also enable Virtualization and Hyper-V support from "Turn on windows features" from within your Windows VM._


After this, bluestacks started working normally and I installed whatsapp in it and logged in to my account _[A strange thing was that all my chats and groups were gone, but maybe that was because I was logging in after around 1 month]._

My next major work was to get whatsapp web working, so that I could use it on my other devices (of course, not going to run the emulator each time I have to use it).

To do so, whatsapp requires web versions to have their QR code scanned by the whatsapp application for login, so I used a usb camera that I had with me, but I got this error when I tried to use the webcam in my Windows VM:

{{< figure align=center src="/img/cam_notFound_big.png">}}

 It required a "Webcam Passthrough", so that the VM instance could access the USB camera.
 
 ### Steps to allow Webcam Passthrough [[Credits](https://askubuntu.com/a/1237808)]:
 
 1. First install certain extra packages to get stuff working:
 
    ```bash
    sudo apt-get install virtualbox-guest-additions-iso virtualbox-ext-pack
    ```
   
2. Next, we need to find our webcam details using this:

    ```bash
    VBoxManage list webcams
    ```
    
    This should show us a list of (for me only 1) webcam/s attached to your host device.
    
3. Now finally we need to connect this webcam to enable passthrough. Issue this command, by replacing the word **Windows** with the name of your VM instance and the .1 with the number that corresponds to your webcam as listed in step 2 (in most cases, it should be .1):

    _**NOTE: This command must be run only when your windows VM is running.**_
    ```bash
    VBoxManage controlvm "Windows" webcam attach .1
    ```
    
{{< figure align=center src="/img/list_Webcam.png">}}

_NOTE: If you shutdown or restart your VM, it will loose access to the camera. You need to re-run the step 3 command to get it working._

### Conclusion
 
With this much, my VM was able to detect the webcam and without much difficulty, I was able to register with the whatsapp web version on my devices.

A small caveat though, remember to open the whatsapp in your windows VM every 14 days to avoid getting logged out of your whatsapp web devices.