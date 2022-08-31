---
layout: post
title:  "How to: Citrix Workspace Cloud | Apps and Desktops"
date:   2016-03-01 17:54:00 +0100
categories:  Citrix, Citrix Workspace Cloud Connector, Citrix Workspace Cloud Apps and Desktops, XenDesktop, CWC, Director, Citrix Workspace Cloud, StoreFront, XenApp
---

This blog is a technical step-by-step guide for configuring and using Citrix Workspace Cloud - Apps and Desktops (CWC-AD). In this blog:  

*   What is Citrix Workspace Cloud - Apps and Desktop
*   Installation of Citrix Workspace Cloud Connector
*   Install a VDA and connect it to the Citrix Workspace Cloud Connector
*   Create a Machine Catalog on Citrix Workspace Cloud Apps and Desktops
*   Create a Delivery Group on Citrix Workspace Cloud Apps and Desktops
*   Configure a Citrix Workspace Cloud Workspace with Apps and Desktops
*   Citrix Workspace Cloud - StoreFront
*   Connect your on-premises StoreFront to Citrix Workspace Cloud
*   Citrix Workspace Cloud Apps and Desktops – Monitor (Director)

## What is Citrix Workspace Cloud - Apps and Desktop
Citrix has built an online cloud platform called “Citrix Workpace Cloud” (CWC). With CWC it is possible to move your on-premises Citrix core components towards the cloud (workspace.cloud.com). The great advantage of this is that you don’t need to install, configure and maintain these core components, Citrix is doing this for you! I.e. for Apps and Desktops you don’t need the following components on-premises:  

*   Citrix License Server
*   Citrix Desktop Delivery Controllers
*   Citrix Datastore
*   Microsoft SQL mirror or AllwaysOn database needs
*   Citrix Datastore
*   Citrix StoreFront

To connect your existing environment to CWC you only need to install the CWC Connector on a server. Citrix recommends to install at least two CWC Connectors for redundancy. The CWC Connectors are acting as a proxy between CWC and your environment. After installation of the CWC Connector you only need to point your Virtual Desktop Agents to the CWC Connector and you’re ready to go. Besides CWC-AD the Citrix Workspace cloud also has the following components: Workspaces, Identity and Access Managent, Resource Locations, Lifecycle management, Secure Documents and Mobility Management. This blog only describes the Apps and Desktop part.  

## Installation of Citrix Workspace Cloud Connector
The first thing to do is to connect your environment to CWC by installing the CWC Connector on a server.  
  
