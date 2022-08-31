---
layout: post
title:  "Fixed: Internet Explorer “MfApHook64.dll” crash with RES One Workspace 2016 and Citrix 7.12 VDA"
date:   2017-04-05 11:09:00 +0100
categories:  Citrix, RES One Workspace, XenApp 7.12, XenDesktop, RES, VDA, XenDesktop 7.12, XenApp
---

Today I was working on a XenApp 6.5 with RES One Workspace 2015 to XenApp 7.12 with RES One Workspace 2016 project. During this project I had crashes with Internet Explorer on Windows 2012 R2 with crashing module “MfApHook64.dll” version “7.12.0.211”. Screenshot in Dutch:  
  
[![]({{ site.url }}/images/2017-02-10/2017-02-10-error_mfaphook64dll.jpg)](https://2.bp.blogspot.com/-1FHNjH-H1Ys/WJ2fPmpdyBI/AAAAAAAABmc/bDSgH3MvJXMIyYrrQdjLpDAgAH6RI9FBACEw/s1600/error_mfaphook64dll.jpg)  
After troubleshooting I figured out that this problem is caused by RES One Workspace 2016. Somehow RES One Workspace is creating 64-bit Internet Explorer shortcuts instead of the default 32-bit Internet Explorer shortcuts. Due to this, Internet Explorer 64-bit is starting instead of Internet Explorer 32-bit. After contact with RES Support (very good helpdesk) I was facing an issue with RES One Workspace and the Citrix 7.12 VDA. Something is changed within the Citrix 7.12 VDA code. I think the new code for feature “HTML5 video redirection for internal web sites” is breaking RES One Workspace.  
  
RES does have a private fix for this issue, but isn’t suitable for production environments at the moment (10-02-2017). So the only workaround is downgrade the VDA to 7.11 until RES is releasing a public fix for this issue.     
  
**Update - 20 March 2017:** RES Software released V10 of RES One Workspace. This version fixes the issue.