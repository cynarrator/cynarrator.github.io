---
Title: Start using Password Manager
date: 2020-12-09 19:35:00
categories: [homelab, hardware]
tags: [servers, dell, hp, supermicro]
---
## Introduction
I cannot tell you how much money you save not having to goto your bank every time to reset your passwords and therefore not allowing them to charge you Rs.50. Modern problems require modern solutions.

> Note that this is what I use and suggest people do if they dont have any idea. You dont have to use MEGA cloud, you dont even need to use any cloud if you dont want. Keepass is an offline extensible password manager that it only saves in a file on your computer. You can do whatever you want with it and yes, its encrypted.
{: .prompt-note }

## Steps
### It is actually quiet easy

* Install Keepass from [Keepass](https://keepass.info)
* Create an account in [MEGA](https://mega.nz)
* Install Megasync desktop client
* Setup a sync between a folder in your computer to a folder in the cloud
* Start using Keepass and save your password database in that folder which will automatically be synced to MEGA.
* From that folder in MEGA, share a link and write it down in a paper(I am talking about writing the MEGA share link and not your passwords) and store it somewhere safe, like with your citizenship. Write link and the decryption key separately in the next line.
* Maby also copy Keepass database file to another storage device and in your phone too.

In the worst case scenario if you lose your Keepass file, you will have to type out that shared link in the browser, but in MEGA its not that long and you can type link and decryption key separately.

## What of cloud based password managers

In those kind of password managers, if I somehow end up guessing your password, all I have to do is login to their website. I can do that from anywhere mostly.
However, in something local like Keepass, I will also need a way to steal your database file which gives you more time to react.
They do allow you to have 2FA, but you will still need a place to securely store those 2FA tokens somewhere. This would be another solution but you will need a phone to login every time, unless you get a good TOTP app on desktop.