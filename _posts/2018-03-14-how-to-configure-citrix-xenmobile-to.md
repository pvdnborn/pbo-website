---
layout: post
title:  "How to: Configure Citrix XenMobile to use Microsoft SQL multi-subnet (Basic) Availability Groups"
date:   2018-03-14 18:40:00 +0100
categories:  BAGs, multi-subnet, XenMobile, Failover cluster, Citrix, Clustering, AlwaysOn, SQL Server 2016 Standard, How to, SQL Server, Failover, Basic Availability Groups
---

In the previous blogs I’ve explained you 
* [How to: Install and configure Microsoft SQL Server 2016 Standard multi-subnet Basic Availability Groups for Citrix XenDesktop and XenMobile]({{ site.url }}/2017-11-06-how-to-install-and-configure-microsoft/)
* [Microsoft SQL and Microsoft SQL AlwaysOn basics for Citrix Admins]({{ site.url }}/2018-01-08-microsoft-sql-and-microsoft-sql/)
  
This blog is a guide for configuring Citrix XenMobile with a multi-subnet SQL AlwaysOn database. When using a multi-subnet SQL AlwaysOn cluster, we need to reconfigure the Windows Failover Cluster Network parameters **RegisterAllProvidersIP=0** and **HostRecordTTL=60** for the XenMobile AlwaysOn listener. Special thanks for the details delivered by [@JanPaulPlaisier](https://twitter.com/JanPaulPlaisier)! Details will be explained later in this blog.  
  
During a customer’s project we’ve build the following XenMobile backend:  

*   One XenMobile cluster
    *   One XenMobile server in Amersfoort: PBO-XMS01.pbo.lan
    *   One XenMobile server in Nijkerk: PBO-XMS02.pbo.lan

*   Both sites have a NetScaler with a GSLB configuration for XenMobile. The NetScaler configuration is not covered in this blog.

*   The XenMobile service in Amersfoort is the primary site. When the XenMobile service in Amersfoort becomes unresponsive, GSLB will failover the XenMobile services to Nijkerk.  
    

Logical overview  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image002.png)](https://3.bp.blogspot.com/-WqzgVPbCkpg/WqlKigAxilI/AAAAAAAACkU/-Ipqxir5q7kK0GDBB9WKKfAPbHx0bJ1nwCEwYBhgL/s1600/image002.png)
  
**Note:** Site-to-site network latency is 4ms.  

Create SQL Login for XenMobile service account
----------------------------------------------

Create an Active Directory Service Account for the XenMobile database connection as follows:  
  
1.    Create a **Domain User** in Active Directory, this is your service account (i.e. **XMSvc**)  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image003.png)](https://3.bp.blogspot.com/-TUszldeQtPs/WqlKitI-vYI/AAAAAAAACkU/SFR5iMLAxIolU0sCGmc_CLbxF3tOg-TqwCEwYBhgL/s1600/image003.png)
  
2.    Open **SQL Management Studio**  

3.    Create a SQL Server login for the XenMobile service account on all the SQL nodes in your AlwaysOn configuration. Starting with node 1:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image004.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image004.png)
  
4.    Select **Windows Authentication** and select your Service Account (i.e. **PBO\\XMSvc**). Then click **Server Roles**:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image005.png)](https://1.bp.blogspot.com/-6HVkF4qAAHs/WqlKjXNbLgI/AAAAAAAACkQ/n79Saq91aTsNT-jPsu8TUVzrM8we2rbswCEwYBhgL/s1600/image005.png)
  
5.    Select **dbcreator** and click **OK**:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image006.png)](https://1.bp.blogspot.com/-6_5laFh_MWc/WqlKj7nEmXI/AAAAAAAACkU/teizty6r6fMvZl-fo5co9AlzHSSY5fdYQCEwYBhgL/s1600/image006.png)
  
6.    Create this login on the remaining SQL servers:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image007.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image007.png)

Creating the initial database
-----------------------------
Before we can join the Citrix XenMobile database to a (Basic) Availability Group, we need to create the database on one of the SQL servers directly using the XenMobile appliance. This how-to is describing an initial setup using the XenMobile appliance. If you need to move an existing XenMobile database to an availability group, please skip this chapter and start with the next chapter.  
  
