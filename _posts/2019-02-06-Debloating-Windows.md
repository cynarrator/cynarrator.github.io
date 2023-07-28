---
Title: Remove Bloatware from Windows
date: 2019-02-06 6:08 AM
categories: [Technology]
tags: [windows]
---

I suggest you to do this step right after you install windows 10 to make sure you dont actually randomly remove what you currently rely on.

First, open up powershell by typing it in run box or on command prompt then run the command:
```powershell
Set-ExecutionPolicy Unrestricted
```

to allow execution permission to the script that we will use. (I frankly have 0 idea on how windows handles read write execute permission. I just read this on forums, Anyone knows better on this???)
Then, clone or download zip file and unzip it from the following git repo:
[https://github.com/Sycnex/Windows10Debloater](https://github.com/Sycnex/Windows10Debloater)

After that, cd into and run the file named Windows10Debloater.ps1 with,
```powershell
.\Windows10Debloater.ps1
```
Wait for it to complete.
You may then reinstall some software if it were removed by this script but you need them. You may also try running individual scripts from the git repo.