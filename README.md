# Unbricking
Information on how to unbrick inMusic brand products.

This tutorial requires linux, fastboot (yes the android utility), device-tree-compiler and thekikgen's mpcimg script.

## Step 1 - Setting up the system

Set up for this proceedure by installing fastboot and device-tree-compiler (using ubuntu)

````
apt-get install fastboot device-tree-compiler
````
next, head over to github.com and download / save this file:
https://github.com/TheKikGen/MPC-LiveXplore/blob/master/imgmaker/mpcimg 
you want to copy it to: /usr/bin/mpcimg

````
cp -Rp mpcimg /usr/bin/mpcimg
chmod a+x /usr/bin/mpcimg
````
(iirc python may need hashlib or something along those lines as well incase you have trouble with some crypto related python stuff)



## Step 2 - Extracting the rootfs image from MPC-2.X.X-Update.img
The next step is to extract the firmware into an unpacked blob, that's where thekikgen's mpcimg tool comes in handy in order to do so we use this command
(special note, the MPC-2.X.X-update.img file is actually a compressed file its not the raw image you need so this next step is vital or forget about it)

````
mpcimg extract MPC-2.X.X-Update.img Extracted.img
````

Alternatively you can create dd backups of your partitions for later restoration.
Now wait for it to finish, extracting and you are done step 2.



## Step 3 - put your unit into DFU mode
On engine os its usually holding down the left and right load buttons and powering on the unit.
On MPC ONE the combination to enter fastboot is:
  hold down FULL LEVEL, BROWSE and PAD BANK C. and then press power on.
On the Numark MIXSTREAM PRO the button combination to enter fastboot mode (upload mode) is: 
  Hold cue, saved loop, auto loop, and the roll pads on the left deck and then press power/



## Step 4 - unlock the system for writing using fastboot
This next command is vital to being able to write to the unit, if this doesn't work you cant restore the rootfs.

````
fastboot oem inmusic-unlock-magic-7de5fbc22b8c524e
````

Reply from fastboot:
````
...
OKAY [  0.000s]
finished. total time: 0.000s
````



## Step 5 flash the rootfs.img
Special note, be paitent here, this takes a couple seconds to transfer to your mpc unit.

````
fastboot flash rootfs rootfs_extracted.img
````

Reply from fastboot:
````
Code: Select all
target reported max download size of 1879048192 bytes
erasing 'rootfs'...
OKAY [  2.440s]
sending 'rootfs' (512000 KB)...
OKAY [ 59.194s]
writing 'rootfs'...
OKAY [ 19.450s]
finished. total time: 81.084s
````


## Step 6 reboot

````
fastboot reboot
````

Reply from fastboot:
````
rebooting...
finished. total time: 0.051s
root@chromebook:/home/ultros/mixstream-img#
````

## Example of the entire process

````
## TEST FASTBOOT
root@chromebook:/home/ultros/mixstream-img# fastboot flash rootfs rootfs_extracted.img
target reported max download size of 1879048192 bytes
erasing 'rootfs'...
FAILED (remote: permission denied)
finished. total time: 0.000s

## DO UNLOCK MAGIC
root@chromebook:/home/ultros/mpc-img# fastboot oem inmusic-unlock-magic-7de5fbc22b8c524e
...
OKAY [  0.000s]
finished. total time: 0.000s

## FLASH THE ROOTFS.IMG
root@chromebook:/home/ultros/mpc-img# fastboot flash rootfs rootfs.img
target reported max download size of 1879048192 bytes
erasing 'rootfs'...
OKAY [  2.440s]
sending 'rootfs' (512000 KB)...
OKAY [ 59.194s]
writing 'rootfs'...
OKAY [ 19.450s]
finished. total time: 81.084s

## REBOOT
root@chromebook:/home/ultros/mpc-img# fastboot reboot
rebooting...
finished. total time: 0.051s
root@chromebook:/home/ultros/mpc-img#
````

And voila! if all went well your machine should fire right back up into a healthy state of being.


## Special Tip

if you dd your partitions to backed up .img files you can easily restore those partitions later via fastboot if you mangle them
say you want to flash a custom bootsplash image, you can fastboot upload that to the "splash" partition by doing (keep in mind these are not png or jpeg files these are raw data)

````
oem inmusic-unlock-magic-7de5fbc22b8c524e
fastboot flash splash splash.img
fastboot reboot
````

## List of partitions by disk and by name on the AKAI MPC / Force

````
mmcblk1p1 uboot-spl       //DO NOT TOUCH THIS (DO NOT FASTBOOT FLASH THIS OR YOU COULD WRECK IT!)
mmcblk1p2 env             //DO NOT TOUCH THIS (DO NOT FASTBOOT FLASH THIS OR YOU COULD WRECK IT!)
mmcblk1p3 uboot           //DO NOT TOUCH THIS (DO NOT FASTBOOT FLASH THIS OR YOU COULD WRECK IT!)
mmcblk1p4 splash          //the splash image that shows up when your power on normally
mmcblk1p5 recoverysplash  //the splash image that shows up in dfu
mmcblk1p6 rootfs          //root
mmcblk1p7 content         //acvs-content
mmcblk1p8 data            //az01-internal
````

## List of partitions by disk and by name on the ENGINE OS DEVICES Denon / Numark.

````
mmcblk1p1 uboot-spl       //DO NOT TOUCH THIS (DO NOT FASTBOOT FLASH THIS OR YOU COULD WRECK IT!)
mmcblk1p2 env             //DO NOT TOUCH THIS (DO NOT FASTBOOT FLASH THIS OR YOU COULD WRECK IT!)
mmcblk1p3 uboot           //DO NOT TOUCH THIS (DO NOT FASTBOOT FLASH THIS OR YOU COULD WRECK IT!)
mmcblk1p4 splash          //the splash image that shows up when your power on normally
mmcblk1p5 recoverysplash  //the splash image that shows up in dfu
mmcblk1p6 rootfs          //root
mmcblk1p7 data            //az01-internal
````

## How to create back up of emmc partitions on MPC / Force

````
cd /media/mymemorystick

dd if=/dev/mmcblk1p1 of=uboot-spl.img
dd if=/dev/mmcblk1p2 of=env.img
dd if=/dev/mmcblk1p3 of=uboot.img
dd if=/dev/mmcblk1p4 of=splash.img
dd if=/dev/mmcblk1p5 of=recoverysplash.img
dd if=/dev/mmcblk1p6 of=rootfs.img
dd if=/dev/mmcblk1p7 of=content.img
dd if=/dev/mmcblk1p8 of=data.img
````

## How to create back up of emmc partitions on denon / numark

````
cd /media/mymemorystick

dd if=/dev/mmcblk1p1 of=uboot-spl.img
dd if=/dev/mmcblk1p2 of=env.img
dd if=/dev/mmcblk1p3 of=uboot.img
dd if=/dev/mmcblk1p4 of=splash.img
dd if=/dev/mmcblk1p5 of=recoverysplash.img
dd if=/dev/mmcblk1p6 of=rootfs.img
dd if=/dev/mmcblk1p7 of=data.img
````


