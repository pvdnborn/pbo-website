---
layout: post
title:  "How to: Citrix XenDesktop 7.7 Machine Creation Services and Microsoft Azure"
date:   2016-01-31 09:48:32 +0100
categories:  Citrix , XenDesktop 7.7 , Machine Creation Services , MCS , XenDesktop , Azure , Azure Classic , Citrix MCS , XenApp 7.7 , How to , XenApp
---


### Citrix XenDesktop 7.7 configuration
For this guide I have setup a basic Citrix XenDesktop 7.7 installation in Azure:

Active Directory Domain Controller

*   SQL-Server
*   Desktop Delivery Controllers
*   StoreFront servers
*   NetScalers

I tried to use as much as Azure SaaS services for this set-up, but I didn’t used one. Everything is configured as Azure IaaS virtual machines. (For example: SQL Databases in Azure only allows SQL Authentication. XenDesktop only uses AD-integrated authentication)  

### Master Image configuration
The Master image, used in this guide, is configured as follows:

*   Azure Classic Virtual Machine A2
*   Azure virtual network name “AMS\_Network” and Subnet name “VMSubnet1”
*   Windows 10 Enterprise
*   Citrix XenDesktop 7.7 VDA
*   Office 2013
*   Notepad++

### Azure and Azure Classic
At the moment two types of virtual machines can deployed in Azure: “Virtual machines” and “Virtual machines (classic)”.  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image001.png)]({{ site.url }}/images/2016-01-31/2016-01-31-image001.png)  
  
The classic virtual machines are managed originally by the old Azure management portal (manage.windowsazure.com) and the new virtual machines are managed by the new Azure management portal (portal.azure.com). The new portal can also manage classic virtual machines. It is good to understand that Citrix Machine Creation Services only provisions Microsoft Azure Classic Virtual machines. New style virtual machines is not possible at the moment.  
  
