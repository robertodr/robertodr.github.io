---
layout: single
title: Installing NixOS
excerpt: Instructions/commands to install NixOS
tags: NixOS
---

{% include toc.html %}

## Get NixOS on the USB stick

Formatting the USB stick:
1. Find out the device:

~~~ bash
fdisk -l
~~~
2. Unmount

~~~ bash
sudo umount /dev/sd??
~~~
3. Format to FAT32

~~~ bash
sudo mkdosfs -F 32 -I /dev/sd??
~~~
4. Detach and reattach
5. Copy NixOS on the USB

~~~ bash
sudo dd bs=4M if=nixos.iso of=/dev/sd??/
~~~

## Install NixOS

My laptop is a [Lenovo Thinkpad T440s](http://www3.lenovo.com/us/en/laptops/thinkpad/t-series/t440s/)
and I've followed (mostly) the set of instructions on [Chris Martin's blog](https://chris-martin.org/2015/installing-nixos).

### Booting and partitioning

First of all, I had to switch to only using UEFI in the boot menu and then boot from the USB stick.
I wanted to wipe the whole disk and reinstall NixOS on an encrypted partition.
The NixOS installer does not (yet!) have a graphical user interface for
partitioning, but what I wanted to do is fairly straightforward to achieve
using `fdisk`. I followed the advice posted on [StackOverflow](https://unix.stackexchange.com/a/190145) [^1]
and ended up creating the following partition table:

~~~ bash
Disk /dev/sda: 238.5 GiB, 256060514304 bytes, 500118192 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: C57FCAC2-37B1-4E17-B70F-D1BE189BB48F

Device       Start       End   Sectors  Size Type
/dev/sda1     2048      4047      2000 1000K BIOS boot
/dev/sda2     4096   1028095   1024000  500M EFI System
/dev/sda3  1028096 500118158 499090063  238G Linux LVM
~~~
The [LVM](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)) device
will contain the OS and will be partitioned into the root folder `/` and the swap.

### Setting up the encryption, LVM groups and volumes

Let's move on to encrypting the device where the OS will actually live, _i.e._ `/dev/sda3`.
I am here following the instructions reported on the NixOS installation instructions.
The first command encrypts `/dev/sda3` and will prompt you to insert a
password, the second command opens the encrypted partition:

~~~ bash
cryptsetup luksFormat /dev/sda3
cryptsetup luksOpen /dev/sda3 enc-pv
~~~
OK, great! Time to make space for `/` and the swap:

~~~ bash
pvcreate /dev/mapper/enc-pv
vgcreate vg /dev/mapper/enc-pv
lvcreate -n swap vg -L 8G
lvcreate -n root vg -l 100%FREE
~~~
and to format:

~~~ bash
mkfs.vfat -n BOOT /dev/sda2
mkfs.ext4 -L root /dev/vg/root
mkswap -L swap /dev/vg/swap
~~~
At this point, launching `fdisk -l` you will see the following output (or at least I do!):

~~~ bash
Disk /dev/sda: 238.5 GiB, 256060514304 bytes, 500118192 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: C57FCAC2-37B1-4E17-B70F-D1BE189BB48F

Device       Start       End   Sectors  Size Type
/dev/sda1     2048      4047      2000 1000K BIOS boot
/dev/sda2     4096   1028095   1024000  500M EFI System
/dev/sda3  1028096 500118158 499090063  238G Linux LVM


Disk /dev/mapper/root: 238 GiB, 255532015104 bytes, 499085967 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/vg-swap: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/vg-root: 230 GiB, 246939648000 bytes, 482304000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
~~~

### Get the installation going!

Let's mount the disks and volumes to `/mnt`:

~~~ bash
mount /dev/vg/root /mnt
mkdir /mnt/boot
mount /dev/sda2 /mnt/boot
~~~
and activate the swap:

~~~ bash
swapon /dev/vg/swap
~~~

NixOS _declaratively_ defines the configuration of the system.
The starting point is to run the following:

~~~ bash
nixos-generate-config --root /mnt
~~~
that generates `hardware-configuration.nix` (**do not touch**) and
`configuration.nix` in the directory `/mnt/etc/nixos`.
My `configuration.nix` is available on
[GitHub](https://github.com/robertodr/nixos-configuration) and I can just clone
the repo with and populate the `/mnt/etc/nixos` directory with the contents of
the repo. This will not only configure the system as I like, but also install
all the software that I need in one go.
Once the `configuration.nix` is all set run:

~~~ bash
nixos-install
~~~
and reboot when the process is done.

### First login

Login as root user when the system reboots and add the 17.03 channel: 

~~~ bash
nix-channel --add http://nixos.org/channels/nixos-17.03
nix-channel --update
~~~
Add whatever you need to `configuration.nix` and rebuild your system:

~~~ bash
nixos-reduild switch
~~~

Moreover, the user that was created in the `configuration.nix` file does not have a password set! Let's do it now:

~~~ bash
passwd roberto
~~~
You can now login as a normal user, instead as root.
The system is now ready for use!


[^1]: Thanks to [Radovan Bast](bast.fr) for the tip.