1.    Import the XenMobile appliance you’ve downloaded from [download.citrix.com](https://download.citrix.com/)  

2.    The “**First time Use mode**” is started, I will not discuss default configuration. I will only discuss database configuration  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image008.png)](https://2.bp.blogspot.com/-spszcDCjYJE/WqlKkNO9wZI/AAAAAAAACkM/5H_JpQd8eRkkuLbetK-dCAj_Dl5ahLhTQCEwYBhgL/s1600/image008.png)
  
3.    Configure the database connection to connect to one of your SQL nodes directly (i.e. **PBO-SQL01.pbo.lan**). Connect using the XenMobile service account (i.e. **PBO\\XMSvc**)  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image009.png)](https://4.bp.blogspot.com/-Ab7oTaQ70vY/WqlKkXSKgmI/AAAAAAAACkQ/x3kQyzA7owgQJrjbq7BKCEzrAmnE4TyCQCEwYBhgL/s1600/image009.png)
  
4.    The **database is created** by the “**First Time Use mode**”  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image010.png)](https://4.bp.blogspot.com/-KzTCkCfh_uc/WqlKkeSNagI/AAAAAAAACkU/Oe3nC-eNMboHqq148hJ0x-UGSJ3YVqm9wCEwYBhgL/s1600/image010.png)
  
5.    **Finish** the “First Time Use mode” configuration with the specific options for your XenMobile design.  

Joining the Citrix XenMobile database to a (Basic) AlwaysOn Availability Group
------------------------------------------------------------------------------
At this point, we want to join the “**CTX-XM**” database to a new Microsoft SQL multi-subnet Basic AlwaysOn Availability Group (BAG). When you have a Citrix XenMobile environment running without BAG, you can also use these steps to move to a BAG setup.  
  
1.    Open **SQL Management Studio**  

2.    Right click the **CTX-XM** database and click **Properties**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image011.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image011.png)
  
3.    Click **Options** and ensure **Recovery model** is **Full**. Click **OK**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image012.png)](https://4.bp.blogspot.com/-xUkWfDUrghs/WqlKlBF24NI/AAAAAAAACkY/SZbP47JkVogI_G22xFY1iKytYxTONOzBACEwYBhgL/s1600/image012.png)
  
4.    Right click **CTX-XM** and click **Tasks --> Back Up..**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image013.png)](https://1.bp.blogspot.com/-RO-0rwcJSK4/WqlKlWI-3NI/AAAAAAAACkM/7s3Qw_sc7XQN6OUzwXEtYaIUjFsVQcSHwCEwYBhgL/s1600/image013.png)
  
5.    Create a **Full backup** of the **CTX-XM** database:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image014.png)](https://1.bp.blogspot.com/-Aqd16GBTUVw/WqlKmESqjXI/AAAAAAAACkY/HNH94zbrFX8sqrzznJFv_02Ev_g6nq27gCEwYBhgL/s1600/image014.png)
  
6.    Right click **Availability Groups** and click **New Availability Group Wizard…**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image015.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image015.png)
  
7.    Click **Next**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image016.png)](https://3.bp.blogspot.com/-Hz5J5RI4sRI/WqlKl9JkuQI/AAAAAAAACkQ/V6F31uyomoc61zDjVIaC9LRffuSkSulXQCEwYBhgL/s1600/image016.png)
  
8.    Enter a name for the Availability Group (i.e. **BAG-CTX-XM**) and click **Next**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image017.png)](https://3.bp.blogspot.com/-crNtnye9fkU/WqlKmV1KxdI/AAAAAAAACkQ/mKRBOt5XZCcUbgojqPnYlBRvdSRBRUVLgCEwYBhgL/s1600/image017.png)
  
9.    Select the **CTX-XM** database and click **Next**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image018.png)](https://1.bp.blogspot.com/-tlMs0WWf8zs/WqlKmc81mtI/AAAAAAAACkQ/zdeGuSUeifoBkwQkn-_wMsdW1AS4g6b6wCEwYBhgL/s1600/image018.png)
  
10.    Click **Add Replica…**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image019.png)](https://2.bp.blogspot.com/-6AepKu-O3QA/WqlKmgDaUQI/AAAAAAAACkU/3fQzbCk1eZQAejOsiMjPpPoDmfgdEmbGgCEwYBhgL/s1600/image019.png)
  
11.    Login to the **second SQL node**:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image020.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image020.png)
  
