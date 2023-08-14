---
Title: Get Saved Wifi Password from Linux
date: 2019-05-16 12:01 AM
categories: [Operting System]
tags: [linux, tip]
---

If your distribution uses NetworkManager, which I believe most mainstream linux distributions do, you can find your wifi passwords stored in text file inside
```bash
/etc/NetworkManager/system-connections/
```
in the corresponding text file as your wifi network name, you will see the wifi password in plain text in psk= line. Those text files can only be read by root user for the sake of security. Remember that your distribution should use NetworkManager.

Or, you can type the following command,

```bash
sudo grep -H '^psk=' /etc/NetworkManager/system-connections/*
```