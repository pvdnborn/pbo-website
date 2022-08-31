---
layout: post
title:  "How to: Install and configure Microsoft SQL Server 2016 Standard multi-subnet Basic Availability Groups for Citrix XenDesktop and XenMobile"
date:   2017-11-06 20:16:00 +0100
categories:  XenDesktop, BAGs, multi-subnet, XenMobile, Failover cluster, Provisioning Services, Citrix, Clustering, AlwaysOn, SQL Server 2016 Standard, SQL Server, Failover, Basic Availability Groups, XenApp
---

Last months I was working with [@JanPaulPlaisier](https://twitter.com/JanPaulPlaisier) and [@AntonvanPelt](https://twitter.com/AntonvanPelt) on a interesting project: "_Create a high available Citrix XenApp and Citrix XenMobile environment between to different data-centers_". In this project we’ve also used NetScaler Global Server Load Balancing for redirecting end-users to the correct active data center. The setup is an active/standby set-up where both data-centers are online. The network latency between the two data centers is 4ms.  
  
In the next series of blogs I want to share some knowledge we’ve learned during this project, starting with creating a “multi-subnet SQL AlwaysOn” with SQL Server 2016 Standard.  
  
With the introduction of SQL Server 2016, Microsoft added a new AlwaysOn feature to the Standard product called “Basic Availability Groups” (BAGs). These BAGs are limited compared to AlwaysOn Availablity Groups (AAGs) which is the feature for Enterprise licensed customers:  

*   BAGs are limited to one database. So every database needs it’s own availability group. 
*   Backup to the database is limited to the primary database only. So your backup vendor needs to support BAGs. 
*   No read access to the secondary database.

For a complete overview of Always On Availability Groups: [https://docs.microsoft.com/en-us/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server](https://docs.microsoft.com/en-us/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server)  
  
If you are using a passive SQL Server instance for failover and the SQL Servers are covered with active SA, then no additional SQL licenses are needed for the failover SQL Server. SQL Server 2016 Licensing Guide: [http://download.microsoft.com/download/9/C/6/9C6EB70A-8D52-48F4-9F04-08970411B7A3/SQL\_Server\_2016\_Licensing\_Guide\_EN\_US.pdf](http://download.microsoft.com/download/9/C/6/9C6EB70A-8D52-48F4-9F04-08970411B7A3/SQL_Server_2016_Licensing_Guide_EN_US.pdf) . So we can use Basic Availability Groups with a SQL Standard with no additional licenses for the passive failover node.  
  
Citrix is supporting SQL AlwaysOn Availability Groups and Citrix changed the most preferred database availability method to “AlwaysOn” since XenDeskop 7.7.  
   
In this blog I will create a multi-subnet AlwaysOn SQL setup with the following components:  

*   PBO-SQL01 is the primary SQL-node in datacenter Amersfoort; (SQL Standard 2016)
*   PBO-SQL02 is the secondary SQL-node in datacenter Nijkerk; (SQL Standard 2016)
*   PBO-FS01 (Nano server) is the file share witness in the datacenter Utrecht;

Logical overview:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image002.png)](https://3.bp.blogspot.com/-yrKEHB2mRl8/WgB-HN7A29I/AAAAAAAAB9A/X9fwoQFcK5oF4qpHiA6jHdKfrdG0z6m0QCLcBGAs/s1600/image002.png)

This blog contains the following sections:  
1.  Create an OU and service account for SQL
2.  Assign user rights to SQL Service account for better SQL performance
3.  Optimal disk configuration for storing User, Log, Temp and Sys Database files
4.  Install Failover clustering feature
5.  Configure Windows Failover Clustering
6.  Configure a file share witness
7.  Configure Active Directory and DNS permissions for the Failover Cluster
8.  Installing SQL Server 2016 Standard
9.  Configure SQL Service for AlwaysOn Availability
10.  Install SQL Management tools

Create an OU and service account for SQL
----------------------------------------
It is a best practice to run the SQL Service with a Service Account. When the SQL Service is configured with a service account, we can apply some additional “user rights” to the service account. These “user rights” will have a positive impact on the performance of your SQL Server.  
  
In my lab Active Directory I’ve created the Domain User **SQLSvc@pbo.lan**. This is the guy we will using for the SQL configuration:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image003.png)](https://1.bp.blogspot.com/-UMN0kWNkrGM/WgB-HKN8bdI/AAAAAAAACB4/NVZKga4RST04YUKBgJrdb0ZBipB3xxtOQCEwYBhgL/s1600/image003.png)   
For the SQL-Servers I’ve created a new OU. Windows Failover Cluster Service will create cluster computer objects in this OU:  

[![]({{ site.url }}/images/2017-11-06/2017-11-06-image004.png)](https://1.bp.blogspot.com/-wMR6XaT8E88/WgB-HGwBYdI/AAAAAAAACB4/K2IDnS9ZGGIVB9TE52hSJIEL-wxr8H_GwCEwYBhgL/s1600/image004.png)

Assign user rights to SQL Service account for better SQL performance
--------------------------------------------------------------------
For better SQL performance we want to add the following user rights to the SQL account:  

*   Lock pages in memory (No swapping of memory used by SQL Service)
*   Perform volume maintenance tasks (No NTFS right evaluation)

_Lock pages in memory  
This security setting determines which accounts can use a process to keep data in physical memory, which prevents the system from paging the data to virtual memory on disk. Exercising this privilege could significantly affect system performance by decreasing the amount of available random access memory (RAM). Source: GPMC.MSC_  
   
_Perform volume maintenance tasks_  
_This security setting determines which users and groups can run maintenance tasks on a volume, such as remote defragmentation.  
Use caution when assigning this user right. Users with this user right can explore disks and extend files in to memory that contains other data. When the extended files are opened, the user might be able to read and modify the acquired data. Source: GPMC.MSC_  
  
1.    Create a new GPO on the SQLAlwaysOn container  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image005.png)](https://4.bp.blogspot.com/-lkgP9Hnqnj4/WgB-HTKOPMI/AAAAAAAACB4/FQp-O3Fi5j8TNdbYfNF8lhYYdcp_aQ_7wCEwYBhgL/s1600/image005.png)  

2.    Edit the new GPO and go to: **Computer Configuration --> Policies --> Windows Settings --> Local Policies --> User Right Assignment**. Double click **Lock pages in memory**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image006.png)](https://2.bp.blogspot.com/-5I_yCnlLhg0/WgB-H38h-2I/AAAAAAAACB4/7ucJO1brlmYpgHfPJ4vOidyNdWef1YiygCEwYBhgL/s1600/image006.png)  

3.    Add your **SQL Service account**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image007.png)](https://3.bp.blogspot.com/-YvFGzI7vGtc/WgB-If_4PZI/AAAAAAAACB8/YVsjRw1hmTcxWvpgAzDrEOXpzbVsblsXwCEwYBhgL/s1600/image007.png)

4.    Double click **Perform volume maintenance tasks**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image008.png)](https://3.bp.blogspot.com/-KKfiSsyHcps/WgB-I7CN1vI/AAAAAAAACB4/74RhF04dhLw-nAYYAplY6DyboAng5ZTuwCEwYBhgL/s1600/image008.png)

5.    Add **Administrators** and your **SQL Service account**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image009.png)](https://4.bp.blogspot.com/-vBGPhp02BE4/WgB-JFfhP-I/AAAAAAAACB4/EQ5-qdQsEPcl9aFUMjou6XmCMLQYjzylQCEwYBhgL/s1600/image009.png)

6.    In my lab setup I’ve added the SQL Service account to the local administrators group. Go to **Computer Configuration --> Preferences --> Local Users and Groups**. Right click and choose **New --> Local Group**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image010.png)](https://4.bp.blogspot.com/-HtFlctll2dc/WgB-J__98TI/AAAAAAAACB4/riQ1PJQm52s5txCzTjPeR9_yZaCGefZVACEwYBhgL/s1600/image010.png)

7.    Add your **SQL service account** to the local Administrators group as follows:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image011.png)](https://1.bp.blogspot.com/-zSRNuyU6msA/WgB-KDrqqcI/AAAAAAAACB8/wX4u-v3Ne4wd0O_whxTTXasBM32bN_dAwCEwYBhgL/s1600/image011.png)

Optimal disk configuration for storing User, Log, Temp and Sys Database files
-----------------------------------------------------------------------------
Configure your SQL-Servers with the following disk configuration:  
Drive    LAB - Size    Usage  
C:\\    64 GB    Windows Server 2016 Operating System  
D:\\    10 GB    SQL User database files  
E:\\    10 GB    SQL Log database files  
F:\\    5 GB    SQL Temp database files  
G:\\    5 GB    SQL System database files  
  
I always advise customers to use a separate disk for each partition. Extending partitions in the future is simple when using separate disks:  

[![]({{ site.url }}/images/2017-11-06/2017-11-06-image012.png)](https://1.bp.blogspot.com/-gjy2j4OzVns/WgB-KqENABI/AAAAAAAACB4/xf2Z9D1-OIEEzxz8TYgHyUYNGucdfp3CwCEwYBhgL/s1600/image012.png)

Install Failover clustering feature
-----------------------------------
Install the Failover clustering feature on both of your SQL-Servers. In my example PBO-SQL01 and PBO-SQL02.  
  
1.    Open **Server Manager** and click **Add roles and features**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image013.png)](https://1.bp.blogspot.com/-Xvb9o3XnPlE/WgB-LZMaFwI/AAAAAAAACBw/MYwCm8UbcyMu8eG1Gp2Tz6fBtTaNAFltwCEwYBhgL/s1600/image013.png)

2.    Add feature **Failover Clustering** and click **Next**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image014.png)](https://2.bp.blogspot.com/-XDyxeRHsCew/WgB-LrT-WII/AAAAAAAACBw/Y9EzGIXFEKI7pS4fuU0kt40HGXczcPwIwCEwYBhgL/s1600/image014.png)

3.    The Failover Clustering feature installation progress:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image015.png)](https://4.bp.blogspot.com/-3D5W_AZEA0Q/WgB-LrEQbKI/AAAAAAAACB8/eW5RxApovycxeRj14C8d_WiNLUg2dhRiQCEwYBhgL/s1600/image015.png)

4.    Click **finish** when done:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image016.png)](https://1.bp.blogspot.com/-rQEghCFng9U/WgB-L8sr_WI/AAAAAAAACBw/kSoDdnDvgmYcxGXK09n1-AB9zO3YQkt9wCEwYBhgL/s1600/image016.png)

5.    Install all Windows updates to both SQL-servers. Ensure installation of both servers are identical:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image017.png)](https://2.bp.blogspot.com/-e_y7iCYGvEg/WgB-L2n-jNI/AAAAAAAACB8/dUnr-GdKXxEq6Cc5mMPG5ZiGdWE1cRSTQCEwYBhgL/s1600/image017.png)

Configure Windows Failover Clustering
-------------------------------------
1.    Go to one of your SQL servers and open **Failover Cluster Manager**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image018.png)](https://4.bp.blogspot.com/-bVuQu7TcC2M/WgB-MZjlZdI/AAAAAAAACB8/XwJv6aXrk0QH69OlDzNacZImaZBrhkDFgCEwYBhgL/s1600/image018.png)

2.    Click “**Validate Configuration..**”  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image019.png)](https://1.bp.blogspot.com/-asrAvTudFW0/WgB-MlW9yhI/AAAAAAAACBw/EfmrUMlJAzs0Tpe_DRS-3TgjeCa01sleQCEwYBhgL/s1600/image019.png)

3.    Welcome wizard, click **Next**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image020.png)](https://4.bp.blogspot.com/-6Axu_zUYbew/WgB-MeY3k7I/AAAAAAAACB4/NAEzn-pDpmsrLP_Xn4rcKnpGm1QPOjlUACEwYBhgL/s1600/image020.png)

4.    Select both SQL-servers for validating, click **Next:**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image021.png)](https://2.bp.blogspot.com/-CZ4VVXl10Yw/WgB-M2Zf-7I/AAAAAAAACBw/PI2G_ucuW3IlCWNy562d10ZSlTgtoZ69wCEwYBhgL/s1600/image021.png)

5.    Select “**Run all tests (recommended)**” and click **Next**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image022.png)](https://1.bp.blogspot.com/-vdZ5bsDStqs/WgB-M0p7iXI/AAAAAAAACBw/VT4OyAmTKx0jL9F4RWPwTVIYCm5NOnZ_ACEwYBhgL/s1600/image022.png)

6.    On the confirmation dialog, click **Next**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image023.png)](https://1.bp.blogspot.com/-aNb29rsorJY/WgB-NMqS5KI/AAAAAAAACB8/lvGYTv6pnIALwBxtBP2ePnGDbVUkygRBwCEwYBhgL/s1600/image023.png)

7.    Both SQL-servers are validating:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image024.png)](https://4.bp.blogspot.com/-rqvMgCy_1r4/WgB-NxYgO8I/AAAAAAAACB4/L0825lkWygAe9uKtQbkg0ei8sJMJciG7ACEwYBhgL/s1600/image024.png)

8.    Select “**Create the cluster now using the validated nodes…**” and click **Finish**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image025.png)](https://4.bp.blogspot.com/-FE-6iblrC6Y/WgB-OON1TVI/AAAAAAAACBw/0-VvZMSUlfwqnJf3qu1slQ9wETJqUBZZgCEwYBhgL/s1600/image025.png)
  
The cluster creation starts.  
9.    Welcome wizard, click **Next**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image026.png)](https://4.bp.blogspot.com/-XxFn8USERQg/WgB-OjpKWwI/AAAAAAAACCA/IDcKwVKsu9sZDKveapyI9vx-0Kf12yDKwCEwYBhgL/s1600/image026.png)

10.    Enter a name for your cluster. And configure IP-addresses in both subnets for the cluster. Click **Next**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image027.png)](https://3.bp.blogspot.com/-YTP9v1Npdt8/WgB-PNBSfrI/AAAAAAAACB8/IZijYkhnelwEkjBna9TrjRYfO_nmb4GzACEwYBhgL/s1600/image027.png)

11.    Deselect “**Add all eligible storage to the cluster.**” and click **Next**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image028.png)](https://3.bp.blogspot.com/-D-Ou3OEp23g/WgB-P6RF5tI/AAAAAAAACBw/eZtyo-FoJcIorc7FxMLUWZ7nP5cmO_hOACEwYBhgL/s1600/image028.png)

12.    Forming cluster:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image029.png)](https://2.bp.blogspot.com/-vDAnnI8fyx4/WgB-QIUStgI/AAAAAAAACBw/8O4eFa7JZo8eTnKTHv09X6DRp5pO7tJFwCEwYBhgL/s1600/image029.png)

13.    Cluster creation completed successfully, click **Finish**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image030.png)](https://4.bp.blogspot.com/-hQaDobadPD0/WgB-QMKhiQI/AAAAAAAACB8/rdH4ALRstDk0ayvreAzwH51Tyo6Z47AewCEwYBhgL/s1600/image030.png)
  
In a multi-subnet failover cluster, only one IP-address is up on the primary site:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image031.png)](https://3.bp.blogspot.com/-vIqlbsNhZOI/WgB-QV2TwoI/AAAAAAAACBw/Y3lhAg4I2qsDc_8bL9D_xJs6hnbEcVLwQCEwYBhgL/s1600/image031.png)

Configure a file share witness
------------------------------
The cluster doesn’t have a file share witness after creation by default. The file share witness is needed to create an odd value of cluster votes. In a failover situation, the SQL-server with majority of the total votes will become primary SQL-Server. For example if the connection to data center Amersfoort is down, the PBO-SQL01 has one vote and the PBO-SQL02 has two votes. The PBO-SQL02 will become primary and the PBO-SQL01 secondary:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image032.png)](https://2.bp.blogspot.com/-XEt3l5Obz8M/WgB-Qh9ZDiI/AAAAAAAACB4/opdqKAK5OV4ejtKhCp-aF5rUgxxzTqyQQCEwYBhgL/s1600/image032.png)
  
Create a file share with the following permissions:  
1.    Create a folder on your witness server:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image033.png)](https://1.bp.blogspot.com/-UihvhlsTazw/WgB-ROSDDoI/AAAAAAAACB4/FfTzQfs5TusMvxb9vuoXpbqksdTnRx5iQCEwYBhgL/s1600/image033.png)

2.    Assign the **SQL-Service account** and the **Cluster computer account** _**Full Control**_ on the folder:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image034.png)](https://2.bp.blogspot.com/-EsQIeuPtohM/WgB-RazckeI/AAAAAAAACCA/af0MwJOm1FQ9Hbe1uvnpMGeGErVmd571ACEwYBhgL/s1600/image034.png)

3.    Create a share and assign **full control** share permissions to the **SQL-Service** and **Cluster computer account**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image035.png)](https://1.bp.blogspot.com/-u5wjs33ekEA/WgB-SBtjhGI/AAAAAAAACBw/j6aUVXWJwkMSRUn3B_pPk7INABt30KZsgCEwYBhgL/s1600/image035.png)
  
Add file share witness to the Failover Cluster as follows:  
1.    Go to one of your SQL servers and open **Failover Cluster Manager**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image018.png)](https://4.bp.blogspot.com/-bVuQu7TcC2M/WgB-MZjlZdI/AAAAAAAACB8/XwJv6aXrk0QH69OlDzNacZImaZBrhkDFgCEwYBhgL/s1600/image018.png)
  
2.    Right click your cluster. Click More **Actions --> Configure Cluster Quorum Settings…**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image036.png)](https://4.bp.blogspot.com/-FzfLjPdAQV4/WgB-R0TsvoI/AAAAAAAACBw/adcHxeDKhXcpNgJQZKGa4CPQ_rwd-iTOQCEwYBhgL/s1600/image036.png)

3.    Welcome wizard, click **Next**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image037.png)](https://1.bp.blogspot.com/-Z9SHqTaKXnY/WgB-R_jXuII/AAAAAAAACBw/LwNwB38w84w0FmnP08CGoH4N5OwYE2FlgCEwYBhgL/s1600/image037.png)

4.    Select “**Select the quorum witness**” and click **Next**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image038.png)](https://1.bp.blogspot.com/-GvjglPB6PXc/WgB-STiZpKI/AAAAAAAACB4/x32U1V7uhtUs8kCcQyG51ttDiwYcCIHVwCEwYBhgL/s1600/image038.png)

5.    Select **Configure a file share witness** and click **Next**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image039.png)](https://2.bp.blogspot.com/-ESlgZ1RYtHA/WgB-SUidK9I/AAAAAAAACBs/VYXFPWjX5xQDpiFCANB9ZPqHV58WDlskQCEwYBhgL/s1600/image039.png)

6.    Configure the **File Share Path** with the **FQDN UNC** Path to your file share quorum share, click **Next**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image040.png)](https://2.bp.blogspot.com/-QQYv3c3COc4/WgB-Ss12SAI/AAAAAAAACBs/2dmqezHgs5giafrlAX8m655cF8t-sficgCEwYBhgL/s1600/image040.png)

7.    Confirmation dialog, click **Next**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image041.png)](https://2.bp.blogspot.com/-7Nw4p-qK96o/WgB-SnggvdI/AAAAAAAACBw/ecP43poCGt0WOZlfAdUd8ki6rTBONzxhQCEwYBhgL/s1600/image041.png)

8.    Click **Finish**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image042.png)](https://2.bp.blogspot.com/-Sh00qsLa3F4/WgB-S6M8USI/AAAAAAAACBs/JyPsPV7Ee4QuHPOEB7mF6QE_FjsTzulMACEwYBhgL/s1600/image042.png)

  
The file share witness is added to the cluster:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image043.png)](https://1.bp.blogspot.com/-sj9s06o5J1A/WgB-TbRZOXI/AAAAAAAACBs/rBtsa0M6K08x8jnIOzHmd5hQJsDHmS7ugCEwYBhgL/s1600/image043.png)

Configure Active Directory and DNS permissions for the Failover Cluster
-----------------------------------------------------------------------
The Failover cluster need additional permissions to Active Directory and DNS to add new cluster resources (BAGs) and manage IP-addresses of the A-records.  
  
1.    Open **dsa.msc** and navigate to the OU we’ve created for SQL AlwaysOn:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image044.png)](https://1.bp.blogspot.com/-88KEdjN8Gd8/WgB-TtsbAqI/AAAAAAAACBs/YFJkYqpHEsQPsK5UgegHATfC8qRmgldogCPcBGAYYCw/s1600/image044.png)  

2.    Click **View --> Advanced Features**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image045.png)](https://2.bp.blogspot.com/-BDZbgEQkXWs/WgB-UOCDHxI/AAAAAAAACBw/XTWlooR0agUDLNVKryjc7XK1sOs1FJwLQCPcBGAYYCw/s1600/image045.png)

3.    Right click on the OU and click **Properties**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image046.png)](https://3.bp.blogspot.com/-FOh24U_mfoY/WgB-UARFOiI/AAAAAAAACBs/88LHfkRrVlcDHyKkuKI_KM_eXDiFU0uDgCPcBGAYYCw/s1600/image046.png)

4.    Click **Add**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image047.png)](https://2.bp.blogspot.com/-Ja1Jifrygc8/WgB-UnUW79I/AAAAAAAACBs/1IDm7G-7dpczrhV6BPo5qNKF7PO0XyCwACPcBGAYYCw/s1600/image047.png)

5.    Click **Object Types**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image048.png)](https://1.bp.blogspot.com/-0SBG9CmYyMk/WgB-Yopp0GI/AAAAAAAACBw/1H18rijNX0YpfDsyzrtTq8HJXwIeyxJwwCPcBGAYYCw/s1600/image048.png)

6.    Select **Computers** and click **OK**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image049.png)](https://4.bp.blogspot.com/-O_XP5vXXxn4/WgB-aO1ql9I/AAAAAAAACBs/Z39tF3yvqQIpB8Jm1OwIEHfOmBtglh4tQCPcBGAYYCw/s1600/image049.png) 

7.    Add **Failover Cluster computer** object and click OK:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image050.png)](https://3.bp.blogspot.com/-LDxSSIvB_Mg/WgB-ZB14rnI/AAAAAAAACBs/oVcPqW8Us20ytGaL25uk__7qtG-cyM12QCPcBGAYYCw/s1600/image050.png)

8.    Click **Advanced**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image051.png)](https://4.bp.blogspot.com/-LeDdtnn5yIs/WgB-ZX8Uu2I/AAAAAAAACBw/-3aBLceNQFcrlK2Inosr3hr1gRivM8o_gCPcBGAYYCw/s1600/image051.png)

9.    Select **Failover cluster computer object** and click **Edit**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image052.png)](https://1.bp.blogspot.com/-5uj4rjk6RXw/WgB-Zpb8jHI/AAAAAAAACBs/XS0gD0BooEUbt8vCpwjC7lSLH-xuUJY4QCPcBGAYYCw/s1600/image052.png)  

10.    Add “**Create computer objects**” and click **OK**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image053.png)](https://1.bp.blogspot.com/-NOLuQe99fVE/WgB-ZsWF69I/AAAAAAAACBs/pjn2RfVJXys-6-BqUwnvHc4MUMIYEAG6QCPcBGAYYCw/s1600/image053.png)

11.    **OK** everything to complete Active Directory permissions for cluster computer object  

Now we need to grant the failover cluster computer object the permission to add DNS-records to the primary zone of your domain.  
  
12.    Open **DNS Management** (dnsmgmt.msc)  

13.    Right click the **primary zone** of your domain. Then click **Properties**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image054.png)](https://4.bp.blogspot.com/-7dxXfd63SQA/WgB-Z3b7vXI/AAAAAAAACBw/r9-qFnxSPjAeeCRVqRbCBsjXzVjGm6aGwCPcBGAYYCw/s1600/image054.png)

14.    Go to the **Security tab** and click **Add**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image055.png)](https://1.bp.blogspot.com/-k3fEc47n7HM/WgB-aL0JR7I/AAAAAAAACBs/eC3kbDwlxV09dB3YEmGTbz40tlc6df2iQCPcBGAYYCw/s1600/image055.png)

15.    Select object type **Computers,** then click **OK**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image056.png)](https://3.bp.blogspot.com/-GOaZ75hTtQU/WgB-aVwg5sI/AAAAAAAACBs/u-YAEwEEuEcGlucUWTYdmBMguchBcGLIACPcBGAYYCw/s1600/image056.png)

16.    Add **Failover cluster computer** object, click **OK**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image057.png)](https://3.bp.blogspot.com/-Ma3c_Aa04uk/WgB-aSuyHVI/AAAAAAAACBw/tvC4K9U6ulAJhy_59WlF14VYTQeDqSKiACPcBGAYYCw/s1600/image057.png)

17.    Assign the **Create all child objects** permissions and click **OK**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image058.png)](https://3.bp.blogspot.com/-bsd580bjP-A/WgB-a1aQQJI/AAAAAAAACBs/txAWvAYCy0UMuJsLqpQhztoyZIQbD3p-QCPcBGAYYCw/s1600/image058.png)

Installing SQL Server 2016 Standard
-----------------------------------
On both SQL Servers, install SQL Server 2016 as follows:  
1.    Mount the SQL 2016 Standard DVD to the DVD-drive  
2.    The SQL Server Installation Center will start. Choose I**nstall --> New SQL stand-alone installation or add features to an existing installation**  

[![]({{ site.url }}/images/2017-11-06/2017-11-06-image059.png)](https://3.bp.blogspot.com/-2ISs8Q-zxkY/WgB-biyA1nI/AAAAAAAACB4/Ug-nFtWBf7sxYPaWEcM4SGSaLvOS9TVMQCPcBGAYYCw/s1600/image059.png)

3.    The SQL Server setup is started. Click **Next**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image060.png)](https://3.bp.blogspot.com/--3y6Oho5JZg/WgB-b12XxCI/AAAAAAAACBs/HZ_K_IEbpy0cJgg7g8gxdkiEksJeilUoACPcBGAYYCw/s1600/image060.png) 

4.    Check “**I accept the license terms.**” and click Next  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image061.png)](https://1.bp.blogspot.com/-Li_rX7gn8Tk/WgB-b9fdNCI/AAAAAAAACB4/WX34Fx213scTISjJe95LU8V9Ve3zU9wGwCPcBGAYYCw/s1600/image061.png) 

5.    Check “**Use Microsoft Update to check for update**s” and click Next:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image062.png)](https://3.bp.blogspot.com/-Z00ke6vZQ6U/WgB-cHlPQ_I/AAAAAAAACBs/ZleaDQY9O3Ua6OhheAZ89too4Hc-tgWPACPcBGAYYCw/s1600/image062.png) 

6.    Select features **Database Engine Services** and **SQL Server Replication** and change paths to **G:\\**. Click **Next**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image063.png)](https://2.bp.blogspot.com/-SseUhxiCBCA/WgB-cEV8QLI/AAAAAAAACBs/Wr9k9ZJoBAAa_gCqv6SiNGCwmZIuKoQlwCPcBGAYYCw/s1600/image063.png) 

7.    Default, click **Next**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image064.png)](https://2.bp.blogspot.com/-E4XxGit368s/WgB-cZCHM2I/AAAAAAAACBs/If4aTBesCFMtTGH48qwkRfWyT-NlO-zFgCPcBGAYYCw/s1600/image064.png) 

8.    Configure your **SQL Service account** with password. And click **Collation tab**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image065.png)](https://2.bp.blogspot.com/-CQCEIqE0z18/WgB-cXoJ7pI/AAAAAAAACBw/JZKhmDOhUvgizZpVcfKUlHaiLU6a-tTsQCPcBGAYYCw/s1600/image065.png) 

9.    Click **Customize**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image066.png)](https://1.bp.blogspot.com/-Blu4oClqZe0/WgB-dJzBHXI/AAAAAAAACBs/RJ97rn_U410yJnUiPoubRY1yKnsIWQp-wCPcBGAYYCw/s1600/image066.png) 

10.    Select “**Collation designator: Latin1\_General\_100**” and **Accent-sensitive** and **Kana-sensitive**. This is the collation used by the Citrix products, click **OK**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image067.png)](https://1.bp.blogspot.com/-4a9utYZkx4c/WgB-dYPD5-I/AAAAAAAACB4/Slt2Bph32ycp5MuuB7cor3tucFylYxjagCPcBGAYYCw/s1600/image067.png) 

11.    Collation has changed, click **Next**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image068.png)](https://4.bp.blogspot.com/-Ntx3aqjv46I/WgB-eESfgeI/AAAAAAAACBs/t9iDqkCggWcTv2D1Iq1A6j1TTDc3c4i5ACPcBGAYYCw/s1600/image068.png) 

12.    If you need SQL-authentication (RES Needs it) select Mixed Mode. Add your account or an Active Directory group to the SQL Server administrators. Click **Data Directories tab**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image069.png)](https://1.bp.blogspot.com/-_BwbJ4Cj-K0/WgB-efFWt_I/AAAAAAAACBs/uvRF_K5TeZ4iQYIWeQOGwLIWWtj_6VO-wCPcBGAYYCw/s1600/image069.png) 

13.    Configure data directories as follows, click **TempDB tab**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image070.png)](https://3.bp.blogspot.com/-jocRisbpWyY/WgB-evoZn1I/AAAAAAAACBs/5a1rurmlHf8sDPp8YHx5iXR_fvOhyTVQwCPcBGAYYCw/s1600/image070.png) 

14.    Edit TempDB path to **F-drive**, click **Next**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image071.png)](https://2.bp.blogspot.com/-LpwvUu-R1aQ/WgB-emE7kUI/AAAAAAAACBw/Iz265lpwzm88PGvy0ooU9HQSrnmnaqbGgCPcBGAYYCw/s1600/image071.png) 

15.    Click **Install** to start installation:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image072.png)](https://1.bp.blogspot.com/-k52y2fEr-nU/WgB-emwfcvI/AAAAAAAACBs/BIMopcJ2ihw9Ubep0EtkKlhdRMWB11wTwCPcBGAYYCw/s1600/image072.png) 

16.    SQL Server is installing:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image073.png)](https://4.bp.blogspot.com/-hFOVutSbbII/WgB-e_fDMpI/AAAAAAAACB4/ndrC5qdrzVAQ9RmE93Fk90b5Xl02mmdIACPcBGAYYCw/s1600/image073.png) 

17.    Installation finished successfully. Click **Close**[![]({{ site.url }}/images/2017-11-06/2017-11-06-image074.png)](https://2.bp.blogspot.com/-7v7LCQFQ1fA/WgB-fM95ljI/AAAAAAAACBs/nnZyKulLRwM856RToqtpdTO_ka7qwv8ygCPcBGAYYCw/s1600/image074.png)  

Configure SQL Service for AlwaysOn Availability
-----------------------------------------------
Enable the AlwaysOn Availabilty to the SQL service on both SQL-Servers as follows:  
  
1.    Open **SQL Server 2016 Configuration Manager**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image075.png)](https://3.bp.blogspot.com/-9RA3AvCjFSA/WgB-feIsMDI/AAAAAAAACBs/AyJ0kWnRae4cOmijFh1ouGvVcShPB50ZACPcBGAYYCw/s1600/image075.png)

2.    Go to **SQL Server Services --> SQL Server (MSSQLSERVER) --> Properties**:  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image076.png)](https://2.bp.blogspot.com/-8iP0Xl-_knw/WgB-fRjF1MI/AAAAAAAACBs/aPC2Py8H4AsXJ81m7Sy9XGqruLC6s3XtQCPcBGAYYCw/s1600/image076.png)

3.    Go to tab **AlwaysOn High Availability** and select **Enable AlwaysOne Availability Groups**. Click **OK**  
[![]({{ site.url }}/images/2017-11-06/2017-11-06-image077.png)](https://4.bp.blogspot.com/-kciuLHB_Flk/WgB-fvHZt2I/AAAAAAAACBw/-py0pNhZEegriLTjZDKq0C5Nf7uq-U7EACPcBGAYYCw/s1600/image077.png)

4.    **Restart** the **SQL Server (MSSQLSERVER)** Service[![]({{ site.url }}/images/2017-11-06/2017-11-06-image078.png)](https://2.bp.blogspot.com/-kQKFDiA9Kug/WgB-gACMg9I/AAAAAAAACBs/30BRq345bmIYijYVZlGk_MVJYddmlzIuACPcBGAYYCw/s1600/image078.png)

Install SQL Management tools
-----------------------------
SQL Management tools aren’t installed by default. Download and install the most recent SQL Management Studio from: [https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms) . Installation is straight forward (next, next, finish)