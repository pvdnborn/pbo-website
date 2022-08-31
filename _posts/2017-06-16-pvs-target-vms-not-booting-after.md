---
layout: post
title:  "Fix: PVS Target VMs not booting after upgrade Citrix XenServer 6.5 to Citrix XenServer 7.x"
date:   2017-06-16 14:03:00 +0100
categories:  Citrix XenServer 6.5, Provisioning Services, Citrix XenServer 7.1, Roling pool upgrade, Citrix XenServer, Citrix XenServer 7.x, has-vendor-device, PVS Target devices, Upgrade
---

Last week [@R\_Kossen](https://twitter.com/R_Kossen) and I ([@pvdnborn](https://twitter.com/pvdnborn)) had a project for upgrading a Citrix XenDesktop 7.8 with Citrix XenServer 6.5 to Citrix XenDesktop 7.13 with Citrix XenServer 7.1. During the Citrix XenServer 6.5 to Citrix XenServer 7.1 upgrade we had an issue with the PVS Target devices which I want to share with you.  
  
Before we started the upgrade of XenDesktop and XenServer, we upgraded all the PVS images with: Citrix XenDesktop 7.13 VDA, Citrix Provisioning Services 7.13 Target Device software and XenTools/Management agent for XenServer 7.1. When we successfully finished the upgrade of XenServer 6.5 to XenServer 7.1, we had problems starting all of our VDI and Session Host PVS Target Devices VMs. We had the following issues:   

*   Some PVS Targets didn’t boot at all: “BNIStack failed: network stack could not be initialized”
*   Some PVS Targets did boot very slow and had Realtek network adapters in Windows Device management.
    *   XenTools/Management agent didn’t initialize.

During the troubleshoot I realized that this behavior looks very similar to virtual hardware differences between the image and target VM. As preparation we had upgraded the PVS-images in the test environment, which has one XenServer 7.1 host. We decided to export a target device VM from this test XenServer 7.1 pool and import it to our upgraded XS6.5 to XS7.1 production pool. The image with the “BNIStack failed” error was now booting perfectly on the upgraded production host. Even the XenTools/Management agent initialized and we have a XenServer PV Network device instead of the Realtek NIC. Due to this tests we agreed that there is a difference in the virtual hardware configuration between the VMs which is causing boot issues and the imported VM from the XenServer 7.1 test pool.  
  
We noticed a difference of the “Virtualization State” in the XenCenter GUI of the VM which didn’t boot and the VM which booted successfully.  
  
The Virtualization state of the VM with boot issues is “Not able to receive updates from Windows Update”[![]({{ site.url }}/images/2017-06-16/2017-06-16-image002.jpg)](https://1.bp.blogspot.com/-SviWOQf_SNw/WUPHSCSw7iI/AAAAAAAABso/Vt28PVKm7cc_jG7UO1QGN_IJjFMgXCyKgCEwYBhgL/s1600/image002.jpg)  
   
The Virtualization state of the VM starting correctly is “Able to receive updates from Windows Update”  

[![]({{ site.url }}/images/2017-06-16/2017-06-16-image004.jpg)](https://4.bp.blogspot.com/-SMkIcfjLhWU/WUPHVP3D1xI/AAAAAAAABss/w4LklITPdiY7oe4P4EcgC91mEd4DDVmIgCEwYBhgL/s1600/image004.jpg)
 
When connecting with SSH to the pool master, we’ve also noticed a difference between the “hardware-platform-version” and “has-vendor-device” per VM. To get all the properties of the VM use the following command: “**xe vm-param-list uuid=<uuid of VM>**”  
  
The VM-Parameter list of the VM with boot issues:  

[![]({{ site.url }}/images/2017-06-16/2017-06-16-image006.jpg)](https://4.bp.blogspot.com/-Xz9H27SQj9Q/WUPHWWGiagI/AAAAAAAABsw/tFcuOow9o6sr_P8nHccl8FIstn7b7mITACEwYBhgL/s1600/image006.jpg)

The VM-Parameter list of the VM starting correctly:  

[![]({{ site.url }}/images/2017-06-16/2017-06-16-image008.jpg)](https://1.bp.blogspot.com/-ThfgaKhjZ5Q/WUPHXg6BZDI/AAAAAAAABs0/YNjQAHcR_BYJI6-qz-PPfXqqi9zqSamewCEwYBhgL/s1600/image008.jpg)

To fix the boot issue, the “**has-vendor-device = true**” value is needed for all the VMs with boot issues after the XenServer 6.5 to XenServer 7.x upgrade. The “hardware-platform-version” and the “Virtualization state” is also modified by updating the “has-vendor-device” property.  
  
Set the “has-vendor-device” property with the following command: “**xe vm-param-set has-vendor-device=true uuid=<UUID of VM>**”  
[![]({{ site.url }}/images/2017-06-16/2017-06-16-image010.png)](https://2.bp.blogspot.com/-vNja7Asxfu8/WUPHYoWSasI/AAAAAAAABtA/J7DVq1Ldy3EAqVknCprwWMIfu7oQbrJtQCEwYBhgL/s1600/image010.png)  
To get a comma separated list UUID’s of all the VMs and Templates in the pool. Execute the following command on the pool master: “**xe vm-list –minimal**”  

[![]({{ site.url }}/images/2017-06-16/2017-06-16-image012.png)](https://3.bp.blogspot.com/-KbyQgB0Q6hk/WUPHZxxWnmI/AAAAAAAABtA/DA3UUtjCmsEzQ_3cSUf8xOyPpyctcb_LwCEwYBhgL/s1600/image012.png)

Use the comma separated UUID list to set the “has-vendor-device = true” property to all the VMs in the pool.