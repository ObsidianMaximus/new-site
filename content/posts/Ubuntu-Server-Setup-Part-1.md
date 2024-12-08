+++
title = 'Ubuntu Server Setup Part-1'
date = '2024-04-30T19:46:03+05:30'
tags = [ "server" , "ubuntu" ]
+++

I was wanting to learn on how to setup my servers and host stuff on them, but to host a server online, one needs to spend some amount of money on them, which i really didn't want to. But then i remembered that i have one old dusty home computer that had been lying in my home since several years, while not really being used anymore.

So i decided to use this opportunity to reuse the same computer for my server tasks and in this way, i will be saving up on those server costs that i would have to pay [electricity is cheap here].

But i had a hurdle in my path, and that was my old monitor not working, due to which i was unable to get my cpu to give a basic display as it had only VGA output. And well, i couldn't really use this PC with my current monitor, as this monitor supports only HDMI and Display Port. I did have one VGA-to-HDMI converter lying around with me, but that had malfunctioned as well.

This had me worried for some time before i thought of just using the gpu in my current setup on the older one. Sadly, this had another issue. The older desktop is an Intel Pentium E5400, and has a motherboard which is of LGA775 socket type. Now, this motherboard has the onboard power supply cables right above the PCIe slot, which rendered me placing my hefty gpu in that slot useless.

Then i had to think a bit more and got an idea of just taking out the HDD from the old PC and placing it in my working desktop [silly me]. I attached the HDD to my newer PC and started the machine.

At first, i went for the trusted server OS, Debian. Flashed it onto my usb drive and tried live booting only to be suprised with no display. Forgot that i had to enable CSM from BIOS and then turn on "legacy only" mode to allow the usb to boot [had partitioned it as MSDOS/MBR, silly me, again..]. This was again because the older computer had legacy BIOS only and i had to test the current HDD to ensure that it did boot alright under legacy conditions.

Adjusted boot options and booted the PC only to be met by a weirdly enlarged debian graphical installer. Pressed enter and the PC restarted and i was met with just blurred out lines and teared display with the text not legible at all. Tried rebooting several times from my usb drive but was met with the same issue again and again. Finally got quite frustrated and flashed ubuntu server on the drive.

Booted from it and to my surprise, it turned on smoothly. At first the screen was enlarged but when i progressed the installation, it auto-adjusted normally and matched my 24" screen resolution quite well. Installed the ubuntu server, which was pretty straight forward, thanks to the KIS ways of Ubuntu.

Then my next task was getting my wifi modem or should i say TP-Link USB Network Adapter (Archer T2U Plus) in working condition, as i planned to keep the pc in an isolated place. **The instructions for the same can be found [here](https://github.com/ObsidianMaximus/wifidrivers/blob/master/commands_for_drivers.sh).** This adapter uses a Realtek **RTL8821AU Chipset** at its heart, which fortunately has linux drivers available, thanks to aircrack-ng! 
Executing the commands in the script installed all it's drivers. Now next, i had to enable it and connect to my home network to make the pc accessible by my other devices.

To do this, here is an excellent guide from [Ubuntu](https://ubuntu.com/core/docs/networkmanager/configure-wifi-connections)!

As for the steps i followed, they were as follow:

- Firstly, i had to install **network-manager** as it was not present by default on the default server installation [ strange? ].
- Next, i ran **`nmcli d`** command to get the name of the wifi device.
- Then, to turn it on, issue: **`nmcli r wifi on`** and list all the scanned wifi networks with: **`nmcli d wifi list`**
- To connect to one of the networks listed, you have to type: **`nmcli d wifi connect <name of the wifi> password <password for that wifi>`**  [Replace stuff within < > with what you have got listed in the above step.]
- It took several seconds but after that it did get connected to my wifi network! I restarted my pc without the ethernet cable and the wifi did indeed work, although on startup, one of the checks done by systemd made me wait for quite a long time, more on that [here](https://askubuntu.com/questions/972215/a-start-job-is-running-for-wait-for-network-to-be-configured-ubuntu-server-17-1).


I then took out the HDD and inserted it into my older desktop and sadly, even after it being turned on for several minutes, i had no green led light show up on the wifi usb adapter [which is used to indicate that the modem is in use by the OS], thus, the older pc was _not_ turning on properly and i had no way to get to diagnosing the issue as i just didn't have a display [running it headless].

I think that I will have to indeed get my monitor repaired to continue on with my experimentations on my newly built home server. 