12.    Select **Automatic Failover** and ensure Availability Mode is **Synchronous commit**. Click **Listener** tab:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image021.png)](https://2.bp.blogspot.com/-tX-O3GvQS5c/WqlKnIEcUfI/AAAAAAAACkM/K04gwzaxE_EaayTUpqmW6G4Q14wtp-WJQCEwYBhgL/s1600/image021.png)
  
13.    Configure **DNS-name** and **static IPs** for the SQL AlwaysOn listener, click **Next**:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image022.png)](https://4.bp.blogspot.com/-HIjX5D3uF3Q/WqlKn11hjFI/AAAAAAAACkU/GRXkWA6WXJE1hDOtqWImdCkTSWXPJCoCACEwYBhgL/s1600/image022.png)
  
14.    Use a **SMB share** for initial seeding, click **next**:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image023.png)](https://3.bp.blogspot.com/-ssLwQGwKvao/WqlKoYDnYxI/AAAAAAAACkY/zTxoHG7jAmsgmBNOroRZ3xqOKZCvTdWmwCEwYBhgL/s1600/image023.png)
  
15.    Availability Group validation. Click **Next**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image024.png)](https://2.bp.blogspot.com/-uxMKXm84oxc/WqlKot6mt4I/AAAAAAAACkg/FKl5AXX7fKAXySfe5vgYBPPHJWKBu1iNACEwYBhgL/s1600/image024.png)

16.    Verify Summary and click **Finish**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image025.png)](https://2.bp.blogspot.com/-U3nYNgM05Co/WqlKowdG2HI/AAAAAAAACkg/W-MKHZEqpWc7eJBVnUYpBzPt8qjME-8mQCEwYBhgL/s1600/image025.png)
  
17.    Creating the AlwaysOn Availability group  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image026.png)](https://4.bp.blogspot.com/-muFnbLYStLs/WqlKpYxxUHI/AAAAAAAACkQ/7qX-G8e_BcQJsmbP7LRbbxnHrm_IEIB0QCEwYBhgL/s1600/image026.png)
  
18.    When everything is **OK**. Click **Close**:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image027.png)](https://2.bp.blogspot.com/-H4QuAVJf_H4/WqlKppVG0aI/AAAAAAAACkM/OakXODTJplQTKF66FqyNN-KG43I-h-L1wCEwYBhgL/s1600/image027.png)
  
19.    The availability group is created:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image028.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image028.png)

Reconfigure Windows Failover Cluster network parameters for BAG-CTX-XM Availability group
-----------------------------------------------------------------------------------------
As I’ve mentioned in the Citrix XenApp/XenDesktop multi-subnet AlwaysOn blog, Windows Failover Cluster is creating a round-robin DNS A-record for the multi-subnet Availability group. This is also the default after creating a multi-subnet Availability group for XenMobile:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image029.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image029.png)  

[![]({{ site.url }}/images/2018-03-14/2018-03-14-image030.png)](https://1.bp.blogspot.com/-oFqbKHqxDAk/WqlKqT_0kDI/AAAAAAAACkU/xUaqvlcr8S87ODpHDIAYDv7fQMRpi8G_wCEwYBhgL/s1600/image030.png)
  
Since XenMobile is supporting (Basic) Availability groups [https://docs.citrix.com/en-us/xenmobile/server/system-requirements.html](https://docs.citrix.com/en-us/xenmobile/server/system-requirements.html)  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image031.png)](https://3.bp.blogspot.com/-bWsneXGgEvI/WqlKq_MKMdI/AAAAAAAACkQ/w06KrB00RQAq-xVDA1P0qeXcs15CYPqgwCEwYBhgL/s1600/image031.png)  
The SQL driver in the XenMobile appliance isn’t aware or configurable with the **MultiSubnetFailover=true** for the SQL connection string. Due to this, the XenMobile appliance will resolve the IP-addresses of the BAG with a round robin mechanism. Result: connection issues to the XenMobile database when the off-line IP off the SQL listener is resolved. Other Citrix products like Provisioning Services, XenApp or XenDesktop are configurable with the **MultiSubnetFailover=true** connection string.  
   
To fix this issue for XenMobile, we can reconfigure the network parameters for the AlwaysOn XenMobile Availability group listener to only register the on-line IP to the DNS A-record. To force a recheck of the operating system, we reconfigure the network parameters to register a short TTL to the DNS record. This will force the XenMobile appliance to check if the IP is changed on the DNS every minute.  
  
Reconfigure the network parameters for the SQL AlwaysOn XenMobile listener as follows:  
  
1.    Login to one SQL nodes of the Windows Failover Cluster (i.e. **PBO-SQL01**)  

2.    Start **Powershell elevated**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image032.png)](https://2.bp.blogspot.com/-kJCqQGDm1XU/WqlKrMCsmkI/AAAAAAAACkQ/q6BqlSPp0zktvLS3Ph5HSVap0yd-Kx-NACEwYBhgL/s1600/image032.png)
 
3.    Type **Import-Module FailoverClusters** to import the powershell modules:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image033.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image033.png)
  
