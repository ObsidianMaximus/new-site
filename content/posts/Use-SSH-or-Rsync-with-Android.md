+++
title = 'Use SSH or Rsync With Android'
date = '2024-04-26T09:28:58+05:30'
tags = [ "android" , "SSH" , "rsync" , "guide" ]
showtoc = true
+++

Today, I wanted to send a file from my desktop to my android device. I thought of using rsync for this purpose, which is a tool that I use frequently to sync files between my desktop and my laptop.

I thought of giving it a try and came across [this great article](https://howtos.davidsebek.com/android-rsync-termux.html) that explained just this.

**NOTE: This guide requires both the computer and the Android device to be on the same wifi network.**

### The process to send files from PC to Android

1. Install rsync on termux using the command: **`apt install rsync`** and install openssh using the command: **`apt install openssh`**.
2. Then setup the internal storage of our Android device to be accessible by termux using the command: **`termux-setup-storage`**. Grant termux the permission to access storage when prompted.
3. Now, change the sshd config file to allow termux to use SSH port. Termux does not have the permission to use the default SSH port 22, so we will use 8022 port instead (can use any available port). For doing this, edit the ../usr/etc/ssh/sshd_config file by simply using the command: **`echo "Port 8022" >> ../usr/etc/ssh/sshd_config`**
4. Let's get the local ip address of our device by using the command: **`ifconfig`**. The ip address should be usually in the end and will be after "inet" . Or you can just go to Settings app and click on wifi and select the wifi you are connected to and look for the ipv4 address. In either of these 2 cases, the ip address should usually begin with **192.168.xx.xx** where x's are to be replaced by your ip address part.
5. Setup a password for the user by using the command: **`passwd`**.
6. Finally, we have to start the SSH daemon. To do this, enter this in termux: **`sshd`**.

#### For using SSH

1. Open terminal on your pc and type: **`ssh -p 8022 192.168.xx.xx`** [where the x's are replaced by your ipv4 address part].
2. You should now be able to use android terminal through your desktop terminal.

#### For using Rsync

1. Simply enter the command : **`rsync -e 'ssh -p 8022' <file to transfer from pc> 192.168.xx.xx:/data/data/com.termux/files/home/storage/`**


	Explanation of some stuff in the above commands :

	- The -e command is used to specify the remote shell to use. For more details, check the manpage of rsync.
	- The x's replace your ipv4 address parts.
	- **/data/data/com.termux/files/home/storage/** is the "usual" path of the main storage in our Android which the user can access, so we are sending the file from our desktop to our Android device's internal storage using this. You can specify some folder to send the file into directly by simply adding the folder after **storage/** in the command. If this shows an error message when you try to transfer files, just type: **`pwd`** in termux to get your current directory where you wish to send/receive the files and replace output of this command with the path in the step 1 command above after your phone's ip address.
	- This should work in vice-versa as well. Just add the path to copy from Android first and the path to copy to in your desktop after this.
***
### Note
- To avoid entering password all the time while accessing SSH/Rsync, generate a key pair on your pc using the command: **`ssh-keygen -t ed25519`** and then copy it to Android using the command: **`ssh-copy-id -p 8022 192.168.xx.xx`** in your desktop terminal. This will ask you password only once and after authenticating it, you can now just use SSH/Rsync without entering that password which we generated in the "Step 5" in the beginning.

- **REMEMBER** that the ip address of your android device can change from time to time due to ip addresses being dynamic when using wifi. Or they "may" change if the router is restarted. In any case, be sure to repeat the "Step 4" which we did in the beginning to find our local ip address.