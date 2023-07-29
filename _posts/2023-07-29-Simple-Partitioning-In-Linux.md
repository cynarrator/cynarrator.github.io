---
Title: Check if an IP address is of a VPN/Proxy or not
date: 2023-7-29 07:49 PM
categories: [Technology]
tags: [linux]
---

## Introduction
Well, you may have always wanted to install Linux and saw that the process is quiet easy. All you do is shrink a volume on a drive, provide some answer about your name, computer name, password, etc and do the partitioning. Many Youtube videos just select automatic partitioning which works well for them to showcase a distro. But you may not be content with their decision or found that it causes difficulties. Also they may not be eager to explain all the options in the partition option. That is exactly what I want to explain today. This works for any Linux distros. We will not consider LVM or encrypted storage in this post. Will save them for later.

## Terminologies
* **Mount Point** - A place where your folder is going to be mounted. This concept will seem odd to you if you are from Windows background. Unlike Windows that uses drive letters for each drive, there is no such thing in Linux, instead, every point in the filesystem is one continous path. On Linux, you can mount to an empty folder. For example you may have a flash drive that you can mount to any empty folder to access its files

* **Primary Partition** Primary partition is as the name suggests, primary. They reside directly on top of the drive and server as a container for all the files. Think of them as dividing a full hard drive into multiple sections each storing data

* **Logical Partition** In MBR, there is a limit of only about 4 primary partitions. To overcome this, they decided to create a partition inside primary partition. Since MBR's limitation only holds for primary partition, we have to mark a primary partition as an **extended** partition and put in **logical** partitions that hold up our data. As UEFI partition scheme does not have the same limitation, you wont have to deal with it in UEFI systems

* **File System** This gets as complicated as you want to make it but right now, think of it this way, filesystem is a way your files are internally organized in a partition(primary or logical) and so every partition needs to have one filesystem. Different partitions in the same harddrive can have different file systems

* **Swap** Swap is basically an area of your harddrive that can act like RAM. But the OS does not directly use it. Because remember, Harddrives and SSDs are still quiet slower compared to the RAM. So your OS will instead use it to dump unnecessary contents from RAM to make room for more content in the RAM for more important task. And when those old contents are requested, they are loaded into RAM as needed. This means atleast the system will not crash in the event of memory running very low. One thing you may find odd is that Swap partition has no mountpoint. This is to be expected because you are not supposed to put any files in there, it is just for the OS to use when it needs to

## General rule of thumb
1: If you want to dualboot Windows with Linux or only want to install Linux, you will first install Windows before Linux.

2: If Linux is to be used as your main Operating system to store important files and real work, you atleast need a seperate ``/home`` partition

3: It is true that running out of memory is rare in Linux systems. So you may question the need of a swap partition. However, swap partition is also used in Hibernation. So if you think you will need to hibernate, I suggest having your swap partition slightly bigger than your RAM

4: Do not use NTFS or FAT32 for Linux partitions with exception of EFI partition

5: You do not need a seperate ``/boot`` partition unless you are doing LVM which is out of scope of this article. It is also not necessary to have bootable flags set to any of your partitions anymore. But if you want or are having issues, set bootable flag for your ``/`` mountpoint

## Do you use UEFI or Legacy BIOS
There are some slight difference between UEFI and Legacy BIOS installation, namely, legacy BIOS does not need a dedicated EFI partition. Also as a side note, make sure your OS installation drive was booted into UEFI mode and not Legacy mode when you want to install that OS into UEFI because that is a common mistake people make.

Bottom line, just ignore the entry for EFI if you are using Legacy BIOS(MBR)

## Recommendations
<table>
    <thead>
        <th>Mount Point</th>
        <th>Size</th>
        <th>Partition Name</th>
        <th>Filesystem</th>
    </thead>
    <tbody>
        <tr>
            <td>
                /
            </td>
            <td>
                25GB to 30GB
            </td>
            <td>
                Root Partition
            </td>
            <td>
                ext4
            </td>
        </tr>
        <tr>
            <td>
                
            </td>
            <td>
            Size of your RAM + 2% extra 
            </td>
            <td>
                Swap Partition
            </td>
            <td>
                swap
            </td>
        </tr>
        <tr>
            <td>
            /boot/efi
            </td>
            <td>
                512MB
            </td>
            <td>
                EFI Partiton
            </td>
            <td>
                FAT32
            </td>
        </tr>
        <tr>
            <td>
                /home
            </td>
            <td>
                Remaining Space
            </td>
            <td>
                Home Partition
            </td>
            <td>
                ext4
            </td>
        </tr>
    </tbody>
</table>