4.    Type **Get-ClusterResource** and find the Cluster Resource Network Name for your XenMobile BAG. In this example **BAG-CTX-XM\_BAG-CTX-XM**  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image034.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image034.png)
  
5.    Reconfigure Cluster parameters as follows (replace **BAG-CTX-XM\_BAG-CTX-XM** with the name of your WFC configuration):  
<table border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none; margin-left: 36.0pt; mso-border-alt: solid windowtext .5pt; mso-padding-alt: 0cm 5.4pt 0cm 5.4pt; mso-yfti-tbllook: 1184;"><tbody><tr style="mso-yfti-firstrow: yes; mso-yfti-irow: 0; mso-yfti-lastrow: yes;"><td style="border: solid windowtext 1.0pt; mso-border-alt: solid windowtext .5pt; padding: 0cm 5.4pt 0cm 5.4pt; width: 450.8pt;" valign="top" width="601"><div class="MsoListParagraphCxSpFirst" style="line-height: normal; margin-bottom: .0001pt; margin: 0cm; mso-add-space: auto;"><i><span lang="EN-US" style="mso-ansi-language: EN-US;">Import-Module FailoverClusters<span style="mso-spacerun: yes;">&nbsp;</span></span></i></div><i></i><br><div class="MsoListParagraphCxSpMiddle" style="line-height: normal; margin-bottom: .0001pt; margin: 0cm; mso-add-space: auto;"><i><span lang="EN-US" style="mso-ansi-language: EN-US;">Get-ClusterResource <b style="mso-bidi-font-weight: normal;">BAG-CTX-XM_BAG-CTX-XM</b> | Set-ClusterParameter RegisterAllProvidersIP 0<span style="mso-spacerun: yes;">&nbsp;&nbsp;</span></span></i></div><i></i><br><div class="MsoListParagraphCxSpMiddle" style="line-height: normal; margin-bottom: .0001pt; margin: 0cm; mso-add-space: auto;"><i><span lang="EN-US" style="mso-ansi-language: EN-US;">Get-ClusterResource <b style="mso-bidi-font-weight: normal;">BAG-CTX-XM_BAG-CTX-XM</b> |Set-ClusterParameter HostRecordTTL 60<span style="mso-spacerun: yes;">&nbsp;</span></span></i></div><i></i><br><div class="MsoListParagraphCxSpMiddle" style="line-height: normal; margin-bottom: .0001pt; margin: 0cm; mso-add-space: auto;"><i><span lang="EN-US" style="mso-ansi-language: EN-US;">Stop-ClusterResource <b style="mso-bidi-font-weight: normal;">BAG-CTX-XM_BAG-CTX-XM</b></span></i></div><i></i><br><div class="MsoListParagraphCxSpLast" style="line-height: normal; margin-bottom: .0001pt; margin: 0cm; mso-add-space: auto;"><i><span lang="EN-US" style="mso-ansi-language: EN-US;">Start-ClusterResource<b style="mso-bidi-font-weight: normal;"> BAG-CTX-XM_BAG-CTX-XM</b></span></i></div></td></tr></tbody></table>
  
