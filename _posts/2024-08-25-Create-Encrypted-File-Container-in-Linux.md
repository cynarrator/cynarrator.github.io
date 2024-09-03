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

  

- It takes the full size of the volume even if there is no content on it or it is not yet full. This can be overcome by using a `sparse` volume but not all filesystems and platforms support it

  

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


### KiB/MiB/GiB vs KB/MB/GB
When using `dd` command, you can either supply file size in `KiB` or `KB`. For example, to define a volume size of 4 Gigs you can pass in as `4G` which defines `4GiB` or `4GB` which defines `4GB`. File sizes are often defined in `KiB` or `MiB` to avoid confusion between whether `1MB = 1024KB` or `1MB = 1000KB`. It is intuitive for us to think of it in terms of multiples of `1024` so we will stick to using `K` for Kilobyte and `M` for megabyte, etc.



### Should you use sparse file or not

Sparse files only take up space that it actually needs. This is very useful because the filesize of the container is reduced when it is not full. However, you should keep in mind that not all filesystems support them. It is reported not to work in Apple HFS+ filesystems and the support varies between filesystems. It is however reported to work with NTFS filesystem as well as most Linux native filesystems like ext4, xfs and so on. Learn more in the [Arch Wiki](https://wiki.archlinux.org/title/Sparse_file)

You can create sparse files using either `dd` or `truncate` commands. I only demonstrate how to use `truncate` command to avoid confusion but you will find how to use `dd` in the linked ArchWiki article. Say for example you wanted to create a volume named `myvolume` that is of size 3 GiB, ie. the maximum size this volume can hold is 3GiB, you would use the truncate command as so,

```
truncate -s 3G myvolume
```

Here you can postfix `M` to denote size in MB and `G` to denote size in GB. For example, to create a file size 512M, you would specify the size as `512M`
  

  

### Things to rememeber with dd and truncate command

  

You need to be very careful when running dd command as well as truncate command, be especially sure to double check `of=` value in case of dd because if the file with that name in that path already exists, dd will just override it without prompting for warning. You may lose your files if you are not careful. Same is true with `truncate`

  

  
## Get the scripts to do it
The whole process is quiet tedious without using scripts. That is why I have also prepared some scripts you can use to quickly get it working. You can get those scripts from [my gitlab](https://gitlab.com/cy_narrator/lukshelper)
  

## Set it up manually
If you want to do everything manually, perhaps you want to understand every step clearly or want to modify the steps below for your needs than follow along this post

  

### Outline of the process

  

- Create a volume with dd

  

- Format that volume with luks

  

- Open that luks volume

  

- Format that volume with the filesystem of your choice

  

- Mount that somewhere

  

- Use it to store files

  

- Unmount the volume

  

- Close the container

- Cleanup in case you made extra directories for mounting

  

  

### Create a volume with dd

  

First, create a new volume using dd. You can either use a sparse file as described in **Notes and Disclaimer** section or a normal file. In case you decide to use a sparse file, you do not need to do any of the below, just create a volume using `truncate` and move to the next step.

For this example, I am creating a normal non-sparse volume of size `32 MiB` named `myluks.vol`. Then,

- Determine the block size you wish to use

    Some common block sizes are `4K`, `8K`, `16K`, `32K`, `64K`, `128K`, `512K`, `1M` and `4M`. This determines how many blocks are read/written in a single read/write operation. If you choose a very large block size, more data gets written at a time taking more memory but taking less processing power and disk usage. This is good if you have huge files but it wastes memory if you have very small files. There is no wrong answer and it is not necessary to choose this super accurately. This is mostly about performance and memory usage. Most people tend to use either `1M` or `4M` which is totally fine. For demonstration, I will choose `4K` block size because I am going to set it's volume size to `32M` later on where one is not expected to store huge files. This is different from filesystem block size also set later

- Determine the size of the volume you wish to create

    This is the total size of the volume you wish to create. It can be of any size you want. For this demonstration, I will pick 32 MiB volume. That will be the size I want for my volume.

    It is recommended that you choose a volume size that is a power of 2 such as `2`, `4`, `8`, `16`, `32`, `64`, `128`, `256`, `512`,...

- Convert the volume size to the size unit used by block size

    I chose `32MiB` volume size and my block size is `4KiB`. My Block size is in KiB and my volume size is in MiB, meaning I will multiply `32` by `1024` which will result in `32768 KiB`

    If I had chosen volume size of `32GiB` I would instead have multiplied **32 \* 1024 \* 1024** to convert `32GiB` into `33554432KiB`

- Divide the resulting volume size by block size

    Now that they use the same storage unit, we can divide them to get count. Since my volume size is `32768 KiB` and my block size is `4 KiB`, I can divide `32768 KiB` by `4KiB` to get `8192` which is the resulting `count`.

    If your `count` is not a whole number, ie. it contains decimal point, you would want to choose a different block size that perfectly divides your volume size. A good idea is to either use `1M` or `1K` for `block` size. This is because any number is divisible by 1. But any other block size that perfectly divides your volume size is okay. In this case you would repeat the calculation again but with a different block size


- Determine the name of the volume

    This can be any name as long as it does not contain spaces. Make sure this file does not already exists or it will be overwritten without any warning resulting in dataloss. For demonstration purpose I will name my volume `myluks.vol`. You can give any extension or no extension if you prefer

Then you would run this command

```
dd if=/dev/zero of=myluks.vol bs=4k count=8192 status=progress
```


**Again, make sure that file named myluks.vol does not exist already or it will be overwritten and data in it will be lost**

  

  

### Format that volume with luks

  

I use LUKS1 as described in **Notes and Disclaimer** Section

  

```
sudo cryptsetup luksFormat --type luks1 myluks.vol
```

  

Type in **YES** in all caps and provide a password. Make sure you provide a good password.

LUKS1 takes up `2M` Header size whereas LUKS2 takes up upto `64M`. If you wish to use LUKS2, make sure you have enough space for the header and data you wish to store. Ideally, much higher than `64M`

  

  

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

  

Then check the name under `loop` section

  

  

### Format the opened volume

  

Once you opened the LUKS volume, you need to format it with a filesystem to be able to use it. For that, you need to know the name of the opened volume from the previous step. I used the name **myVolume** so I will use `/dev/mapper/myVolume` as a device to format. I have also decided to use **exFAT** as described in **Notes and Disclaimer**

It is recommended to use the same block size/cluster size that you used in `dd` as you used in creation of the volume when formatting it for optimal performance. It is not necessary however and things will still work normally even when the block sizes are not aligned. Unfortunately, the flags to provide block size for a filesystem differs from one filesystem to the next so you will have to read the man page to figure that out. In `exfat` we use the `-s` option to specify block size in bytes. Since we chose `4K` for the block size, when converted to bytes by multiplying with `1024` we get `4096` so we use

```
sudo mkfs.exfat -s 4096 /dev/mapper/myVolume
```

Also keep in mind that filesystems also take up header space on top of LUKS. Filesystem header space depends on the filesystem and the cluster size

#### ERROR: boundary alignment is too big

If you get this error, chances are your block size does not divide the volume size or some other error. This error is specific to `exFAT` filesystem but other filesystems display error in a similar way. I would try setting the filesystem cluster size as same as the block size I used in `dd` and if that didnt work out, I will switch the cluster size to `1M`. In some weird case, you may need to reboot the system in order to not have this issue again depending on what version of filesystem driver you are using. But is rarely the case.

### Mount the filesystem

- Choose a mount point

    Mount point can be any empty directory in the system at any location owned by any user. Make sure the directory exists, create it if it does not. In this example, I am going to choose `/mnt` which is already present standard mount point used for this purpose

- Create a folder inside your desired mount point

    It is recommended that you create a folder inside your desired mount point with the name you used to create LUKS volume. This makes it easy to manage multiple volumes that you wish to use. In this example, I will make a folder named `myVolume`. Note that since `/mnt` is owned by `root` user, I need to run this as a superuser. But if it were in a directory your user owned, you should not run this command as a superuser but the user that owned the parent directory
    
    ```
sudo mkdir /mnt/myVolume
    ```

- Mount the volume with permission for your user

    If you are using a filesystem that does not support Linux permission and ownership, you would pass in the option `-o uid=$(id -u),gid=$(id -g)` when mounting. This works well for filesystem such as `exfat` This, however will cause you an error when mounting a filesystem that supports Linux permission and ownership such as in case of `ext4` and other linux native filesystems. In that case, you would mount the directory normally then `chown` the files within to your user.

    In this case, I am using `exFAT` filesystem and wish to mount into `/mnt/myVolume` so I would do it as

    ```
sudo mount -t exfat -o uid=$(id -u),gid=$(id -g) /dev/mapper/myVolume /mnt/myVolume
    ```

    But say you used `ext4` filesystem when formatting the volume. In that case, you would first mount the volume normally as

    ```
sudo mount -t ext4 /mnt/myVolume
    ```

    followed by

    ```
sudo chown $USER /mnt/myVolume
    ```

    This means, the actual mount point for my case now is `/mnt/myVolume`

  

  

### Use that volume to store sensitive files

  

The file is mounted in `/mnt/myVolume` so that means you can open up you file manager, navigate to `/mnt/myVolume` and start operating on the files there normally. Everything in `/mnt/myVolume` in this case will be stored encrypted inside that volume you created

  

### Sync disks
If you have recently written alot of data into that volume, it is a good idea to sync the disks after you have completed the copy or move operation to ensure that file buffers were properly written to disks and there will be no file corruption. To do that you can simply run the command

```
sudo sync
```
  

### Unmount the volume

  

After you are done, you can unmount the volume before closing the container

  

```
sudo umount /mnt/myVolume
```

  

### Close the volume

  

After you are done, you should close the access to the container via its name you used when opening

  

```
sudo cryptsetup luksClose myVolume
```

  
### Remove the folder you created to mount the volume within your chosen mountpoint
I created a folder called `myVolume` inside `/mnt`. After I am done working with the volume, I will delete that folder because it is no longer needed. Make sure you are choosing the correct path for the folder you wish to delete. You would adjust this command according to the mount point you used

```
sudo rm -r /mnt/myVolume
```

### [Optional] Change ownership of /mnt back to root
It is recommended that you do not mount directly into `/mnt`, especially if you used a linux native filesystem because you may end up unintentionally owning `/mnt` directory for your user. This is not default behaviour and can cause some issues in some cases. To fix that, you will have to own `/mnt` as `root` user

```
sudo chown /mnt root
```

## Normal usage

  

You are not always going to follow all these steps when using the volume, you can just

  

- Open the LUKS volume

  

- Mount the opened volume from /dev/mapper/...

  

- Start using it

  

- Unmount the opened volume

  

- Close the LUKS container


- Remove the folder you made to mount if it was not already present

  

  

## Conclusion

  

Even though the steps are quiet cumbersome at first, remember that you can script most of it to make it easier to interact everyday. You can use my scripts from my gitlab linked above or write your own scripts. There are many features in LUKS such as using multiple passwords to open the same volume, storing the header in another location, etc. Everything I have written is just a tip of the iceberg and i suggest you read more on LUKS's features and so on if you want to learn more
