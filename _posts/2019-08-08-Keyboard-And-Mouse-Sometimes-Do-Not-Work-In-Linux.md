---
Title: Keyboard and mouse do not work sometime in Linux when booting up
date: 2019-06-28 9:50 AM
categories: [Technology]
tags: [linux, issue]
---

Problem: *Keyboard and touchpad suddenly stops working after a reboot on Linux.*

I am writing this because I was the one who have been facing this issue for a very long time and noone could help me. So I am writing what I found out after some research so that you guys wont face the same problem I faced.
This is the most common problem in linux based Operating System and no, this is not because of your Distro's fault as most think, but is the problem in the kernel itself.

## Explanation of the situation
Lets start it off here. There are two ways you can connect keyboard and mouse (touchpad in this case) on a computer. You either use the classical PS/2 port or the modern USB port. And this causes confusion because in a laptop you dont even have to connect a keyboard or mouse as its already there. It depends on how they are connected to the motherboard that you do not see.

> PS/2 controller is also known as i8042 controller.
{: .prompt-tip }

Most older laptop connect as a PS/2 port. Generally, there used to be a single port for a mouse and keyboard in older motherboard. But to make matter more confusing, now they started supporting multiplexing (a technique that allows merging multiple signals into one signal and sending it from the same medium).

Generally, it is advised to disable multiplexing unless your laptop has multiple PS/2 ports for mouse and keyboard. So, while kernel boots, it sometimes detect that but sometimes it doesn't and you have to restart.

Until and including kernel 2.6.31 all was good using a script unbinding the devices connected to it and rebinding them on resume/thaw. That does not work anymore. That's why you will have to fiddle around with any of these boot options of the i8042 nastiness:

* **i8042.direct** - Put keyboard port into non-translated mode
* **i8042.dumbkbd** - Pretend that controller can only read data from keyboard and cannot control its state (Don't attempt to blink the leds)
* **i8042.noaux** - Don't check for auxiliary (== mouse) port
* **i8042.nokbd** - Don't check/create keyboard port
* **i8042.noloop** - Disable the AUX Loopback command while probing for the AUX port
* **i8042.nomux** - Don't check presence of an active multiplexing controller
* **i8042.nopnp** - Don't use ACPIPnP / PnPBIOS to discover KBD/AUX controllers
* **i8042.reset** - Reset the controller during init and cleanup
* **i8042.unlock** - Unlock (ignore) the keylock

## Where do you put in these boot parameters
This depends on the bootloader you are using. Different bootloaders allow you to pass in these boot parameters, also known as kernel parameters into the kernel in different ways. Check that with your bootloader that you use

## Too Long Didn't read
Well right now, if you are having issues where your laptop trackpad and Keyboard randomly do not work sometimes after turning it on, pass the following kernel parameters

```
i8042.reset i8042.nomux i8042.nopnp i8042.noloop
```

## Sources
* [https://lightrush.ndoytchev.com/random-1/i8042quirkoptions](https://lightrush.ndoytchev.com/random-1/i8042quirkoptions)

* [https://www.kernel.org/doc/Documentation/admin-guide/kernel-parameters.txt](https://www.kernel.org/doc/Documentation/admin-guide/kernel-parameters.txt)