Then wait or force DNS replication to all your DNS-servers. After that flush DNS information on all the WFC nodes of the SQL AlwaysOn cluster and the XenMobile appliances.  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image035.png)](https://3.bp.blogspot.com/-7jdnYGmDyYY/WqlKrs2KgGI/AAAAAAAACkY/MPhUKyYvl2U0XLxWUrQJeMf96SqIKBvswCEwYBhgL/s1600/image035.png)
  
6.    You will now have only one A-record with the active IP for the BAG:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image037.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image037.png)

[![]({{ site.url }}/images/2018-03-14/2018-03-14-image036.png)](https://1.bp.blogspot.com/-Apm3AnjMxJk/WqlKsMC8SNI/AAAAAAAACkU/4kY1HJOAqhkzI8cpIKwAXf5zmixXInT4QCEwYBhgL/s1600/image036.png)
 
7.    And after a failover to the other SQL server in the other subnet:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image038.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image038.png)
  
8.    The DNS record is adjusted:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image040.png)]({{ site.url }}/images/2018-03-14/2018-03-14-image040.png)

[![]({{ site.url }}/images/2018-03-14/2018-03-14-image039.png)](https://1.bp.blogspot.com/-VbdVWCEUMWI/WqlKs44A4wI/AAAAAAAACkM/XTFynenEBaE2kLRfn7YPZd7xwmHyjJTzgCEwYBhgL/s1600/image039.png)
  
As described in this blog I have a DNS-server PBO-DC01 in Amersfoort and a PBO-DC02 in Nijkerk. It is important that the XenMobile appliance (PBO-XM01) and SQL Server (PBO-SQL01) are using the DNS-server (PBO-DC01) as primary DNS-server in Amersfoort. For the Nijkerk site it is important that the XenMobile Appliance (PBO-XM02) and SQL Server (PBO-SQL02) are using DNS-server (PBO-DC02) as the primary DNS-server. The Windows Failover Cluster will contact the primary DNS-server to reconfigure the DNS-records in case of a failover. So if SQL and XenMobile appliances are configured the same way as WFC/SQL, XenMobile will notice the A-record change within 1 minute since it is contacting the same primary DNS-server.  

Reconfigure Citrix XenMobile to use the multi-subnet Basic Availability Group
-----------------------------------------------------------------------------
You can reconfigure XenMobile as follows:  
  
1.    Login to the **console** of your XenMobile appliance and select **0 (Configuration)** from the menu:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image041.png)](https://2.bp.blogspot.com/-KDnpMIi0y8U/WqlKtCXlUXI/AAAAAAAACkQ/sfXCbod_8kESzl7H-I9vxKbbKgKt9efmwCEwYBhgL/s1600/image041.png)
  
2.    Select option **3 (Database)** from the menu:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image042.png)](https://3.bp.blogspot.com/-DMngt9k5TAQ/WqlKtVxt4CI/AAAAAAAACkQ/jKqD4rTZl6kRUe86cQgyr_yqBFzslEsQQCEwYBhgL/s1600/image042.png)
  
3.    Reconfigure the server property to connect to the **BAG FQDN**:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image043.png)](https://4.bp.blogspot.com/-cA9bFiJmtXM/WqlKtswsK5I/AAAAAAAACkg/1j1S1pCEgkM5gy7DeaYXQGHeiKkSjqmjACEwYBhgL/s1600/image043.png)
  
4.    Reboot the appliance:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image044.png)](https://4.bp.blogspot.com/-TUe3CYkcO_0/WqlKthke9HI/AAAAAAAACkM/ghLrpRxSSwUwwGxuF3eB7PJOellYRXcGwCEwYBhgL/s1600/image044.png)
  
5.    Rebooting  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image045.png)](https://1.bp.blogspot.com/-xwl2bZtkjvY/WqlKtjuEWqI/AAAAAAAACkM/4E3U7YXHIg0CDngvi2rv5cg7ASKgRCsFgCEwYBhgL/s1600/image045.png)

6.    XenMobile is now reconfigured using the BAG:  
[![]({{ site.url }}/images/2018-03-14/2018-03-14-image046.png)](https://3.bp.blogspot.com/-M-07YET7SuM/WqlKt1ELSJI/AAAAAAAACkg/NO9rlqdN5wQ4Qexje6U5otu8uTD2hmX-gCEwYBhgL/s1600/image046.png)