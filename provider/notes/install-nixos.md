---
title: Installing NixOS
published: April 4, 2017
excerpt: Instructions/commands to install NixOS
tags: NixOS
toc: off
---

## Get NixOS on the USB stick

Formatting the USB stick:
1. Find out the device:
```bash
sudo fdisk -l
```
2. Unmount
```bash
sudo umount /dev/sd??
```
3. Format to FAT32
```bash
sudo mkdosfs -F 32 -I /dev/sd??
```
4. Detach and reattach
5. Copy NixOS on the USB
```bash
sudo dd bs=4M if=nixos.iso of=/dev/sd??/
```
