---
Title: File Sharing with SAMBA over LAN in Linux
date: 2024-10-31 8:13 PM
categories: [Misc]
tags: [tip, linux, utility]
---

## Introduction
I am sure you have been in a situation where you want to share files over LAN with various other systems. There are various techniques to do so easily in Linux such as setting up FTP, SFTP, etc. These are easy enough to use and there is nothing wrong with using them. In this article however, I wish to explore how to setup SAMBA share, also referred to as SMB. This is mainly because SMB seems to be available out of the box in many different platforms.

## Pros
* Fast
* Available on almost any platform

## Cons
* No encryption by default, look below on possible hardening tips

## Important side note
You need to provide execute permission to a directory, otherwise one cannot access files stored in there. You however, dont want to provide execute permission to files within your shares in most cases for security reason. Another side note is you will see me interchangeably using `folder` and `directory`. They are the same thing just one's habit on what they choose to use.

`$USER` in this context generally will be resolved to the currently logged in user. But in case you want to share files as another user, you would replace that users's username there instead. This however does not work in configuration files such as `/etc/samba/smb.conf` and `/etc/fstab`. In that case, you will replace `user` with your username.

## Installation
On Ubuntu and Ubuntu based distributions, you would install samba as
```bash
sudo apt-get install samba
```

Your SAMBA configuration file is then located in `/etc/samba/smb.conf`

## [Optional] Disable SAMBA from running on boot automatically
By default on Debian/Ubuntu and various other distributions, the service associated with the application you install is enabled to run on boot. This means it will autostart on boot. If you dont want it to run automatically on boot, you can do so with the command
```bash
sudo systemctl disable smbd
```
In this case, you would start SAMBA when you need by running the command
```bash
sudo systemctl start smbd
```

## [Optional] Disable Printer Sharing
If you do not wish to share your connected printer with anyone in the network, you may comment all the lines related to printer in `/etc/samba/smb.conf`. You do so by adding a `#` in front of the lines you wish to comment. Here is what that looks like
```conf
#[printers]
#   comment = All Printers
#   browseable = no
#   path = /var/spool/samba
#   printable = yes
#   guest ok = no
#   read only = yes
#   create mask = 0700

# [print$]
#   comment = Printer Drivers
#   path = /var/lib/samba/printers
#   browseable = yes
#   read only = yes
#   guest ok = no
```

You also want to put the following under `[global]` section to be sure printer is disabled
```conf
[global]
load printers = no
printing = bsd
printcap name = /dev/null
disable spoolss = yes
```

## Register your user with SAMBA
Before you forget, its a good idea to register your user that you wish to share file as with SAMBA. You will be prompted for a password. This password can be and should be different from your account password. This is the password you will use to access files in a folder you share as a private share. Keep in mind that anyone who can get their hands on this password can access your private share you are about to create below.

To register your user with SAMBA, you would run the command
```bash
sudo smbpasswd -a $USER
```

## Allow SAMBA through your firewall
You may also want to allow samba through the firewall. If you are using `ufw`, you can just do
```bash
sudo ufw allow samba
```

If you are using other kinds of firewall, check if samba is running under different port in `/etc/samba/smb.conf`, otherwise it runs on TCP port 445 by default

## Create a private share
In case of private share, you wish to provide full read-write access to a folder in your computer provided username and password. Username is the username that you registered with SAMBA previously, password is the password that you set with `smbpasswd`. You however, dont want to provide execute permission to the clients.

### Create a new folder that you wish to share.

I wish to create a folder named `SambaShare` in `/home/user/SambaShare` that I will share as a private share.

### Change ownership of share folder to the user sharing it

If that folder is not inside that user's home directory like mine is, you will have to make that directory own by the user. You would run the following command to do so
```bash
sudo chown $USER /home/user/SambaShare
```

### Provide appropriate permission

You want to provide `700` permission to the `SambaShare` directory. This provides read, write and execute only to the owner of the folder. Which is what you want. Keep in mind, you are not changing permission of contents within the folder, just the folder itself

```bash
sudo chmod 700 /home/$USER/SambaShare
```

### Add configuration in the configuration file

```conf
[private_share]
path=/home/user/SambaShare
browseable=no
read only=no
create mask = 0644
directory mask = 0755
guest ok=no
force user=user
valid users=user
```
We first start with a section name within `[]`. This name will show up when browsing the share and can be set as path in some clients. 

Next, we put in `path=` option. This is the path to the directory that we wish to share. You would replace this with where your share folder is located

Then we set `browsable` option. This sets whether the share name is hidden from the client or not. Setting it `no` because we dont want clients to know there is a private share

Then we set `read only` to `no` because we want read-write access to the clients

