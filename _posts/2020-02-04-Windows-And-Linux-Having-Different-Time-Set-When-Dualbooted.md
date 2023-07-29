---
Title: Windows and Linux have differnt time set in dualboot
date: 2020-02-04 07:10 PM
categories: [Technology]
tags: [linux, windows]
---

## Introduction
When our computer shuts down, everyting in our RAM is deleted. This includes date and time. So, in order to keep the time running, there is a special clock in the motherboard. It is supposed to be always running.

When the Computer starts, the Operating System reads the time from this clock. And when the computer shutdowns, the Operating System will write that time down to the motherboard.

There would have been no problem, except, Windows uses Localtime and Linux uses UTC. To solve this, you will either have to convert Windows to use UTC, or Linux to use Local Time.
## Do one of the following
### Convert Windows to use UTC
Its actually quiet straight forward,

* Open up notepad and paste the following content
```
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation]
"RealTimeIsUniversal"=dword:00000001
```
* Save it with ``.reg`` extension
* Double click to run it

### Convert Linux to use Localtime
Its also as easy as running a single command as root,
```bash
timedatectl set-local-rtc 1
```
