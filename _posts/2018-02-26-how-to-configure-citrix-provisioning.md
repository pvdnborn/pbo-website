---
layout: post
title:  "How to: Configure Citrix Provisioning Services to use Microsoft SQL multi-subnet (Basic) Availability Groups"
date:   2018-02-26 17:30:00 +0100
categories:  Provisioning Services, Citrix, Clustering, BAGs, multi-subnet, SQL Server 2016 Standard, How to, Basic Availability Groups
---

In the previous blogs I’ve explained you “[_How to: Install and configure Microsoft SQL Server 2016 Standard multi-subnet Basic Availability Groups for Citrix XenDesktop and XenMobile_](https://patrickvandenborn.blogspot.nl/2017/11/how-to-install-and-configure-microsoft.html)” and “[_Microsoft SQL and Microsoft SQL AlwaysOn basics for Citrix Admins_](https://patrickvandenborn.blogspot.nl/2018/01/microsoft-sql-and-microsoft-sql.html)”.   
  
In the next series of blogs I will explain to you how to configure “Citrix XenApp/XenDesktop” and “Citrix XenMobile” using SQL MultiSubnetFailover.  
This blog starts with “How to configure Citrix Provisioning Services using MS SQL MultiSubnetFailover (Basic) Availability groups”. This is the most easiest one of the three I will describe in my blog.  
   
During the customers project I’ve created the following Citrix Provisioning Services configuration:  

*   One PVS Farm called: “PBO”
*   This PVS Farm contains two PVS sites

*   Amersfoort with PVS Servers:
    *   PBO-PVS01
    *   PBO-PVS02

*   Nijkerk with PVS Servers
    *   PBO-PVS03
    *   PBO-PVS04

*   Both sites having their own PVS-Targets. Since this is a multi-subnet environment, the PVS targets communicate to PVS Servers in their own site.

Logical overview:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image050.png)](https://2.bp.blogspot.com/-YDt37Scfkbs/WpQogJQ8G3I/AAAAAAAACck/2qjBGty90KkPJA6rpBnfHReN6iO1zT-egCEwYBhgL/s1600/image050.png)

**Note:** Site-to-site network latency is 4ms.  

Create initial database
-----------------------
Before we can join the Citrix Provisioning Services database to a (Basic) Availability Group, we need to create a database to one of the SQL Servers. This how-to is describing it using the “**Provisioning Services Configuration Wizard**”. So this blog can also be used to move from a single SQL database to a (Basic) AlwaysOn Availability Group if you’re already in production.  
  
1.    Mount the Citrix Provisioning Services ISO and install “**Console**” and then “**Server**”:[![]({{ site.url }}/images/2018-02-26/2018-02-26-image003.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image003.png)
  
2.    Assuming you’ve already installed PVS before, I will skip the Installation instructions:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image004.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image004.png)
  
3.    Our SQL environment currently doesn’t have a Citrix Provisioning Services Database:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image005.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image005.png)
  
4.    When the “PVS Server Installation” finishes, the “Citrix Provisioning Services Configuration Wizard” is started automatically. Click **next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image006.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image006.png)
  
5.    Select **DHCP properties** for your environment. Click **next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image007.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image007.png)
  
6.    If you want to use **PXE-services**, configure it for your PVS Environment. Click **next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image008.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image008.png)
  
7.    Select “**Create Farm**” as we want to create a new Farm here. Click **next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image009.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image009.png)
  
8.    Enter the direct server name of your primary/first SQL server. Do not use a listener address here, we will configure AlwaysOn and MultiSubnetFailover later in this procedure. Click **next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image010.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image010.png)
  
9.    Enter **new names** for the SQL Database and PVS environment, **click next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image011.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image011.png)
  
10.    Enter details for the new **vDisk store**, click **Next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image012.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image012.png)
  
11.    Enter the configuration to your **Citrix License** server, click **Next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image013.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image013.png)
  
12.    Enter the credentials for your **Citrix Provisioning Services service Account**, click **Next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image014.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image014.png)
  
13.    Leave default, click **next**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image015.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image015.png)
  
14.    Configure the PVS network configuration according to your design. Click **next**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image016.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image016.png)
  
15.    **Enable TFTP** services. Click **next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image017.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image017.png)
  
16.    Configure bootstrap according to your design. Click **next**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image018.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image018.png)
  
