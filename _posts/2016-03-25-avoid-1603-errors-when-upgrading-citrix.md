---
layout: post
title:  "Avoid 1603 errors when upgrading Citrix StoreFront 2.x to Citrix StoreFront 3.5"
date:   2016-03-25 12:54:00 +0100
categories:  StoreFront, Upgrade
---

Today I was facing a StoreFront 2.1 to 3.5 upgrade. My experience with upgrading StoreFront is that it will fail most times (1603 error). So I’ve learned some fixes that I want to share with you to make the upgrade of StoreFront easier.

Log files of StoreFront upgrade are located in **%windir%\temp\StoreFront**

**Customer load balanced StoreFront set-up**
--------------------------------------------

*   KEMP Network load balancer for http, https round robin load balancing between StoreFront servers
*   StoreFront Server Group
    *   pbo-sf01: Windows 2012, StoreFront 2.1
    *   pbo-sf02: Windows 2012, StoreFront 2.1

[![]({{ site.url }}/images/2016-03-25/2016-03-25-image002.png)](https://1.bp.blogspot.com/-vyU4ZIEkfAU/VvUWAoo2MlI/AAAAAAAAAV4/I4bngNgczBoBTZwgTqevnOwEFY__msfJQ/s1600/image002.png)

The upgrade plan
----------------

The upgrade plan looks straight forward, VMware snapshots are used as fallback scenario:  
1\. Disable **pbo-sf02** from load balancing  
2\. Shutdown **pbo-sf02**  
3\. Take snapshots of both StoreFront servers (online quiesce snapshot of **pbo-sf01**)  
4\. Start **pbo-sf02** and upgrade **pbo-sf02** to StoreFront 3.5  
  

[![]({{ site.url }}/images/2016-03-25/2016-03-25-image004.png)](https://2.bp.blogspot.com/-_Q7OK-cF1kM/VvUWAjRdGUI/AAAAAAAAAV4/Aj7Qpm203HMNzw3y-kIIflTeX0p3zylhg/s1600/image004.png)

  
5\. Test StoreFront upgrade by connecting (thin)clients directly to upgraded **pbo-sf02** StoreFront server  
6\. Enable **pbo-sf02** in load balancing and disable load balancing for **pbo-sf01**  
7\. Shutdown **pbo-sf01**  
8\. Take snapshots of both StoreFront servers (online quiesce snapshot of **pbo-sf02**)  
9\. Start **pbo-sf01** and upgrade **pbo-sf01** to StoreFront 3.5  

[![]({{ site.url }}/images/2016-03-25/2016-03-25-image006.png)](https://4.bp.blogspot.com/-DAIjaz7gWko/VvUWAutrA9I/AAAAAAAAAV4/chyeqZgiWO8jAysnoYGvPzZKKiy_11jFw/s1600/image006.png)

  
10\. Test StoreFront upgrade by connecting (thin)clients directly to upgraded **pbo-sf01** StoreFront server  
11\. Test StoreFront synchronization by clicking "Propagate changes"  
12\. Restore load balancing  

[![]({{ site.url }}/images/2016-03-25/2016-03-25-image008.png)](https://1.bp.blogspot.com/-G34J68dHBiM/VvUWBDKuv3I/AAAAAAAAAV4/Y0t9eWdz7VwcY0Abqe9PrsuXQJ7YH1bYA/s1600/image008.png)

**Locked StoreFront files are causing StoreFront upgrade to fail**
------------------------------------------------------------------

During upgrade to StoreFront 3.5 you must ensure that the existing StoreFront files are not in use or the upgrade will fail:  

*   Make sure there are no client connections to the StoreFront server before and during upgrade
*   Disable all monitoring services on the StoreFront server, periodically monitoring checks can lock files during upgrade
    *   Microsoft Monitoring Agent (HealthService)
    *   Comtrade service
    *   Nagios NSClient++
*   Temporary disable all agentless remote monitoring checks to the StoreFront server:
    *   SCOM: Put server in maintenance mode
    *   Nagios: Disable checks for during upgrade period
*   Temporary disable virus scan on the StoreFront server
    *   On access scanner
    *   Web reputation
*   Reboot server before upgrade to ensure all active sessions are closed

Virtual path /Citrix/StoreWeb is already in use
-----------------------------------------------

During upgrades I also faced the following error:  
  
_**An error occurred configuring the installation: System.InvalidOperationException: Physical path: C:\\inetpub\\wwwroot\\Citrix\\StoreWeb associated with Virtual path: /Citrix/StoreWeb is already in use.**_  
  
When troubleshooting this error I did not have the path **C:\\inetpub\\wwwroot\\Citrix\\StoreWeb** on StoreFront server **pbo-sf02**, the upgrade process moved it successfully to **C:\\ProgramData\\Citrix**. So why I’m facing this error? The answer is quite simple. At the end of the upgrade process, the Citrix StoreFront website folders/applications (i.e. WebReceiver) are copied back to Internet Information Services website. I figured out that the upgrade process is using the StoreFront Server Group base URL for this. The base URL is configured to **desktop.pbo.local** which is the DNS A-record of the KEMP load balancer. Because the load balancing is disabled to the server **pbo-sf02** I‘m upgrading, the upgrade process is contacting the other **pbo-sf01** StoreFront server through the load balancer **desktop.pbo.local**. On the **pbo-sf01** the virtual path /Citrix/StoreWeb exist so the error is legitimate. I can also imagine that if you get load balanced (i.e. by round robin) to the wrong server during upgrade, the same error occurs.  

[![]({{ site.url }}/images/2016-03-25/2016-03-25-image010.png)](https://1.bp.blogspot.com/-fpL6g9-6BwQ/VvUWBPY01XI/AAAAAAAAAV4/8rHOpsiHHpUOZAPSDLmXcVBiMJHv02PGg/s1600/image010.png)
  
To bypass this error you can add a record for the load balanced address to the local hosts file of the StoreFront server to force traffic to the local IPv4 address. This registration should be removed after the upgrade completes. Windows host file can be found: **C:\\Windows\\System32\\drivers\\etc\\hosts**, add line (example upgrade **pbo-sf02**):  
   
_**10.175.201.118    desktop.pbo.local**_  

[![]({{ site.url }}/images/2016-03-25/2016-03-25-image012.png)](https://1.bp.blogspot.com/-lyozigy2rvk/VvUWBW2RLRI/AAAAAAAAAV4/Ym1UXfprNH8Zd1BQEpZDlXam4C49jFDuw/s1600/image012.png)

**Thumbs up**
-------------

If you have had other issues upgrading StoreFront, please share the fixes with me.  

[![]({{ site.url }}/images/2016-03-25/2016-03-25-image013.png)](https://1.bp.blogspot.com/-Ak3l9yKR2zA/VvUWBqzIg-I/AAAAAAAAAV4/H89JB2S59h8wVYrBtKew8GzRIjpuCQ65g/s1600/image013.png)