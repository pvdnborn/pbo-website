---
layout: post
title:  "Windows 10 Multi-Session OS corruption when using “Install-Language” Powershell Module"
date:   2023-01-24 21:00:00 +0100
categories: Packer, Azure Virtual Desktop, AVD, Windows 10, Windows 10 Multi Session, Install-Language
---
Currently, I'm doing some Microsoft Azure image deployments for Windows 10 Multi-Session 22H2. During the automation part, I found an annoying bug that I want to share with you. This bug is so irritating that it will corrupt Windows after using the ```install language``` module. You will notice that you can't connect via TCP/IP to the system anymore (including RDP/Bastion). Redeployment of the operating system is needed since there is only access via the serial console.

Multiple people contacted me, facing the same issues since I tweeted about this issue. Tweet link: ([https://twitter.com/pvdnborn/status/1615672687199748097](https://twitter.com/pvdnborn/status/1615672687199748097)). There is a workaround for this issue, which I will describe in this blog too.

## Symptoms
When using the latest marketplace image for Windows 10 Multi Session 22H2:

```javascript 
#ARM Marketplace Image Info
os_type = "Windows"
image_publisher = "MicrosoftWindowsDesktop"
image_offer = "Windows-10"
image_sku = "win10-22h2-avd"
```
The operating system will fail after executing the following command:
```powershell
Install-Language nl-NL -CopyToSettings
```
Or executing a similar command like:
```powershell
Install-Language de-DE -CopyToSettings
```
After running the ```Install-Language``` command, the system will be inaccessible via TCP/IP when rebooting the machine. So, no access to the VM with RDP or Bastion. Via the serial console, I have figured out that the Windows Firewall service isn't running and can't be started. Something breaks the operating system when running the ```install-language``` command. Even my automated Packer build will fail with the weirdest filesystem error messages like:

```
==> azure-arm.DTJ-AVD-DEV-Win10MS: Attempting to perform the InitializeDefaultDrives operation on the 'FileSystem' provider failed.
```

## Workaround
I found a workaround like the old days: just reinstalling Windows NT4 SP6A every time. People who faced this issue confirmed this workaround works for them.

The workaround is as follows:
1.	Install-Language nl-NL -CopyToSettings
2.	DO NOT REBOOT THE MACHINE!
3.	Install Windows updates!
4.	Reboot machine

Installing a cumulative update after using the ```Install-Language``` seems to fix the issue or repair the operating system. So if you don't have updates, or the marketplace image is up-to-date, then you're screwed! 

## Use an alternative method to install language packs
At first, I liked the new PowerShell command ```Install-Language```. The ```Install-Language``` command saves much time and complexity compared to the alternative method. Since ```Install-Language``` is breaking the operating system, I prefer to use the alternative method with DISM and LXP. I've written a blog in the past about this procedure: ([https://www.vandenborn.it/2019-10-18-wvd-and-vdi-automation-change-in/](https://www.vandenborn.it/2019-10-18-wvd-and-vdi-automation-change-in/))

## Last words
I hope I've saved you some troubleshoot time by posting this information. The first time I ran into this issue, it took me hours to troubleshoot and identify it was Install-Language breaking the Windows 10 Multi Session operating system. Note: I didn't verify if Windows 11 Multi Session has the same issues.

It would be nice if Microsoft will fix this issue in the next releases of Windows 10 Multi Session, so ```Install-Language``` will make our automation live easier.