Classic [manage.windowsazure.com](http://manage.windowsazure.com/) portal:  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image002.png)](http://4.bp.blogspot.com/-DvuByVH8OO8/Vq4P_his7xI/AAAAAAAAAJ0/TknWOAH6Ut4/s1600/image002.png)  
  
New [portal.azure.com](http://portal.azure.com/)  

[![]({{ site.url }}/images/2016-01-31/2016-01-31-image003.png)](http://1.bp.blogspot.com/-aSLxDasPBK0/Vq4P_iIlngI/AAAAAAAAAKM/6I5LbM3qxa4/s1600/image003.png)

### Create Azure Hosting connection in Citrix Studio
1\. Goto [https://manage.windowsazure.com/publishsettings/index](https://manage.windowsazure.com/publishsettings/index) and download your **publishsettings** file. (If prompted for login, login with your Azure credentials:[![]({{ site.url }}/images/2016-01-31/2016-01-31-image004.png)](http://3.bp.blogspot.com/-E5mvaw8Y2yw/Vq4P_uzQLDI/AAAAAAAAAOA/8JQxBggHlSc/s1600/image004.png)  
  
2\. Open Citrix Studio:  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image005.png)]({{ site.url }}/images/2016-01-31/2016-01-31-image005.png)  
  
3\. Go to **Configuration --> Hosting** and click **Add Connection and Resources**  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image006.png)](http://1.bp.blogspot.com/-DqZQOycYSmc/Vq4QALEMn8I/AAAAAAAAAM4/mE_vA6eT-r8/s1600/image006.png)
  
4\. Choose connection type: **Microsoft Azure Classic** and click on **Import**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image007.png)](http://4.bp.blogspot.com/-0Djl5dor38g/Vq4QAPRELGI/AAAAAAAAAOA/w5GM35VzRB0/s1600/image007.png)  
  
5\. Select the **publishsettings** file downloaded at step 1[![]({{ site.url }}/images/2016-01-31/2016-01-31-image008.png)](http://2.bp.blogspot.com/-9OwRIeWmdm8/Vq4QARi4H1I/AAAAAAAAAM8/09f7wsrltYk/s1600/image008.png)  
  
6\. Your Azure Subscription ID will appear, give the connection a name like **MSAzureConnection** and choose **Machine Creation Services**. Click Next:  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image009.png)](http://3.bp.blogspot.com/-O2N2wio-kDg/Vq4QAswEJFI/AAAAAAAAAOA/sYvoqYhN25c/s1600/image009.png)

  
7\. Select an Azure region where the virtual machines will be stored and click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image010.png)](http://2.bp.blogspot.com/-oaftsIobBWE/Vq4QAh1ZoAI/AAAAAAAAAOA/kwd4eUaePEM/s1600/image010.png)  
  
8\. Select the Azure Virtual Network and select the Azure Subnet you want to use for your virtual machines, click **Next**  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image011.png)](http://3.bp.blogspot.com/-tQC_NEpuEng/Vq4QAn0me9I/AAAAAAAAANY/GatgkYRSB0s/s1600/image011.png)

9\. Click **Finish** to create the Azure connection[![]({{ site.url }}/images/2016-01-31/2016-01-31-image012.png)](http://4.bp.blogspot.com/-mJ4SrPyoeWk/Vq4QA7PDUiI/AAAAAAAAANI/DSFMU8uUsRA/s1600/image012.png)  
  
10\. The connection to Azure is created:  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image013.png)](http://2.bp.blogspot.com/-1F7Nh7AcuxU/Vq4QBMq6uFI/AAAAAAAAALA/wRqT32HQgQQ/s1600/image013.png)  

### Create new Machine Catalog
Like using Citrix Machine Creation Services with on-premises Hypervisor we need to install a master VM, shut it down and create a snaphot. On Microsoft Azure we need to create a “Capture”. This Capture is used to create new Virtual Machines. In this example I’ve used master virtual machine **AZ-W10-GM**.  

1\. Go to **Virtual Machines** on **manage.windowsazure.com**, select the Master VM and click **Capture**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image014.png)](http://1.bp.blogspot.com/-pd0PbJFrlK8/Vq4QBPwzRwI/AAAAAAAAANM/s9vJCO7YPAg/s1600/image014.png)  
  
2\. Give the Capture an _Image name_ and _Image Description_ and click **V** (Do not select _I have run Sysprep on the virtual machine_)  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image015.png)](http://2.bp.blogspot.com/-SYSAtxhx88w/Vq4QBWM4yyI/AAAAAAAAAOA/98tcDzN1KBU/s1600/image015.png)
  
3\. Wait until the captured image is in the available state[![]({{ site.url }}/images/2016-01-31/2016-01-31-image016.png)](http://3.bp.blogspot.com/-o8Mm0hWsRyg/Vq4QBWTZyUI/AAAAAAAAANQ/UQsfgZS1W-w/s1600/image016.png)  
  
4\. Open Citrix Studio  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image005.png)]({{ site.url }}/images/2016-01-31/2016-01-31-image005.png)  

