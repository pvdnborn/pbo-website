---
layout: post
title:  "Rumour: Technical limitations Citrix Provisioning Service Datacenter edition in combination with Citrix XenApp and XenDesktop"
date:   2016-11-11 15:28:00 +0100
categories:  Provisioning Services, Citrix, XenApp 7.9, XenDesktop 7.11, XenDesktop 7.9, rumour, XenApp 7.11, XenDesktop 7.8, XenApp 7.8
---
Since the release of XenApp/Desktop 7.8 a rumour was spread about some technical limitations for Citrix Provisioning Services when Citrix Provisioning Services Datacenter licenses are used in combination with Citrix XenApp and XenDesktop.   
  
The rumour is a follows: When your Citrix XenApp and Citrix Provisioning Services is upgraded to 7.8 or higher and Citrix Provisioning Services Datacenter licenses are used, the Citrix Provisioning Services Target Devices are shut down when the XenApp VDA is installed.   
  
The Citrix UELA describes that usage of Provisioning Services Datacenter edition is not allowed in a XenApp or XenDesktop environments. This blog only covers the technical discussion not the EULA part.  

[![]({{ site.url }}/images/2016-11-11/2016-11-11-image002.jpg)](https://1.bp.blogspot.com/-zts_pXZrxkU/WCXOvBdUOtI/AAAAAAAAAhM/8EPub3jMJjMXGUBLsYru-yehRmR899PKACEw/s1600/image002.jpg)

Due to this rumour I was very cautious for upgrading customers to 7.8 and higher. Upgrades of Citrix Current Release versions is something what I do very often, so I need to be sure this rumour is true or false. I decided to install a Citrix XenApp Advanced environment with Citrix Provisioning Services Datacenter licenses in my home lab. For this test I started with XenApp 7.8 and Provisioning Services 7.8 because the rumour tells me that this is the version when all the trouble is started. After testing 7.8 I’ve upgraded both XA and PVS to 7.9 and finally to 7.11.

After testing, my conclusion is: The rumour is **not true**! There are no sudden PVS shutdowns or strange error messages when using Citrix Provisioning Services Datacenter licenses in combination with XenApp Advanced.

Screenshots: XenApp 7.8 and Provisioning Services 7.8 test
----------------------------------------------------------

The test setup is as follows:

Citrix Backend, one Windows 2012 R2 Server with roles:
*   Citrix Licensing Server 
*   Citrix XenApp Delivery Controller 7.8
*   XenApp Advanced Concurrent licenses
*   Citrix Provisioning Services 7.8
*   Citrix Provisioning Services licenses

  

Citrix Worker, one PVS Streamed Windows 2012 R2 Session Host:
*   Citrix XenApp 7.8 VDA
*   Citrix Provisioning Services 7.8 Target Device Software
*   VM: Hyper-V Gen2
*   PVS Boot: TFTP UEFI

### Citrix Desktop Delivery Controller 7.8
[![]({{ site.url }}/images/2016-11-11/2016-11-11-image004.jpg)](https://3.bp.blogspot.com/-ifnSsTPhEvQ/WCXOvFmcAzI/AAAAAAAAAhI/DNMgmlEoO5cu0fQ1zIykmRi7EkYFzV2kgCEw/s1600/image004.jpg)

### Citrix License Administration Console
[![]({{ site.url }}/images/2016-11-11/2016-11-11-image006.jpg)](https://2.bp.blogspot.com/-vvAwbivVmKE/WCXOvCqcyoI/AAAAAAAAAhQ/h_6nt9fbmJAAWhiuQ5xZjSpBGbAtk81bACEw/s1600/image006.jpg)

### Citrix Provisioning Services Server 7.8
[![]({{ site.url }}/images/2016-11-11/2016-11-11-image008.jpg)](https://1.bp.blogspot.com/-7NrInjetHHM/WCXOveU5TkI/AAAAAAAAAhU/9SMBqPuONQ4UzgO8oACXa1pbgeWRNFuhACEw/s1600/image008.jpg)

### Citrix XenApp 7.8 VDA - Session Host
[![]({{ site.url }}/images/2016-11-11/2016-11-11-image010.jpg)](https://2.bp.blogspot.com/-zB-KV50EXeM/WCXOvUVYo_I/AAAAAAAAAhY/eDgziPG67BgssWwDX6eZRTUrHcaStpJbACEw/s1600/image010.jpg)

### User ICA Session to XenApp 7.8 VDA
[![]({{ site.url }}/images/2016-11-11/2016-11-11-image012.jpg)](https://1.bp.blogspot.com/-QQ3HKqPurkQ/WCXOvT_rTeI/AAAAAAAAAhc/rWbEcrNaKUEp7jWomDWq1U70ldS5jPaugCEw/s1600/image012.jpg)

Screenshots: XenApp 7.9 and Provisioning Services 7.9 test
----------------------------------------------------------

The test setup is upgrade to the following setup follows:

Citrix Backend, one Windows 2012 R2 Server with roles:
*   Citrix Licensing Server 
*   Citrix XenApp Delivery Controller 7.9
*   XenApp Advanced Concurrent licenses
*   Citrix Provisioning Services 7.9
*   Citrix Provisioning Services licenses

Citrix Worker, one PVS Streamed Windows 2012 R2 Session Host:
*   Citrix XenApp 7.9 VDA
*   Citrix Provisioning Services 7.9 Target Device Software
*   VM: Hyper-V Gen2
*   PVS Boot: TFTP UEFI

### Citrix Desktop Delivery Controller 7.9
[![]({{ site.url }}/images/2016-11-11/2016-11-11-image014.jpg)](https://1.bp.blogspot.com/-7kakVSYUyqg/WCXOvuZbcUI/AAAAAAAAAhg/fnIfpguw-h42v1Scyymm6TMSTsS7mnYpQCEw/s1600/image014.jpg)

### Citrix License Administration Console
[![]({{ site.url }}/images/2016-11-11/2016-11-11-image016.jpg)](https://4.bp.blogspot.com/-d9R47qc8XDM/WCXOvvo18jI/AAAAAAAAAhk/7wdrelwNBWQPrWbWnyYG6dmxeOu9pCVWgCEw/s1600/image016.jpg)

### Citrix Provisioning Services Server 7.9
[![]({{ site.url }}/images/2016-11-11/2016-11-11-image018.jpg)](https://2.bp.blogspot.com/-oUwH_9J3P1U/WCXOvqLAEYI/AAAAAAAAAho/z4rP2zIFoAIt4IXOnrVVbVSo6VqvyPhswCEw/s1600/image018.jpg)

### Citrix XenApp 7.9 VDA - Session Host
[![]({{ site.url }}/images/2016-11-11/2016-11-11-image020.jpg)](https://1.bp.blogspot.com/-tok4QjvY034/WCXOvwY80DI/AAAAAAAAAhs/QUBnsmiq6xobIHlo1KDKlLrtktjVz4rUACEw/s1600/image020.jpg)

### User ICA Session to XenApp 7.9 VDA
[![]({{ site.url }}/images/2016-11-11/2016-11-11-image022.jpg)](https://4.bp.blogspot.com/-hKc64ymf0mQ/WCXOv2dNCTI/AAAAAAAAAhw/MvMFxjU7I8oD2l0Lm4Y_wvVHmY2vL14dwCEw/s1600/image022.jpg)

Screenshots: XenApp 7.11 and Provisioning Services 7.11 test
------------------------------------------------------------
The test setup is upgrade to the following setup follows:

Citrix Backend, one Windows 2012 R2 Server with roles:
*   Citrix Licensing Server 
*   Citrix XenApp Delivery Controller 7.11
*   XenApp Advanced Concurrent licenses
*   Citrix Provisioning Services 7.11
*   Citrix Provisioning Services licenses

Citrix Worker, one PVS Streamed Windows 2012 R2 Session Host:
*   Citrix XenApp 7.11 VDA
*   Citrix Provisioning Services 7.11 Target Device Software
*   VM: Hyper-V Gen2
*   PVS Boot: TFTP UEFI

### Citrix Desktop Delivery Controller 7.11
[![]({{ site.url }}/images/2016-11-11/2016-11-11-711_ddc.jpg)](https://3.bp.blogspot.com/-N0fhs1Xy2Nw/WCXS8m35zBI/AAAAAAAAAiE/BLCQqDwgqoojIxvVlMz543j_yh9DYg32QCLcB/s1600/711_ddc.jpg)

### Citrix License Administration Console
[![]({{ site.url }}/images/2016-11-11/2016-11-11-711_lic.jpg)](https://3.bp.blogspot.com/-yREUO03Asmc/WCXTfxEtwcI/AAAAAAAAAiI/_VY3D71LI6g2fw7bylZaruK2iUrT5Q5OQCLcB/s1600/711_lic.jpg)

### Citrix Provisioning Services Server 7.11
[![]({{ site.url }}/images/2016-11-11/2016-11-11-711_pvs.jpg)](https://2.bp.blogspot.com/-lNemqt3reVI/WCXT00L_KKI/AAAAAAAAAiM/rdbr4TOoj_Q7n1K3a3CiNshx2-kw0zMywCLcB/s1600/711_pvs.jpg)

### Citrix XenApp 7.11 VDA - Session Host
[![]({{ site.url }}/images/2016-11-11/2016-11-11-711_vda.jpg)](https://3.bp.blogspot.com/-exdrqusp0aE/WCXUlyszcHI/AAAAAAAAAiQ/X9ACsnUW190iR1hPwdXRPXsUUz9a51kuACLcB/s1600/711_vda.jpg)

### User ICA Session to XenApp 7.11 VDA
[![]({{ site.url }}/images/2016-11-11/2016-11-11-711_user.jpg)](https://4.bp.blogspot.com/-wXqESphVa9E/WCXUzLQs9uI/AAAAAAAAAiU/i5YFIku6riQjqn0O8esGq2PuVks7TeDJwCLcB/s1600/711_user.jpg)