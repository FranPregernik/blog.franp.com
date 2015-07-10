+++
date = "2015-07-09T10:11:11+02:00"
draft = true
title = "DOH! Rebooted without kernels installed"

+++

Ever wonder What happens when you reboot a linux OS after removing all the
installed kernels.

<!--more-->

## Paint the picture

I am currently at the seaside in my family cottage on an island typing away on
my work laptop after having successfully fixed it. The internet is slow and this
is the only PC here (maybe even in the neighborhood). I have an Ubuntu 15.04
laptop with an encrypted whole disk setup and yesterday I was informed that
there is a new kernel available to be installed. I am not afraid, a fresh backup
was available, so off we go.

## First issue

The installer created a separate boot partition of 250 MB and it had older
kernels installed so as it so happens apt-get complained that there is no more
space left. I bingled the problem and found the procedure to remove the older
kernels and verified I had enough free space.

I put the kids to sleep and went back to work. The laptop did not boot. Nothing
in GRUB except that BIOS setup thingy. I actually forgot to install the new
kernel before rebooting. DOH!

Horror set in ... work laptop, encrypted, unbootable, 3 days to end of vacation.
Time to investigate just how screwed I am.

* No way to download live ISO and burn it - no OS to download and no CD-ROM on laptop
* All the instructions call for the live boot, chroot, install kernel http://askubuntu.com/a/28100
* None of them mention how to fix this on an encrypted volume

## Solving the boot issue

How to download and boot from a live CD? Well I had my android phone with me.

Some sort of PXE boot app? Nothing worth mentioning and PXE linux setup is way above my patience level.

Could you use a android phone as a boot device? YES!!!!!! DriveDroid to the
rescue. This fantastic little piece of app can download and turn the phone into
a bootable USB/CDROM drive. Please note that I have a Galaxy S3 Mini and it
works. The app says some devices might not work.

I downloaded the Ubuntu desktop ISO and turned on the boot feature in the app.
Plugged int the USB cable and rebooted the laptop. Configure the BIOS to boot
from USB drive first.

## Installing the kernel

Please note that this will guide you to the solution but it might be different
on your system. I hacked my way to the solution.

To install the latest kernel you do the following in Ubuntu live.

### 1. Find the LUKS name

In the unity taskbar click on the disk icon with the little yellow thingy
(encrypted partition) and enter the password

You will get one or more new disks in the taskbar - figure out what your root
partition is and click on it to mount it. Write down the UUID of the decrypted
root partition somewhere.

### 2. Read crypttab

Find out what the /etc/crypttab file says. I've included mine below for reference

~~~
sda3_crypt UUID=6f7fe8dc-9034-48a9-9ce2-axxxxxxxxxa1 none luks,discard
~~~

Note the first part (i.e. sda3_crypt) and the UUID (i.e.
6f7fe8dc-9034-48a9-9ce2-axxxxxxxxxa1).

### 3. Cleanup before fix

Unmount the mounted volumes - I did it by rebooting into the live environment,
lazy... I know

### 4. Kernel installation

Open a terminal window and run

~~~bash
sudo cryptsetup luksOpen /dev/disk/by-uuid/6f7fe8dc-9034-48a9-9ce2-axxxxxxxxxa1 sda3_crypt
~~~

It will ask you for the partition password. Afterwards you will get new entries
in /dev/disk/by-uuid. I got one for the root partition and one for the swap.
Also there is the boot partition.

Mount the decrypted root partition

~~~bash
sudo mount /dev/disk/by-uuid/3d5f47c3-3f9f-4c3a-8b78-9d34e88958da /mnt
~~~

Mount the boot partition

~~~bash
sudo mount /dev/disk/by-uuid/90df322e-3d42-4200-89fb-d5e95540acdd /mnt/boot
~~~

Mount the special partitions

~~~bash
sudo mount --bind /dev  /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys  /mnt/sys
~~~

Chroot into the /mnt

~~~bash
sudo chroot /mnt
~~~

Install the Linux kernel (no sudo required as you are root after a chroot)

~~~bash
apt-get install linux-image-generic
~~~

After a successful installation of the kernel, get out the chroot and unmount some filesystems:

~~~bash
exit
sudo umount /mnt/sys
sudo umount /mnt/proc
sudo umount /mnt/dev
sudo umount /mnt/boot
sudo umount /mnt
~~~

If this last step fails, shutting down the live OS will unmount it for you.
