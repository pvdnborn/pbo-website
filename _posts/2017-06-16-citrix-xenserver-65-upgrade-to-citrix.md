---
layout: post
title:  "Be patient: Citrix XenServer 6.5 upgrade to Citrix XenServer 7.1 stuck at 72% for an hour"
date:   2017-06-16 13:47:00 +0100
categories:  Citrix XenServer 7.1, 7.2, XenServer 7.1, stuck, Upgrade, Citrix, Roling pool upgrade, XenServer 6.5, 72 percent, XenServer, XenServer 7.x, Citrix XenServer 7.x
---

Last week [@R\_Kossen](https://twitter.com/R_Kossen) and I ([@pvdnborn](https://twitter.com/pvdnborn)) had a project for upgrading a Citrix XenDesktop 7.8 with Citrix XenServer 6.5 to Citrix XenDesktop 7.13 with Citrix XenServer 7.1. During the Citrix XenServer 6.5 to Citrix XenServer 7.1 upgrade we had an issue which I want to share with you.  
  
For the upgrade to Citrix XenServer 7.1 we used a “Rolling Pool Upgrade” with the XenServer media on a NFS Export. During the Rolling Pool Upgrade the “Completing installation…” stuck at 72%. A “Rolling Pool Upgrade” is always starting with the pool master, so I was worried about this and expected the pool master upgrade will fail.  

[![]({{ site.url }}/images/2017-06-16/2017-06-16-image002.png)](https://2.bp.blogspot.com/-ZUBkZRL0I4A/WUPD-6kP1ZI/AAAAAAAABsc/PttBdhv6kXM0gGbpU0ZA7cOCq_Ery2tZwCLcBGAs/s1600/image002.png)
  
When switching to another TTY (by pressing ALT+F3), I’ve noticed progress in changing permissions (chmod) of messages files on the filesystem. I think these messages are files of the XenServer performance data which are migrated from the old XenServer 6.5 partition to the new XenServer 7.1 partition. The XenServer hosts of this customer are in production for years now, so there are a lot of messages to “upgrade”. After waiting for about one hour per host the “Rolling Pool Upgrade” completed successful.  
  
If you are experiencing this issue, be patient and press ALT+F3 to see the progress at this 72% stage. Depending on the amount of messages and the production time of your host the duration of this process is between one second and ages. We waited for 1 hour per host and 9 hours for all the hosts in the pool.  
  
At the moment, I didn’t figure out yet how to clean-up these messages in XenServer 6.5 before upgrading to XenServer 7.x.