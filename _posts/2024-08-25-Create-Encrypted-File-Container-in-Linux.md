---
Title: Create an Encrypted file container in Linux
date: 2024-08-25 9:51 AM
categories: [Operating System]
tags: [linux, tip, privacy, security]
---
## Introduction

Most people recommend using [Veracrypt](https://veracrypt.fr/en/Home.html) to create an encrypted file container. While it is really a good software for this job, I wanted to explore how you can also create an encrypted file container in Linux without installing any third party software. Veracrypt in most cases also provide the same or similar features. However, this solution is native to Linux and as a result is much faster.

### Pros
- You do not need to install any third party software
- You can use any filesystem you want making it much more flexible
- It has many features you can add such as a recovery key or multiple key slots that allow multiple people to unlock and use the same volume with multiple passwords eliminating the need to share passwords
- It is open source, tried and tested in enterprise environment and secure solution
- You can name it anything and give it any extension
- You do not need to encrypt the whole drive

### Cons
- It is not cross platform, although some have been able to use it with WSL2 on Windows recently
- I have not found a way to change the size of the volume after creation. The only way I know is to create a new volume of different size, copy all the content from old volume then delete the old volume
- It takes the full size of the volume even if there is no content on it or it is not yet full
- LUKS header can take in from 2MB to 64MB depending on whether you are using LUKS1 or LUKS2 but you can place your header in a different place if you want

## Notes and Disclaimer
### When to run commands as superuser vs when not to
If you see a command prefixed with `sudo` you are supposed to run those as superuser. You may either choose to use `sudo` as described or use the actual `root` user.

### Choice of filesystem
In the examples below you will see me using **exFAT** filesystem. The reason is because I want it to be compatible with [EDS Lite](https://f-droid.org/packages/com.sovworks.edslite/) application on Android. However, remember that you can use any filesystem of your choice including **ext4**, **btrfs**, **xfs**, whatever you want. Just replace `mkfs.exfat` with `mkfs.ext4` in case of ext4 or `mkfs.fat -F32` in case of FAT32 before adding other flags.

If you are having issues with formatting exFAT filesystem below, you may need to install exFAT utilities depending on your Linux distribution. For Ubuntu, you may get them by running the command
```
sudo apt-get install exfat-fuse exfatprogs
```
For other distros, search up `exfat` in the repos and install all the relevant packages. For ext4 or other filesystems you may or may not need to install anything.

### Should you use LUKS 1 or LUKS 2
If you have a strict security requirement and have a large volume size to spare, you are absolutely recommended to use LUKS2. However, keep in mind that LUKS2 headers are quiet big, almost as big as 64MB. So when using LUKS2, make sure you have much more storage than 64MB. In order to use luks2, you are supposed to put in `--type luks2`, however even if you dont put in the `--type` flag it will by default select LUKS 2.

For most people, LUKS1 is fine. It also has very small header size of around 2MB. It also makes your volume compatible with EDS Lite application. This is what you will see me demonstrating in the examples below. To set luks to luks1 you have to put in `--type luks1` otherwise it will select luks2 by default.

### Things to rememeber with dd command
You need to be very careful when running dd command, be especially sure to double check `of=` value because if the file with that name in that path already exists, dd will just override it without prompting for warning. You may lose your files if you are not careful.

### Steps are pretty tedious
I wanted to explain everything manually. However all this can be automated with scripts. You can easily write a bash script to do all these in a single command.

## Steps to perform
### Outline of the process
- Create a volume with dd
- Format that volume with luks
- Open that luks volume
- Format that volume with the filesystem of your choice
- Mount that somewhere
- Use it to store files
- Unmount the volume
- Close the container

### Create a volume with dd
First, create a new volume using dd. Here, I will create a new volume of size 32MB. I will name the volume myluks.vol but you can name it whatever with whatever extension you want. I will be using LUKS1 for the reasons explained above in **Notes and Disclaimer** section
```
dd if=/dev/zero of=myluks.vol count=1 status=progress bs=32M
```
You can use the postfix `M` for MB and `G` for GB, etc. To create a 32GB volume I would have used `bs=32G` for example

**Again, make sure that file named myluks.vol does not exist already or it will be overwritten and data in it will be lost**

### Format that volume with luks
I use LUKS1 as described in **Notes and Disclaimer** Section
```
sudo cryptsetup luksFormat --type luks1 myluks.vol
```
Type in **YES** in all caps and provide a password. Make sure you provide a good password

### Open that volume
You can use any name to unlock that volume. This name is temporary and will be lost after you close it so you do not have to stress into giving it a good name. Any name that is descriptive enough for the purpose will work. Just do not use spaces in the middle of the name. When opening, you will be asked for the password you set previously. You may also be asked for root password before the luks password

In this case I will name it **myVolume**
```
sudo cryptsetup luksOpen myluks.vol myVolume
```
In this case, the opened volume is accessible at `/dev/mapper/myVolume`

If you used another name, ie. `myEncVol` then it would be accessible at `/dev/mapper/myEncVol`

#### What to do if you forgot the name of the volume after opening it
You can use the command
```
sudo lsblk
```
Then check the name under `loop0` section

### Format the opened volume
Once you opened the LUKS volume, you need to format it with a filesystem to be able to use it. For that, you need to know the name of the opened volume from the previous step. I used the name **myVolume** so I will use `/dev/mapper/myVolume` as a device to format. I have also decided to use **exFAT** as described in **Notes and Disclaimer** section onto `/mnt`
```
sudo mount -t exfat -o uid=$(id -u),gid=$(id -g) /dev/mapper/myVolume /mnt
```
Here, I have mounted that volume in `/mnt` and given it a permission of the current user to do whatever that user wants

### Use that volume to store sensitive files
The file is mounted in `/mnt` so that means you can open up you file manager, navigate to `/mnt` and start operating on the files there normally. Everything in `/mnt` in this case will be stored inside that volume you created

### Unmount the volume
After you are done, you can unmount the volume before closing the container
```
sudo umount /mnt
```
### Close the volume
After you are done, you should close the access to the container via its name you used when opening
```
sudo cryptsetup luksClose myVolume
```

## Normal usage
You are not always going to follow all these steps when using the volume, you can just 
- Open the LUKS volume
- Mount the opened volume from /dev/mapper/...
- Start using it
- Unmount the opened volume
- Close the LUKS container

## Conclusion
Even though the steps are quiet cumbersome at first, remember that you can script most of it to make it easier to interact everyday.