5\. Click **Machine Catalogs** and click **Create Machine Catalog**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image017.png)](http://1.bp.blogspot.com/-46wvZby8dJI/Vq4QBpl_igI/AAAAAAAAAOA/CJVjA6hfx2I/s1600/image017.png)  

6\. Select type of Operating system (in this example Desktop OS) and click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image018.png)](http://4.bp.blogspot.com/-wlZc3PetPtA/Vq4QBjLV7SI/AAAAAAAAAOA/TvHy48m0EM8/s1600/image018.png)  
  
7\. Select **Machines that are power managed** and select **Citrix Machine Creation Services,** click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image019.png)](http://3.bp.blogspot.com/-x3XTJpbnvmc/Vq4QB3_DOVI/AAAAAAAAAOA/MNPxz6Gp3oU/s1600/image019.png)  
  
8\. Choose your type of Desktop Experience and click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image020.png)](http://1.bp.blogspot.com/-pRbhhyJSESE/Vq4QB9lL-qI/AAAAAAAAANc/KtVbYobdhw8/s1600/image020.png)  
  
9\. Select the **Capture** of the Master Image created at step 1, than select the **7.7 VDA version** and click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image021.png)](http://1.bp.blogspot.com/-1TWlFRwpe54/Vq4QCFnyZrI/AAAAAAAAAMo/4s5uq0u_G3Y/s1600/image021.png)  
  
10\. Select the amount of machines (i.e. 3) to create and select the Microsoft Azure Machine size (i.e. Basic\_A2). Click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image022.png)](http://2.bp.blogspot.com/-LGcjUQROVW8/Vq4QCOzLywI/AAAAAAAAAOA/Yid1ri0kx4Y/s1600/image022.png)  
  
11\. Select the network card and associate it with the Azure Subnet name for your Citrix Machines (i.e. VMSubnet1), click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image023.png)](http://3.bp.blogspot.com/-mWM-55rNWXY/Vq4QCev6ZOI/AAAAAAAAANg/chpiolhT6s8/s1600/image023.png)  
  
12\. Select **Create new Active Directory accounts**. Then select the **OU** for the new machines. And fill in an **Account naming scheme** for the new machines (use ## for auto increment numbers)[![]({{ site.url }}/images/2016-01-31/2016-01-31-image024.png)](http://2.bp.blogspot.com/-0_OUrqHoRB8/Vq4QCokzgmI/AAAAAAAAAOA/hICdbS6343g/s1600/image024.png)  
  
13\. A summary is displayed. Fill in a **Machine Catalog name** and a **Machine Catalog description for administrators** and click **Finish**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image025.png)](http://4.bp.blogspot.com/-v9hrCmmdF70/Vq4QCgAHcvI/AAAAAAAAAOA/BCcdy1ss6h8/s1600/image025.png)  
  
14\. The **Machine Catalog** is copying the master image (capture) to a capture like**Win10-Azure-MCS-baseDisk-128ti**  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image026.png)]({{ site.url }}/images/2016-01-31/2016-01-31-image026.png)  

[![]({{ site.url }}/images/2016-01-31/2016-01-31-image027.png)](http://2.bp.blogspot.com/-wkHujP5WnBM/Vq4QDE-2f_I/AAAAAAAAAOA/LMBXXC-a6uo/s1600/image027.png)

15\. Like the on-premises MCS a Preparation VM (**Preparati-lg5gl**) is starting and prepares the MCS basedisk for deployment[![]({{ site.url }}/images/2016-01-31/2016-01-31-image028.png)](http://1.bp.blogspot.com/-I58Ntg-acVI/Vq4QDG1It6I/AAAAAAAAANo/sm3QBo6x_WM/s1600/image028.png)  
  
16\. After preparation, MCS is creating the MCS virtual machines  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image029.png)]({{ site.url }}/images/2016-01-31/2016-01-31-image029.png)

