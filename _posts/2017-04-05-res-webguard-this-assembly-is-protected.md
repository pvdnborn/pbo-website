---
layout: post
title:  "RES WebGuard: This assembly is protected by an unregistered version of Eziriz''s '.NET Reactor'!"
date:   2017-04-05 11:48:00 +0100
categories:  Windows 2012 R2, RES One Workspace, Eziriz's, RES, VDA, Virtual Delivery Agent, Citrix, ".NET Reactor", Eziriz, RES WebGuard, WebGuard, RES.WebGuard.WebGuardIE
---

We are about to finish our XenApp 6.5 to XenApp 7.12 project at a customer. In the new Citrix setup, we are using RES One Workspace 2016 for now. An upgrade to RES One Workspace V10 is planned after go live.  
  
One of the last annoying issues on the issue list is the following pop-up: _This assembly is protected by an unregistered version of Eziriz’s “.NET Reactor”!_  

[![]({{ site.url }}/images/2017-04-05/2017-04-05-image001.png)](https://1.bp.blogspot.com/-tq2wGmervBc/WOS8xfqPCQI/AAAAAAAABqM/EnpapkUFhFMFWaAqvHdEHn3Wubik-9rmQCLcB/s1600/image001.png)

This message box doesn’t popup every time, so it was hard to reproduce the pop-up for investigating it’s source. After a long troubleshoot and google queries, the project member “Frank Slomp” has found the solution which I want to share with you.  
  
The source of this Eziriz’s message box is RES One Workspace WebGuard Internet Explorer Add-in (RES.WebGuard.WebGuardIE). When contacting RES-Support they mentioned some other customer cases with the same Eziriz’s .NET Reactor message box. They told us that an upgrade to RES One Workspace V10 should solve this issue (did not test this out yet).  

[![]({{ site.url }}/images/2017-04-05/2017-04-05-image003.png)](https://3.bp.blogspot.com/-N9uahgCv6Rc/WOS7xiJ9VTI/AAAAAAAABqE/0jA2IuH6dREoelnEUBlMoS4BAh8afrWOgCEw/s1600/image003.png)

<!-- /\* Font Definitions \*/ @font-face {font-family:"Cambria Math"; panose-1:2 4 5 3 5 4 6 3 2 4; mso-font-charset:1; mso-generic-font-family:roman; mso-font-format:other; mso-font-pitch:variable; mso-font-signature:0 0 0 0 0 0;} @font-face {font-family:Calibri; panose-1:2 15 5 2 2 2 4 3 2 4; mso-font-charset:0; mso-generic-font-family:auto; mso-font-pitch:variable; mso-font-signature:-536870145 1073786111 1 0 415 0;} /\* Style Definitions \*/ p.MsoNormal, li.MsoNormal, div.MsoNormal {mso-style-unhide:no; mso-style-qformat:yes; mso-style-parent:""; margin:0cm; margin-bottom:.0001pt; mso-pagination:widow-orphan; font-size:12.0pt; font-family:Calibri; mso-ascii-font-family:Calibri; mso-ascii-theme-font:minor-latin; mso-fareast-font-family:Calibri; mso-fareast-theme-font:minor-latin; mso-hansi-font-family:Calibri; mso-hansi-theme-font:minor-latin; mso-bidi-font-family:"Times New Roman"; mso-bidi-theme-font:minor-bidi; mso-fareast-language:EN-US;} .MsoChpDefault {mso-style-type:export-only; mso-default-props:yes; font-family:Calibri; mso-ascii-font-family:Calibri; mso-ascii-theme-font:minor-latin; mso-fareast-font-family:Calibri; mso-fareast-theme-font:minor-latin; mso-hansi-font-family:Calibri; mso-hansi-theme-font:minor-latin; mso-bidi-font-family:"Times New Roman"; mso-bidi-theme-font:minor-bidi; mso-fareast-language:EN-US;} @page WordSection1 {size:612.0pt 792.0pt; margin:72.0pt 72.0pt 72.0pt 72.0pt; mso-header-margin:36.0pt; mso-footer-margin:36.0pt; mso-paper-source:0;} div.WordSection1 {page:WordSection1;} -->  

Since all the testing by the customer is done on RES One Workspace 2016 we don’t want to upgrade to V10 one day before go live. So I have another solution for users who don't want to upgrade to V10 yet. The customer had website security configured in “Learning mode”, so after disabling this feature the Eziriz’s .NET message box is gone (see Dutch screenshot).

[![]({{ site.url }}/images/2017-04-05/2017-04-05-image005.png)](https://1.bp.blogspot.com/-hUCs1ep2ero/WOS8kYUnUsI/AAAAAAAABqI/TttURS3j5doi2L0Xfff22-P4W12dpVmlgCEw/s1600/image005.png)