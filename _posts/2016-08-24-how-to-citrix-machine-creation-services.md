---
layout: post
title:  "How to: Citrix Machine Creation Services and Microsoft Azure Resource Manager"
date:   2016-08-24 17:24:00 +0100
categories:  Windows 2012 R2, Machine Creation Services, Citrix MCS, Citrix, MCS, Azure, Azure Classic, XenDesktop and XenApp Services, Citrix Cloud, CWC, How to, XenApp
---

In January of this year I wrote a how-to blog about Machine Creation Services and Microsoft Azure Classic. Citrix has updated their Machine Creation Services to support Microsoft Azure Resource manager. At the time of writing this MCS update is only available for customers who are using “Citrix Cloud | XenDesktop and XenApp Services”. Those who have a traditional on-prem XenDesktop 7.7, 7.8 or 7.9 set-up only have the option for Microsoft Azure Classic at the moment.  
  
This blog is a step-by-step guide about how to provision machines in Microsoft Azure using Machine Creation Services. This blog describes the creation of a hosting connection to Microsoft Azure and making a new Machine Catalog with three new Windows 2012 R2 Session Host machines based on the Golden Master.  

**Citrix XenDesktop configuration**

For this guide I have setup a basic Citrix Cloud setup with IaaS VMs in Microsoft Azure (Resource manager):

*   Citrix Cloud | XenDesktop and XenApp Services
*   Citrix Cloud Connector Virtual Machine
*   Established connection between Citrix Cloud and Citrix Cloud Connection (Resource location and Domain connection)

![]({{ site.url }}/images/2016-08-24/2016-08-24-image001.png)

**Master Image Configuration**

The Master image, used in this guide, is configured as follows:

*   Azure Resource Manager Virtual Machine A2
*   Azure virtual network name "VN\_172.16.0.0-24"and Subnet name "SN\_172.16.1.0-24"
*   Windows 2012 R2 Session Host
*   Citrix XenDesktop 7.9 VDA
*   LibreOffice 5.2
*   Notepad++
*   VLC Media player

**Creating Azure Hosting connection in Citrix Cloud | XenDesktop and XenApp Services**  

1. Browse to [https://portal.azure.com](https://portal.azure.com/) and click **Subscriptions**. Write down your Azure **Subscription ID**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image002.png)  
  
2. Browse to [https://citrix.cloud.com](https://citrix.cloud.com/) and go to **XenApp and XenDesktop Services**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image003.png)  
  
3. Click **Manage** and go to **Hosting**. Click Add **Connection and Resources**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image004.png)

4. Select connection type **Microsoft Azure**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image005.png)

5. Enter your Azure **subscription ID** (step 1) and give the **connection** a **name**. Then click **Create new..**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image006.png)  

6. Login with your Microsoft Account which has access to your Microsoft Azure subscription  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image007.png)  

7. Verify the login. If you have a problem (like me) sending a security code, edit your Microsoft account for mobile app “Microsoft Account” verification ([https://account.live.com/proofs/Manage](https://account.live.com/proofs/Manage))  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image008.png)  
 
8. **Accept** Citrix XenDesktop permissions  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image009.png)  

9. Click **Next**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image010.png)  
  
10. Select the Azure region for this hosting connection and click **Next**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image011.png)  
  
11. Select your virtual network and subnet, click **Next**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image012.png)  
  
12. Click **Finish**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image013.png)  
  
13. The hosting connection to Microsoft Azure Resource Manager is completed  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image014.png)  
  
**Create new Machine Catalog**  
Like using Citrix Machine Creation Services with on-premises Hypervisor we need to install a master VM and shut it down. There is no need to create a snapshot (hypervisor) or capture (azure classic). In this example I’ve used master virtual machine **AzureCTXGM**.  
1. Install your Golden Master VM with software and Citrix VDA. When finished shut the machine down and power off (deallocated)  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image015.png)  
  
2. Browse to **Citrix Cloud | XenApp and XenDesktop Services** and go to **Machine Catalogs**. Click **Create Machine Catalog**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image016.png)  
  
3. Select type of Operating System (in this example Server OS) and click **Next**![]({{ site.url }}/images/2016-08-24/2016-08-24-image017.png)  
  
4. Select **Machines that are power managed** and select **Citrix Machine Creation Services**, click **Next**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image018.png)

5. Select the **VHD** of the Golden Master VM and click **Next**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image019.png)

6. Select the destination storage type for your provisioned virtual machines and click **Next**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image020.png)  

7. Select the **amount of machines** (i.e. 3) to create and select the **Microsoft Azure Machine size** (i.e. Standard\_A2). Click **Next**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image021.png)  

8. Select the network card and associate it with the Azure Subnet name for your Citrix Machines (i.e. SN\_172.16.1.0-24), click **Next**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image022.png)

9. Select **Create new Active Directory accounts**. Then select the **OU** for the new machines. And fill in an **Account naming scheme** for the new machines (use ## for auto increment numbers)  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image023.png)  

10. Enter your domain account credentials, and click **next** (this account is used to create the computer accounts)  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image024.png)

11. A summary is displayed. Fill in a **Machine Catalog name** and a **Machine Catalog description for administrators** and click **Finish**  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image025.png)

12. The Machine Catalog is copying the master image to a base disk  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image026.png)  

13. The Virtual Machines are created  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image027.png)
![]({{ site.url }}/images/2016-08-24/2016-08-24-image028.png)  

14. A new storage account is created on Azure![]({{ site.url }}/images/2016-08-24/2016-08-24-image029.png)  
15. While MCS is deploying the virtual machines, a preparation VM with VHD is temporary created on Azure  
![]({{ site.url }}/images/2016-08-24/2016-08-24-image030.png)

16. When MCS finishes, the following VHDs are created for 3 VMs (ARM-XDSH-##)
![]({{ site.url }}/images/2016-08-24/2016-08-24-image031.png)  

You are now able to create a Delivery Group.
*   For on-premisses Delivery Controller, read this blog [http://patrickvandenborn.blogspot.nl/2016/01/citrix-xendesktop-77-machine-creation\_76.html](http://patrickvandenborn.blogspot.nl/2016/01/citrix-xendesktop-77-machine-creation_76.html)
*   For Citrix Cloud | XenDesktop and XenApp Services, read this blog [http://patrickvandenborn.blogspot.nl/2016/03/how-to-citrix-workspace-cloud-apps-and.html](http://patrickvandenborn.blogspot.nl/2016/03/how-to-citrix-workspace-cloud-apps-and.html)