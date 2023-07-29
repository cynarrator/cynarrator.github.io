---
Title: Signing the text in your about me section of your Facebook profile using GPG
date: 2021-06-12 5:19:30 PM
categories: [Technology]
tags: [privacy, security, tip]
---
## Introduction
Have you ever wondered if there is actually a way to prove you own a Facebook account using GPG. Well, there is a way to sign a text in your about me section in your Facebook account and have it be verified using your GPG secret key

## Prerequisite
* You understand how to sign and verify documents using GPG or OpenPGP application

## Limitation
* Obviously, this requires that those wishing to verify your profile knows how to use GPG which is unfortunately rare.

* This only works with [mbasic.facebook.com](https://mbasic.facebook.com)

## How do we do it
1: Write your message that you want for About me in a text file. Name it ``about.txt``. Use simple text editor such as notepad and not something like MS Word. *Leave two extra blank lines at the top and the bottom of about.txt to make it tidier and easier to read*.

2: Clearsign that text using gpg.

```bash
gpg --clearsign about.txt
```

3: It will produce a text file called ``about.txt.asc``

4: You need to add four spaces in front of each line in that signed file(about.txt.asc). You can use any method to accomplish this, if you have sed installed (Most GNU/Linux and UNIX like distributions have it preinstalled) you can use sed to do it and put the result in another file. You can use following command;

```bash
sed 's/^/    /' about.txt.asc > about_seded.txt.asc
```

**Remember, thats four spaces in there between the slashes**. Or you can use any online tool to do this, I will suggest using [https://sed.js.org](https://sed.js.org) if you dont have sed installed.

5: Open [mbasic.facebook.com](https://mbasic.facebook.com) and navigate to the About me section in your profile. Paste the contents of ``about_seded.txt.asc`` in there.

6: When you paste it in there, only remove four spaces in front of the first line, ie the line with ``-----BEGIN PGP SIGNED MESSAGE-----`` **Repeat: Only remove the beginnig four spaces for this line. Leave others intact**

7: Save and check signature. Now it should verify

## Why cant I just paste my signature file in there and have it verify
I do not know why that is the case, in my test, when I just paste the text in there without anything I described above, the signature does not verify

## Why cant I use the new Facebook
Well, new Facebook will malform the text in order to make it look pretty, that is something you do not want