17.    Configure SOAP SSL if you have a valid certificate. Otherwise click **next**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image019.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image019.png)
  
18.    If you want to use problem reporting. Please provide your MyCitrix credentials (optional):  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image020.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image020.png)
  
19.    Press Finish to configure PVS and create a new database, click **next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image021.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image021.png)
  
20.    Creating Services and database right now:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image022.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image022.png)
  
21.    New SQL database created on “PBO-SQL01” and not on “PBO-SQL02”:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image023.png) ]({{ site.url }}/images/2018-02-26/2018-02-26-image023.png)

Joining the Citrix Provisioning Services database to a (Basic) AlwaysOn Group
-----------------------------------------------------------------------------
At this point, we want to join the “**CTX-PVS**” database to a new Microsoft SQL MultiSubnet Basic AlwaysOn Availability Group (BAG). When you have a Citrix Provisioning Services environment running without using BAG, you can also follow these steps to move to BAG.  
  
1.    **Stop** all the “Citrix Provisioning Services” services on all of your Citrix Provisioning Servers. This will stop all communication to the Provisioning Services database:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image024.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image024.png)
  
2.    In the SQL Management Studio, right click the new Citrix Provisioning Services database (**CTX-PVS**) and click **Properties**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image025.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image025.png)
  
3.    Go to **Options**. And select **Full** as the recovery model for the database:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image026.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image026.png)
  
4.    **Right click** the PVS database again and click **Tasks --> Back Up..**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image027.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image027.png)
  
5.    Create a backup of the database:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image028.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image028.png)
  
6.    In SQL Management Studio, go to: **AlwaysOn High Availability --> Availability Groups**. Right click and choose: “**New Availability Group Wizard**”  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image029.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image029.png)
  
7.    At the Welcome Wizard, click **Next**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image030.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image030.png)
  
8.    **Enter the name of the new Availability group**. I.e.: BAG-CTX-PVS and click **Next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image031.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image031.png)
  
9.    Select the Citrix Provisioning Services database (**CTX-PVS**) and click **Next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image032.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image032.png)
  
10.    Click **Add Replica**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image033.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image033.png)
  
11.    Login to your second SQL server, click **Connect**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image034.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image034.png)

  
12.    Both servers are added. Select “**Automatic Failover**” and make sure “**Synchronous Commit**” is configured. Click **Listener** tab:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image035.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image035.png)
  
13.    Seed database using a temporary **SMB share**, click **next**:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image036.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image036.png)
  
14.    When the availability group validation is successful, click **next**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image037.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image037.png)
  
15.    Verify configuration and click **next**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image038.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image038.png)
  
16.    Creating the availability group:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image039.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image039.png)
  
17.    Creation of availability group is successful, click **Close**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image040.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image040.png)
  
18.    Availability group from a SQL point of view:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image041.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image041.png)

Reconfigure Citrix Provisioning Services to use the MultiSubnetFailover availability group
------------------------------------------------------------------------------------------
The Citrix Provisioning Server services are still stopped on all of your Citrix Provisioning Services Servers at this point. Reconfigure Citrix Provisioning Services to use the Microsoft SQL MultiSubnet (Basic) Availability group as follows:  
  
1.    Run this procedure on all of your Provisioning Services Servers one by one  
2.    On the Citrix Provisioning Services Server start “**Provisioning Services Configuration Wizard**”  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image042.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image042.png)
  
3.    I’ve skipped some default steps of the wizard.  
4.    On the “**Farm Configuration Screen**” choose: **Join existing farm** and click **next**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image043.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image043.png)
  
5.    Enter the SQL listener FQDN of the BAG created in SQL and enable “**Enable MultiSubnetFailover for SQL Server Always On**”. Click **Next**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image044.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image044.png)
  
6.    Select Farm:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image045.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image045.png)
  
7.    Choose the correct PVS Site for this PVS Server  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-Untitled.png)]({{ site.url }}/images/2018-02-26/2018-02-26-Untitled.png)
  
8.    Choose the correct Store for vDisks and click **Next**  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image047.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image047.png)
  
9.    Citrix Provisioning Services services are reconfigured and Provisioning Services started, click Done:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image048.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image048.png)
  
10.    Farm properties in Provisioning Services Console:  
[![]({{ site.url }}/images/2018-02-26/2018-02-26-image049.png)]({{ site.url }}/images/2018-02-26/2018-02-26-image049.png)