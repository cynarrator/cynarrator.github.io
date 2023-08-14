---
Title: Adding UNIX Style sudo in Windows
date: 2019-02-01 7:11 AM
categories: [Operating System]
tags: [windows]
---

Well, most of us, when we need to run CMD on windows as Administrator, simply open it as Administrator by right clicking it. It does work pretty fine but there may be situations when you need to only run few commands as Administrator but others as normal user.

In *NIX systems, you would simply type sudo in front of commands to only run them as Administrator while other as normal user, but sudo is not a valid option in Windows.

In windows, there is a rather lengthy command called ``runas`` which is relatively longer than sudo but works just like sudo. So here, we will make it so that every time we type sudo, that long runas command gets replaced in place of sudo.

Well, you must have an administrator password for it to work, or atleast, I couldnt get it working when I had no password for adminstrator account. Also note that you will use the name of the administrator account and not administrator itself. Assuming the name of administrator account is ``dell``, which in my case it really is. Your's will be different.

First, open computer (Formally known as My Computer), goto ``C:\\Users\<CurrentUsername>\`` and create a folder called ``bin`` where you will later create the batch script.

Then, right click on computer, click on Properties, click on advanced settings on left side, then click on environment variables. Edit the Path variable on bottom box by adding the directory path to the bin folder you created beforehand. If all the paths are in the same line, keep in mind that ';' is used to separate different directories in there. If it is given in list format, just click on add and add it. I suggest you add this at the end of Path variable.

Then, open up notepad and type the following;
```bat
@echo off
@runas /user:dell "%1 %2
```
Remember to replace dell with the name of administrator account you have on your system. After typing this, save this file over to the bin folder you created and added to Path earlier as sudo.bat. Make sure you get the extension(.bat) correct.

If you did everything correctly, sudo should now work in the command prompt.