1\. Goto **workspace.cloud.com** and login with your CWC account:  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image001.png)](https://2.bp.blogspot.com/-Uj5QBY0XnVs/VtW-eM1n4sI/AAAAAAAAAUU/jQcH2kuEiUc/s1600/image001.png)

2\. Goto **Identity and Access Management**  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image002.png)](https://1.bp.blogspot.com/-HU8eKcEdkLc/VtW-KinSWPI/AAAAAAAAAUQ/JPQuubL0EUY/s1600/image002.png)

3\. Click **Domains** and click **plus sign** for adding a new domain  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image003.png)](https://1.bp.blogspot.com/-0LQdmhMjqWA/VtW-Kt-pI6I/AAAAAAAAAUQ/ALCwxrkNrxg/s1600/image003.png)

4\. Klik **Download** to download the CWC Connector:  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image004.png)](https://4.bp.blogspot.com/-LJ0LeiGiB1Q/VtW-LEuk6yI/AAAAAAAAAUQ/wqyS16MZoxU/s1600/image004.png)

5\. Save the **cwcconnector.exe** on the server and **execute it**  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image005.png)](https://2.bp.blogspot.com/-mpqS3RUgaco/VtW-LBVVW0I/AAAAAAAAAUQ/n9-MzGskjpc/s1600/image005.png)

6\. A **login popup** appears, login with your **CWC credentials**  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image006.png)](https://3.bp.blogspot.com/-IMTet-D3Rz8/VtW-Llde71I/AAAAAAAAAUQ/sp8pGxTEnIw/s1600/image006.png)

7\. Within two minutes, the installation box disappears and the following services are installed and started on the CWC Connector server:  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image007.png)](https://1.bp.blogspot.com/-m-bIc0bXUPU/VtW-LwwIoyI/AAAAAAAAAUQ/WYExRKLcbew/s1600/image007.png)

8\. Your domain will appear on **Identity and Access Management** at CWC  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image008.png)](https://2.bp.blogspot.com/-zE5U9NxyqNI/VtW-L0pxNTI/AAAAAAAAAUQ/Dd57IVO9QLo/s1600/image008.png)

## Install a VDA and connect it to the Citrix Workspace Cloud Connector
When the CWC Connector is in place and connected to CWC, you need to install a VDA and point it to the CWC Connector. In this example I will use a Microsoft Azure Windows 2012 R2 Session Host and install the VDA manually on it. You can also use On-premises MCS and PVS, Azure MCS or Amazon EC2.  
  
1\. **Mount** the **XenDesktop ISO** (or download the VDAServerSetup\_78.exe / VDAWorkstationSetup\_7.8.exe from download.citrix.com) and select **Virtual Delivery Agent for Windows Server OS**  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image009.png)](https://4.bp.blogspot.com/-PmoGCg11IbU/VtW-MItAHcI/AAAAAAAAAUQ/aNjuVtoZ0LM/s1600/image009.png)

2\. Select **Enable connections to a server machine** and click **Next**  
**[![]({{ site.url }}/images/2016-03-01/2016-03-01-image010.png)](https://1.bp.blogspot.com/-GCg6EOcbPcA/VtW-MQUrl9I/AAAAAAAAAUQ/Xov9Q3a2bVU/s1600/image010.png)**

3\. Click **Next**  
**[![]({{ site.url }}/images/2016-03-01/2016-03-01-image011.png)](https://3.bp.blogspot.com/-rNEjq123Qhg/VtW-Mp5NZCI/AAAAAAAAAUQ/n8CmmQYcv-o/s1600/image011.png)**

4\. At the **Controller address** field, fill in the **server names** of the **CWC Connector** (in this example Domain Controller **AZ-DC-01.azureworkspace.pqrlab.local**)  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image012.png)](https://1.bp.blogspot.com/-M7r3wXgulWo/VtW-MpIZbtI/AAAAAAAAAUQ/Obj6h2DLlts/s1600/image012.png)

5\. Select your components and click **Next**  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image013.png)](https://2.bp.blogspot.com/-4H0omGfyhhs/VtW-MwyeBSI/AAAAAAAAAUQ/YpppKhLdQC8/s1600/image013.png)

6\. Click **Next**  
**[![]({{ site.url }}/images/2016-03-01/2016-03-01-image014.png)](https://2.bp.blogspot.com/-yjBKd5qDORw/VtW-NOloA2I/AAAAAAAAAUQ/ll4u198Z1yE/s1600/image014.png)**

7\. Click **Install**  
**[![]({{ site.url }}/images/2016-03-01/2016-03-01-image015.png)](https://4.bp.blogspot.com/-T64ht2P8dFU/VtW-NcfQVgI/AAAAAAAAAUg/eQPnv5c6sJY/s1600/image015.png)**

8\. Once the installation has finished, click **Finish** to restart the Session Host  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image016.png)](https://4.bp.blogspot.com/-4kBXeqtyFFE/VtW-Ns35peI/AAAAAAAAAUg/BrDmqOh0Rtw/s1600/image016.png)

## Create a Machine Catalog on Citrix Workspace Cloud Apps and Desktops
1\. On CWC go to **Apps and Desktops**  
**[![]({{ site.url }}/images/2016-03-01/2016-03-01-image017.png)](https://1.bp.blogspot.com/-qzSO8x1T9Vs/VtW-N8B8KUI/AAAAAAAAAUg/-FTPwK6a99A/s1600/image017.png)**

  
2\. Click **Manage**, a HTML5 based PubApp with a Microsoft MMC of Citrix Studio will open for you.
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image018.png)](https://3.bp.blogspot.com/-Vro7l4KysaU/VtW-N5GwCRI/AAAAAAAAAUg/G0axRQrZjoQ/s1600/image018.png)

3\. Goto **Machine Catalogs** and click **Create Machine Catalog**  
**[![]({{ site.url }}/images/2016-03-01/2016-03-01-image019.png)](https://4.bp.blogspot.com/-7nVg6nJVvR0/VtW-OKvslQI/AAAAAAAAAUg/Yv3BngdzJDw/s1600/image019.png)**

4\. Click **Next
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image020.png)](https://1.bp.blogspot.com/-Nom03XH0VTk/VtW-OUbixTI/AAAAAAAAAUc/8GsLIzOavrU/s1600/image020.png)

**5\. Select **Server OS** and click **Next
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image021.png)](https://3.bp.blogspot.com/-AMQL2GL4PCU/VtW-OeO4tJI/AAAAAAAAAUc/R4T7h8BBss4/s1600/image021.png)

**6\. Select **Machines that are not powered on** (we’re using manually provisioned Citrix VDA on Azure) and click **Next
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image022.png)](https://1.bp.blogspot.com/-DQxmzRWdPfs/VtW-OwR30wI/AAAAAAAAAUc/PF9Pyz53xdM/s1600/image022.png)

**7\. Click **Add Computer…** and search for your ServerVDA machine (**AZ-XA-01.azureworkspace.pqrlab.local**)
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image023.png)](https://4.bp.blogspot.com/-_bfdeaYvWaQ/VtW-O7KHOfI/AAAAAAAAAUc/NzsFPBAblGM/s1600/image023.png)

8\. Click **Next
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image024.png)](https://1.bp.blogspot.com/-2Rri7dn092o/VtW-PHvUBRI/AAAAAAAAAUc/507p4c-K7Ec/s1600/image024.png)

**9\. Give your **Machine Catalog** a **name** and a **description**, click **Finish
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image025.png)](https://4.bp.blogspot.com/-VnG6rxGIkhE/VtW-PZMfZ0I/AAAAAAAAAUc/Tx0_OH-dVuI/s1600/image025.png)

**10\. Your Machine Catalog on CWC is created
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image026.png)](https://2.bp.blogspot.com/-9F21OBbAm00/VtW-PksB3II/AAAAAAAAAUc/HsgM8yadLow/s1600/image026.png)

## Create a Delivery Group on Citrix Workspace Cloud Apps and Desktops

1\. On CWC go to **Apps and Desktops**  
**[![]({{ site.url }}/images/2016-03-01/2016-03-01-image017.png)](https://1.bp.blogspot.com/-qzSO8x1T9Vs/VtW-N8B8KUI/AAAAAAAAAUg/-FTPwK6a99A/s1600/image017.png)**

2\. Click **Manage**, a HTML5 based PubApp with a Microsoft MMC of Citrix Studio will open for you.  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image018.png)](https://3.bp.blogspot.com/-Vro7l4KysaU/VtW-N5GwCRI/AAAAAAAAAUg/G0axRQrZjoQ/s1600/image018.png)

3\. Go to **Delivery Groups** and click **Create Delivery Group
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image027.png)](https://2.bp.blogspot.com/-RBAVn2XuXJw/VtW-PsmjPaI/AAAAAAAAAUc/La80NIh-WRM/s1600/image027.png)

**4\. Click **Next
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image028.png)](https://2.bp.blogspot.com/--VeJcCS9LG4/VtW-P2d331I/AAAAAAAAAUc/To7IUgyLCC4/s1600/image028.png)

**5\. Add machines from **Machine Catalog** and click **Next
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image029.png)](https://2.bp.blogspot.com/-Ddv2YI7QJRM/VtW-QB2mCyI/AAAAAAAAAUc/Gdoi55sBDYc/s1600/image029.png)

**6\. Select **Leave user management to Citrix Workspace Cloud** and click **Next
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image030.png)](https://4.bp.blogspot.com/-8hpLkVSZ0LE/VtW-QeeHeGI/AAAAAAAAAUc/eSf5pdomuk8/s1600/image030.png)

**7\. Click **Add** to add applications
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image031.png)](https://4.bp.blogspot.com/-EmJJVQAiSpg/VtW-QeR0oAI/AAAAAAAAAUc/pIlccsCj-Jo/s1600/image031.png)

8\. **Select** the ServerVDA **installed applications** you want to publish in CWC Workspaces
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image032.png)](https://2.bp.blogspot.com/-1Bxum_uJ96Y/VtW-QszHULI/AAAAAAAAAUc/aknr_Y0fvVg/s1600/image032.png)

9\. Click **Next
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image033.png)](https://3.bp.blogspot.com/-dBpSY5eKL48/VtW-Q-VlPdI/AAAAAAAAAUc/1a0MkrJ1qUY/s1600/image033.png)

**10\. Give the **Delivery group** a **name** and **description**, click **Finish
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image034.png)](https://1.bp.blogspot.com/-QC9MBnCZILM/VtW-RDHNSiI/AAAAAAAAAUc/hKGOE_EYzeY/s1600/image034.png)

**11\. The delivery group is created
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image035.png)](https://1.bp.blogspot.com/-SqZlkTikvaE/VtW-RCOrURI/AAAAAAAAAUc/ml6lGLTDJa8/s1600/image035.png)

## Configure a Citrix Workspace Cloud Workspace with Apps and Desktops

1\. On CWC go to **Workspaces**  
**[![]({{ site.url }}/images/2016-03-01/2016-03-01-image036.png)](https://4.bp.blogspot.com/-2gvEOlEuu9I/VtW-RpM9uUI/AAAAAAAAAUc/UvOP0XjfDDs/s1600/image036.png)**

2\. Click **plus Workspace**s to create a new CWC workspace
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image037.png)](https://1.bp.blogspot.com/-cF1yCMdVE34/VtW-Rrypl0I/AAAAAAAAAUc/HNMv2VaHrFU/s1600/image037.png)

3\. Give the Workspace a name and click **Applications and Desktops
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image038.png)](https://1.bp.blogspot.com/-1RUbcLwyzHs/VtW-R0lDg2I/AAAAAAAAAUc/EhfOqjpaHrA/s1600/image038.png)

**4\. At this page the Delivery groups with CWC user management are displayed. **Add** Apps and Desktops from delivery group **Man – Azure – XenApp
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image039.png)](https://4.bp.blogspot.com/-ZHyA7GPQmWs/VtW-R1yOvYI/AAAAAAAAAUc/yGGXHh7R8Eo/s1600/image039.png)

**5\. Than click **Create Workspace
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image040.png)](https://3.bp.blogspot.com/-jIJSqnVtX6Q/VtW-SD5ChcI/AAAAAAAAAUU/q1dwk9uUSM0/s1600/image040.png)

**6\. **Click** the Workspace **Azure XenApp** to manage the new workspace
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image041.png)](https://1.bp.blogspot.com/-sUGN0LPjUlU/VtW-SDZA2DI/AAAAAAAAAUU/g8cKL_4SIzY/s1600/image041.png)

7\. If you want to manage services, you can do it over here.
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image042.png)](https://2.bp.blogspot.com/-8tzlw3yv6K8/VtW-SVEVGxI/AAAAAAAAAUU/-5RRcW3vLsY/s1600/image042.png)

8\. Click **Subscribers** and add the domain users you want to give access to the workspace. Click **Publish** to publish the Workspace.
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image043.png)](https://3.bp.blogspot.com/-co-b2QQ3spk/VtW-Sgy_sPI/AAAAAAAAAUU/0WLJjUHY96g/s1600/image043.png)

## Citrix Workspace Cloud - StoreFront
By default you will get an CWC managed StoreFront address. You can access this by **https://%customername%.xendesktop.net**. Keep in mind that you can only use this page for internal access of the Apps and Desktops. You need a NetScaler Gateway for remote access over the internet.  
   
1\. Goto **https://%customername%.xendesktop.net** and login with you credentials  
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image044.png)](https://4.bp.blogspot.com/-YQKDE2GhxYA/VtW-S0c-4DI/AAAAAAAAAUU/rgvY3Tml3no/s1600/image044.png)

2\. You can see the CWC Workspace Published Apps here, click **Notepad
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image045.png)](https://4.bp.blogspot.com/-fnJ3GYTmsQY/VtW-TJ4uPaI/AAAAAAAAAUU/9THk_2hpPfo/s1600/image045.png)

**3\. The PubApp **Notepad** is started on the Azure ServerVDA
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image047.png)](https://4.bp.blogspot.com/-9_SGRIE1qAQ/VtW-TUmQECI/AAAAAAAAAUU/ch1AFNw0inw/s1600/image047.png)

4\. Goto **Desktops** to see you CWC Workspace Published Desktops, click **Azure XenApp** to start the PubDesktop
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image048.png)](https://2.bp.blogspot.com/-_zaKHnHkkTQ/VtW-TfdOgjI/AAAAAAAAAUU/m7rdneO9bc0/s1600/image048.png)

## Connect your on-premises StoreFront to Citrix Workspace Cloud
It is also possible to connect an on-premises storefront to CWC.   
  
1\. Open **StoreFront Console** on your StoreFront server, go to **Stores** and click **Create Store**  
**[![]({{ site.url }}/images/2016-03-01/2016-03-01-image049.png)](https://2.bp.blogspot.com/-5HRpjlysBBE/VtW-Tg21vuI/AAAAAAAAAUU/IWqUW0U2604/s1600/image049.png)**

2\. Give you StoreFront store a name and click **Next
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image050.png)](https://4.bp.blogspot.com/-CQoPDQJ4MdQ/VtW-TxzDjqI/AAAAAAAAAUU/VAt32RXo4RE/s1600/image050.png)

**3\. Point your Delivery controllers to the CWC Cloud connector
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image051.png)](https://3.bp.blogspot.com/-C5ucMqgLqag/VtW-UDLlLhI/AAAAAAAAAUU/2mOWU4nrTv0/s1600/image051.png)

4\. Click **create
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image052.png)](https://4.bp.blogspot.com/-Wzq8DmqvxOM/VtW-UCIyR7I/AAAAAAAAAUU/Y27eSbwUl5I/s1600/image052.png)

**5\. Click **Finish
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image053.png)](https://4.bp.blogspot.com/-yIUDv-BmBzw/VtW-Ug9jgnI/AAAAAAAAAUU/NBvaw4Jc-kc/s1600/image053.png)

**6\. Your on-premises StoreFront is connected to CWC
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image054.png)](https://2.bp.blogspot.com/-PDXjSCbb8K8/VtW-UnEDgmI/AAAAAAAAAUU/f-how9VUUQY/s1600/image054.png)

## Citrix Workspace Cloud Apps and Desktops – Monitor (Director)
The Citrix Director is also implemented on CWC. Director is working the same as an on-premises installation, except you cannot configure any role based access. I.e. you cannot limit helpdesk employees in the console.  
  
1\. On CWC go to **Apps and Desktops**  
**[![]({{ site.url }}/images/2016-03-01/2016-03-01-image017.png)](https://1.bp.blogspot.com/-qzSO8x1T9Vs/VtW-N8B8KUI/AAAAAAAAAUg/-FTPwK6a99A/s1600/image017.png)**

2\. Then click **Monitor**, you will see the familiar Citrix Director dashboard
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image055.png)](https://3.bp.blogspot.com/-G1PHzaqb7UM/VtW-UxDXL0I/AAAAAAAAAUU/bczJfAgzlaY/s1600/image055.png)

3\. Click **Trends** to view the historical trends of your Citrix environment
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image056.png)](https://4.bp.blogspot.com/-TWZ68eGZERQ/VtW-VH8hYoI/AAAAAAAAAUU/86rey8tMJ8M/s1600/image056.png)

4\. Click **Search** to search and manage a user
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image057.png)](https://1.bp.blogspot.com/-cCtAjXuWhg4/VtW-VWpIERI/AAAAAAAAAUU/Uebf0T9dtzs/s1600/image057.png)

5\. You can manage Applications and Processes opened by the user on the ServerVDA
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image058.png)](https://2.bp.blogspot.com/-7xQ-buJM-24/VtW-Vfm4onI/AAAAAAAAAUU/lKAiZh6Csgg/s1600/image058.png)
6\. Or see more details about the user session (like server load evaluator index, ICA RTT, user connection latency):
[![]({{ site.url }}/images/2016-03-01/2016-03-01-image059.png)](https://2.bp.blogspot.com/-JeBQShNV8Do/VtW-VhOHuxI/AAAAAAAAAUU/zg5IbJlqa-8/s1600/image059.png)