[![]({{ site.url }}/images/2016-01-31/2016-01-31-image030.png)](http://1.bp.blogspot.com/--AiKMQ816f8/Vq4QDeS4HxI/AAAAAAAAAOA/y9tdAaMilK8/s1600/image030.png)
  
17\. Each machine gets an **Operating System** and **Identity disk**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image031.png)](http://4.bp.blogspot.com/-BVrw6s9fg7Q/Vq4QDnu3IxI/AAAAAAAAANs/s2pjSf8ilXM/s1600/image031.png)  
  
18\. The Microsoft Azure MCS Machine Catalog is created[![]({{ site.url }}/images/2016-01-31/2016-01-31-image032.png)](http://4.bp.blogspot.com/-m5xqQSxicZM/Vq4QDtYYRyI/AAAAAAAAANE/C51dLgDahog/s1600/image032.png)  
  
19\. Total duration of deployment:[![]({{ site.url }}/images/2016-01-31/2016-01-31-image033.png)](http://4.bp.blogspot.com/-W04F-sd9bWc/Vq4QDxXtCKI/AAAAAAAAANw/t68YYtMkx2c/s1600/image033.png)  

### Creating a delivery group

1\. Open Citrix Studio  
 [![]({{ site.url }}/images/2016-01-31/2016-01-31-image005.png)]({{ site.url }}/images/2016-01-31/2016-01-31-image005.png)  
  
2\. Click **Delivery Groups** and click **Create Delivery Group**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image034.png)](http://1.bp.blogspot.com/-S8ssHpB82w8/Vq4QD1QcwkI/AAAAAAAAAOA/22gT-Agn3sg/s1600/image034.png)  
  
3\. Select the amount of machines you want to add from the **Win10 Azure MCS** Machine Catalog and click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image035.png)](http://3.bp.blogspot.com/-QBc0xFv0BxA/Vq4QEJJm2oI/AAAAAAAAAOA/k-H80H9ocEE/s1600/image035.png)  
  
4\. Select Delivery type, in this example **Desktops** and click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image036.png)](http://4.bp.blogspot.com/-vS29yQxT_6c/Vq4QECcTAlI/AAAAAAAAAOA/BIAkYdEuNi0/s1600/image036.png)  
  
5\. Select Users and/or Groups who need access to the Desktop and click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image037.png)](http://4.bp.blogspot.com/-TqjdopAtASg/Vq4QEdSWbNI/AAAAAAAAAN0/n5llMUJT_m8/s1600/image037.png)  
  
6\. Select **Manually** and click **Next**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image038.png)](http://1.bp.blogspot.com/-lj2Hlbq9jM8/Vq4QEcsdLAI/AAAAAAAAANU/oNbSugDRbOQ/s1600/image038.png)  
  
7\. Type in a **Delivery Group name** and **Display name** and click **Finish**[![]({{ site.url }}/images/2016-01-31/2016-01-31-image039.png)](http://2.bp.blogspot.com/-yarI6i8AAnk/Vq4QEQMXJNI/AAAAAAAAAN4/54UaxQbt4Ic/s1600/image039.png)  
  
8\. The delivery group is created[![]({{ site.url }}/images/2016-01-31/2016-01-31-image040.png)](http://2.bp.blogspot.com/-eB_Ah5VGDG4/Vq4QEsdex-I/AAAAAAAAAMw/gr7sq4GqTr4/s1600/image040.png)  

### XenDesktop power management
The **power management** function for VDI machines is also available for Delivery Groups with Azure MCS Machines. This feature will give you the advantage to start a number of machines in a specific period, thus allowing you to configure XenDesktop to start less machines outside business peak hours. Tuning this feature for your organization allows you to cut Azure costs, not all VDI machines need to be started 24/7.  
 [![]({{ site.url }}/images/2016-01-31/2016-01-31-image041.png)](http://4.bp.blogspot.com/-Sml0CHOF5ns/Vq4QE5bG6CI/AAAAAAAAAOA/SKFMKtLPoqM/s1600/image041.png)  

### Starting and rebooting random (non-persistent) VDI machines on Azure
Starting and restarting random (non-persistent) VDI machines on Azure is taking a lot of more time then rebooting an VDI machine on your on-premises hypervisor. Therefore a good tuned **power management** will give the best user experience for your Azure workspace.  
  
1\. When the Azure MCS Machine Catalog was created, I’ve noticed the following Virtual Machines on Azure[![]({{ site.url }}/images/2016-01-31/2016-01-31-image042.png)](http://4.bp.blogspot.com/-TT_DVm4MAD4/Vq4QFE6-QvI/AAAAAAAAANA/8cFFe_gcmgs/s1600/image042.png)  
  
2\. When machine **W10-MCS-01** is started by the Citrix Delivery Controller, Citrix MCS first removes “the old”  **W10-MCS-01** machine  

[![]({{ site.url }}/images/2016-01-31/2016-01-31-Untitled.png)](http://3.bp.blogspot.com/-35HL82fBXvM/Vq4YxQUTPgI/AAAAAAAAAOM/HtxKwpWbdaE/s1600/Untitled.png)

3\. Then provisions a new **W10-MCS-01** machine in Azure  
[![]({{ site.url }}/images/2016-01-31/2016-01-31-image043.png)](http://2.bp.blogspot.com/-zeK32h_TKh4/Vq4QFd5RQYI/AAAAAAAAAM0/NCcBP5GTQjI/s1600/image043.png)   
4\. After that, you are able to login to your Azure VDI machine:  

[![]({{ site.url }}/images/2016-01-31/2016-01-31-image044.png)](http://4.bp.blogspot.com/-iig-BFGESBw/Vq4QFTi3YxI/AAAAAAAAAMs/WT4L0RkY1M4/s1600/image044.png)