Then we set `create mask` and `directory mask`. They are the permissions of newly created files and directories within the share respectively. The `0` in front is just to denote its an octal number system. For files, I only want read and write permission to the user owning the share. Everyone else only gets read permission. For a directory, I want read/write/execute for the user owning the share and read + execute permission for everyone else

Then we set `guest ok` to `no` to explicitly state we dont want anonymous users to access files in here. Adds more to prevent unknown users from accessing the share

Then we set `force user` to the name of user who we wish to own files created by the clients. We want to set this because not all clients support Linux ownership and permission system. This ensures that is not a problem.

Finally, we set `valid users` to the list of users who can login to the share and access the files in there. This is a comma seperated list of users. All the users in here should be registered to samba by running `smbpasswd`. In case you need to allow multiple users to access this share, you would put all these users in the same group and let that group access to the share. But in this case, you can just set it to the user you want to share as. Provides yet another layer of authentication

## Create a public share
In case of a public share, you want anyone in your LAN access to files without login or providing any password. For this reason, you would not place in any sensitive files in there. But this is more convinient when you want many people in your LAN have access to files and you do not want to share your passwords around.

### Create a new folder in your home directory and in the root filesystem
When you share anonymously, you want to share as the built in user `nobody` as opposed to your user. This causes issue because `nobody` does not have access to your home directory. You may provide access to a folder within your home directory to `nobody` but it cannot even get past your home directory. It will be like me providing you access to someone's house but you are banned from entering the neighbourhood where that house lies.

To get around this problem, you will create one directory outside the home directory and another one inside your home directory. You will bind mount these two directories so that they share the same space from your home directory. You will also own this as your user to make permission system more consistent in subsequent steps

```bash
sudo mkdir /Public
sudo mkdir /home/$USER/Public
```
Some distributions and desktop environments will have a folder named `Public` in the home directory of user by default. You can use that as it was intended for similar purpose

### Own the folder outside home directory by the user sharing the files
To make things easier on yourself, its a good idea to own the directory outside the home directory by the user sharing the files. Here I created a folder named `/Public` that I will own my my user
```bash
sudo chmod $USER /Public
```

### Bind mount directory from within your home directory and the one outside
That way, both of those directories have the same contents. You would set the directory inside home directory as the source, provided first and the one outside as the destination, provided second.
```bash
sudo mount --bind /home/$USER/Public /Public
```

You may also want to add the following line at the end of `/etc/fstab` to auto bind mount on boot so that you dont have to do it every time you start your computer
```conf
/home/user/Public /Public none bind 0 0
```

### Provide appropriate permission
You want to provide permission `755` to `/home/user/Public` because you do not want anyone other than the owner to modify files in there. Others cannot add new files or edit/delete files. They can only read files.

```bash
sudo chmod 755 /home/$USER/Public
sudo chmod 755 /Public
```

### Add the following section

```conf
[public_share]
path=/Public
read only=yes
browseable=yes
create mask = 0644
directory mask = 0755
guest ok=yes
guest only=no
```
For explanation of these options, check on **Create a private share** section, its mostly the same with minor changes. We set `guest only` to `no` because we want users who can access private share also be able to view this share.

## Hardening SAMBA
SAMBA by default does not offer much in the name of security. There is no encryption by default meaning that anyone in the network can read and modify traffic on the fly if not secured. You are still not recommended to solely rely on SAMBA encryption as it is not really meant to be secure, its only really good with LAN where you trust devices within. But regardless, here are some configuration changes you can have that can increase security in your SAMBA share. It is okay if you place these in your private share or within `[global]` section if you want all shares to be encrypted. I choose to put these only under my private share.

You are trading compatibility for security in the case of SAMBA

```conf
[private_share]
# https://serverfault.com/questions/913504/samba-smb-encryption-how-safe-is-it
server min protocol = SMB3_11
server smb encrypt = required
server signing = mandatory
server smb3 encryption algorithms = AES-128-GCM, AES-128-CCM, AES-256-GCM, AES-256-CCM
server smb3 signing algorithms = AES-128-GMAC, AES-128-CMAC, HMAC-SHA256

client min protocol = SMB3_11
client smb encrypt = required
client signing = required
client ipc signing  = required
client protection = encrypt
client smb3 encryption algorithms = AES-128-GCM, AES-128-CCM, AES-256-GCM, AES-256-CCM
client smb3 signing algorithms = AES-128-GMAC, AES-128-CMAC, HMAC-SHA256

# client use kerberos = < off | desired | required >
# kerberos encryption types = < all | strong | legacy >
```

On top of that, you are advised to use a private VPN to connect to your Samba share.