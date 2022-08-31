---
layout: post
title:  "In-place-upgrade and installation of Citrix XenDesktop/XenApp 7.8 VDA on Windows 2012 R2 fails 'Ica\_TS.msi'"
date:   2016-05-24 13:40:00 +0100
categories:  Citrix, Windows 2012 R2, XenDesktop, VDA, Virtual Delivery Agent, XenDesktop 7.8, XenApp 7.8, Upgrade, XenApp
---

Today I unexpectedly had a challenge upgrading a Citrix XenApp 7.6 VDA to Citrix XenDesktop 7.8 VDA on Windows 2012 R2. During the in-place-upgrade of the 7.8 VDA or a reinstallation of the 7.8 VDA, the installation fails at the section Virtual Delivery Agent:  

[![]({{ site.url }}/images/2016-05-24/2016-05-24-image001.png)](https://2.bp.blogspot.com/-7tn3HrT6Fvw/V0Q8URFCWHI/AAAAAAAAAaE/uTY4cP6meIkVhD3YRHBAmIRrnJrtj2gGwCKgB/s1600/image001.png)

  
The XenDesktop Installation log (opened if **Show error log** is enabled) is showing the following error:  
  

_13:20:54.8513 $ERR$ : XenDesktopSetup:Installation of MSI File 'IcaTS\_x64.msi' failed with code 'InstallFailure' (1603)._

_13:20:54.8513 $ERR$ : XenDesktopSetup:InstallComponent: Failed to install component 'ICA for Remote Desktop Services'. Installation of MSI File 'IcaTS\_x64.msi' failed with code 'InstallFailure' (1603)._

_13:20:54.8523 $ERR$ : XenDesktopSetup:Recording installation failure. Installation of MSI File 'IcaTS\_x64.msi' failed with code 'InstallFailure' (1603)._

The following error is in the Application log of the Windows Eventviewer:

_Product: Citrix HDX TS (retail) -- Error 1911. Could not register type library for file C:\\Program Files\\Citrix\\Euem\\Service\\SemsComLibrary.tlb. Contact your support personnel._

[![]({{ site.url }}/images/2016-05-24/2016-05-24-image003.png)](https://3.bp.blogspot.com/-yZ4X9M5kx1I/V0Q8URLxPUI/AAAAAAAAAaI/0QYZpEFXa0wbUn1m8yCsrtAIk4YdXXrugCKgB/s1600/image003.png)

To solve this issue, uninstall the following Windows updates prior updating or installing the Citrix XenDesktop 7.8 VDA:

*   KB3139923, [https://support.microsoft.com/en-us/kb/3139923](https://support.microsoft.com/en-us/kb/3139923)
*   KB3072630, [https://support.microsoft.com/en-us/kb/3072630](https://support.microsoft.com/en-us/kb/3072630)

After the VDA upgrade/installation it is save to reinstall the KB updates again!