+++
date = '2024-04-24T21:53:31+05:30'
title = 'Boot Image Extraction Guide'
tags = [ "guide", "android" ]
+++

Guide on how to extract a boot image from any Android phone [Requires Magisk and Computer]

## Getting started!

Accessing and modifying system files on your device typically requires superuser permissions. To extract the boot image, we'll need a root environment. We'll achieve this by using Magisk.

#### Procedure

1. First, make sure you install TWRP and then flash Magisk through it.
2. Setup Magisk in your phone.
3. Connect your phone to your Computer and type adb shell [Ensure that you have adb setup properly].
4. Now, become root by typing **su** in the shell on your PC.
5. Copy/Paste the following code in the shell.

```bash
for PARTITION in "boot" "boot_a" "boot_b"; do
  BLOCK=$(find /dev/block \( -type b -o -type c -o -type l \) -iname "$PARTITION" -print -quit 2>/dev/null)
  if [ -n "$BLOCK" ]; then
    echo "$PARTITION" = $(readlink -f "$BLOCK")
  fi
done
```

This command will display the boot partition paths for both A/B and A-only devices.

**Note: On [A/B devices](https://source.android.com/docs/core/ota/ab), the loop command will display the boot partition paths for both slots, something like this!**

```bash
boot_a = /dev/block/sda40
boot_b = /dev/block/sda41
```
In this case, you can extract the image corresponding to your currently active slot.To determine the active slot, enter the command ```getprop ro.boot.slot_suffix```. If the output is _a, use the path for boot_a; otherwise, use the path for boot_b.

6. Finally, use the following command to extract the image from the specified boot path:

```bash
dd if=<boot_partition_path> of=<output_path>
```

For example:

```bash
dd if=/dev/block/mmcblk0p42 of=/sdcard/boot_a.img
```
I hope this did work for you, as it did indeed work for me!


CREDITS : [Here](https://gist.github.com/gitclone-url/a1f693b64d8f8701ec24477a2ccaab87)