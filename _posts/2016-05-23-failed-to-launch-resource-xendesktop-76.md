---
layout: post
title:  "Failed to launch the resource ‘XenDesktop 7.6 $S1-1’as it was not found.” After upgrade to StoreFront 3.5"
date:   2016-05-23 15:04:00 +0100
categories:  Citrix, XenDesktop, StoreFront, Upgrade, XenApp
---

This weekend I was upgrading Citrix StoreFront 2.6 to Citrix StoreFront 3.5. The in-place-upgrade was working out of the box and finishes successfully at once! When testing the upgrade by logging in with ThinClients, we cannot connect to any XenApp 7.6 resource (PubApps and PubDesktops). All the XenApp servers are in a Registerd state to the Desktop Delivery Controller. When investigating the eventviewer of the StoreFront servers I found the following Warning message: **“Failed to launch the resource ‘XenDesktop 7.6 $1-1’ as it was not found”**  

[![]({{ site.url }}/images/2016-05-23/2016-05-23-image001.png)](https://1.bp.blogspot.com/-9lnwQjqVw0w/V0L-pvt1gHI/AAAAAAAAAZs/6_yiE7JJ7uEuFoIRAE-QudspAnyAF-MFQCKgB/s1600/image001.png)

After some investigation I’ve found out that Citrix StoreFront 3.5 cannot handle spaces and dot characters in the Display Name of Controllers. My customer has **XenDesktop 7.6** as display name containing spaces and a dot:  

[![]({{ site.url }}/images/2016-05-23/2016-05-23-image003.png)](https://1.bp.blogspot.com/-vAKqAQYXYpE/V0L-ppuW7zI/AAAAAAAAAZs/eccUVobXgmUJvDzUEBQgnlut9F0RlqnFQCKgB/s1600/image003.png)

Renaming the Delivery Controller Display name to **XenDesktop** solved the problem:  

[![]({{ site.url }}/images/2016-05-23/2016-05-23-image005.png)](https://3.bp.blogspot.com/-fPAFG2zW-Fg/V0L-pqR8uGI/AAAAAAAAAZs/oKF4Qu6vJ7o2uSuNWupkNCWGHVyGVb87gCKgB/s1600/image005.png)

Attention is needed for users which have added resources in their Citrix Receiver. After renaming the Delivery Group Display name, they should add the resource again to their receiver.