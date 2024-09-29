---
Title: My Keepass Setup with MEGA Cloud
date: 2024-08-29 5:53 PM
categories: [Misc]
tags: [tip, security]
---

## Introduction
Keepass is a pretty simple and easy to use password manager designed to be used locally by oneself in their own devices. This ensures that Keepass runs totally offline meaning that you can use it in places with limited internet access. This also ensures flexibility as you decide where your password vault lives. You can use Keepass with any cloud storage service you want or not use it with over the internet at all depending on your preference. You can create multiple vaults for different reasons as opposed to being tied to a single account.

The only gripe I see people have with Keepass is Password sharing between multiple people. Though this can be done, it works differently than most expect. But this article is not solely about Keepass and how to use it. There are many great articles on that. What I want to focus on is how I have been using it ever since the lockdowns in the most convinient way possible while minimizing as much limitations of Keepass as I can.

## Why MEGA?
While MEGA may not have the most solid reputation among general public, their share link is something you can write down in a piece of paper and type in when you need it. Making it suitable in case of emergency. Here is what I mean.

Assume the following MEGA Link:

    https://mega.nz/folder/yuYGhYiZ#jOFY01CXS6EBNLcyXLaEsg

MEGA Links can be broken down into two parts, the link part and the decryption  key part. They are seperated by a `#`. Here, if you split the above link by a #, you will end up with the the following:

    Link: https://mega.nz/folder/yuYGhYiZ
    Decryption Key: jOFY01CXS6EBNLcyXLaEsg

When you access the link, you will be prompted for a decryption key. Because they are both relatively shorter, you can write them in paper and type them by hand when you need them. You will only be needing them in case of emergency and you will be glad you wrote them when you do need them. So it is a good idea to write those. It also helps prevent chicken and egg dilemma because you can generate a pretty secure password for MEGA saved in the same vault and you can access your keepass vault without knowing your MEGA password.

## Which Keepass application to use
You should use Keepass 2.x compatible application. You can download keepass from their [official website](https://keepass.info/download.html). I personally use a fork of Keepass called [KeepassXC](https://keepassxc.org/) on desktop and [KeepassDX](https://www.keepassdx.com/) on Android. Although I do not own an iPhone at this moment, I have heard good things about [Strongbox](https://apps.apple.com/us/app/strongbox-password-manager/id897283731) on ios.

In general, if you use any Keepass 2.x compatible application that produces the database of the format `kdbx` you are fine, it really does not matter which app you use as long as it is listed under Keepass's official downloads page.

## Keeweb
Keeweb is a software that allows you to access Keepass database in a web browser without installing any software. Even though Keepass is something you are supposed to install on your computer, this is great as it allows you to access your password vaults when you are in someone else's computer and do not have permission to install any other software.

Follow the steps to get Keeweb working
* Download Keeweb

    You can download Keeweb by going to [their Github Releases page](https://github.com/keeweb/keeweb/releases/) then download the file named `KeeWeb.html.zip` where the version number does not matter.

* Extract the zip file

    You can extract that zip file using Winrar or any other unzipping program available in the platform you are currently on

* Open your valut

    You can open `index.html` using a web browser and click on the bar to locate and open your vault.

**You are only expected to use Keeweb in case of emergency. Do not use it regularly as it is not meant to be. Use a normal installed version of Keepass for daily use**

It is recommended that you place this zip file with your password in MEGA in case you need it.

## Syncing Passwords
I personally use MEGA's sync software that is cross platform. You just setup sync with your desktop computer to a folder in MEGA and sync it. That way you dont have to remember to update the vault in MEGA when you change your password from your computer.

It is absolutely important to note that Keepass is a local password manager and your changes will not automatically be shared among your devices. You will have to manually copy your new database file and replace the old one.

When I am on the Go and need to change my password, I change my password in Keepass app on my phone, then upload that new `kdbx` file into MEGA where old file is, essentially replacing the old password database. And because that keepass folder is synced with my computer, it will get replaced in my computer with the new one thanks to MEGA Sync when it is turned on and connected to the internet.

And when I change my password in my computer, it gets replaced on the cloud drive meaning that I just have to download the new database file on my phone using the MEGA app and replace the old one already present in my phone.

## Non synced backups
MEGA does not sync with the phone. Meaning that a non synced backup is preserved in my phone. In case something happens and my Keepass database is corrupted in my computer. Thanks to sync, it will also get corrupted in the cloud drive. But thanks to me having a copy of the database in my phone that is not synced with MEGA, I can just copy that vault back to my computer via MEGA.

Still, it is recommended that you backup your password database regularly. You should remember to diligently replace your backup with new version of the database every time you change the password depending on how often you need to change your passwords.

## Conclusion
Keepass is a great password manager, you can use it totally offline without ever having it connected to the internet. This is both a blessing and a curse. On one hand, you are sure your passwords are available without internet and you can control where your passwords live. On the other hand, it is a curse because you will have to manually make sure to change and update your password database in all your devices when you change some things. You can use a cloud storage to ease that pain abit. Also, thanks to MEGA, I can be sure that I can access that vault even when I dont have access to my devices with the share link I noted in a paper.