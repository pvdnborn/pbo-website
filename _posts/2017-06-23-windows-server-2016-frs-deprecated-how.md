---
layout: post
title:  "Windows Server 2016 - FRS Deprecated: How to migrate SYSVOL replication from FRS to DFS replication"
date:   2017-06-23 14:45:00 +0100
categories:  FRS to DFS, DFSR, DFS, How to, Windows 2003, Windows Server 2016, FRS to DFSR, FRS, SYSVOL
---

This blog contains a step-by-step guide: “how to migrate the SYSVOL FRS to DFS Replication (DFSR)”  

Windows Server 2016 Domain Controllers: FRS Replication deprecated!
-------------------------------------------------------------------
Since the introduction of Windows Server 2008, Microsoft moved away from FRS replication and introduced DFS replication for SYSVOL. With the introduction of Windows Server 2016 the old FRS SYSVOL replication is deprecated. For now (23-06-2017) this means the FRS feature is still there, but you will receive warnings while promoting a Windows 2016 DC and still using FRS. The FRS-feature will be removed in nearby future of new Windows 2016 releases. So migrate your SYSVOL FRS replication to DFSR before introducing new Windows 2016 Domain Controllers to your domain.  

**Customers don’t know they still using FRS SYSVOL replication after Windows 2008 or higher migration**
-------------------------------------------------------------------------------------------------------
As a consultant I see a lot of customers, which are running Windows 2008 or higher domains/forest functional levels and unconsciously still using the old FRS-replication technique. The FRS-replication is still in place because their domain/forest had a functional level of Windows 2003 R2 or lower in the past. The customer migrated away from Windows 2003 R2 to a domain/forest functional level of Windows 2008 or higher and didn’t migrate to DFSR. When the domain/forest functional level is raised to Windows 2008 or higher the SYSVOL replication doesn’t automatically upgrades to DFSR. You need to do this manually!  
  
This is also the case when you install a brand new domain with Windows 2012 R2 DC’s and configure the forest/domain functional level to Windows 2003 R2 or lower. In a domain with functional level of Windows 2003 R2 you can introduce Windows 2003 R2 Domain Controllers and since they don’t have DFSR technology the Windows 2008 or higher DC’s fall back to FRS to communicate with the 2003 R2 DC’s in your domain.  

**Check if you still using FRS replication**
--------------------------------------------
Check if you are still using FRS as follows:  

1.  On your Domain Controller the “File Replication Service” is present and is running. Startup type: Automatic.
2.  In the Application log events appear in the “File Replication Service” log. Which contains information about FRS replication and the message “The File Replication Service is no longer preventing the computer PBO-DC01 from becoming a domain controller….” is registered after a domain controller reboot.

Example: **[![]({{ site.url }}/images/2017-06-23/2017-06-23-image001.jpg)](https://2.bp.blogspot.com/-larOAnz1Jko/WU0FI_eGMtI/AAAAAAAABw4/6IV9zVGGBKUtuZOYjPykp0qu4_r7I4NhQCEwYBhgL/s1600/image001.jpg) Visual: The migration process with dfsrmig**  

The DFSRMIG process has three states:  
[![]({{ site.url }}/images/2017-06-23/2017-06-23-image003.jpg)](https://4.bp.blogspot.com/-dy6MlSNEhqQ/WU0FI_ePQlI/AAAAAAAABw4/iVrN2ed7Aw0RwxGkiGueVirQb2SQTFMngCEwYBhgL/s1600/image003.jpg)

Preparations
------------
**WARNING:** Execute the DFS migration on the Domain Controller which has the PDC-emulator role.  
  
INFO: Read the following technet blog for more detailed information about the FRS to DFSR migration: [http://blogs.technet.com/b/filecab/archive/2008/02/08/sysvol-migration-series-part-1-introduction-to-the-sysvol-migration-process.aspx](http://blogs.technet.com/b/filecab/archive/2008/02/08/sysvol-migration-series-part-1-introduction-to-the-sysvol-migration-process.aspx)  

1.  Create a backup of your domain (systemstate)
2.  Make sure you know the Active Directory Restore Mode password. If you are not 100% sure, reset the ADRM password to something you know. You will need this password if you want to restore your Active Directory backup.
3.  Ensure that the domain and forest level is at least Windows 2008 (you can use AD Trusts cpl)
4.  Check your Domain Controllers with **DCDIAG** and ensure there are no issues[![]({{ site.url }}/images/2017-06-23/2017-06-23-image005.png)](https://1.bp.blogspot.com/--JyQC0blGb0/WU0FJE8cKCI/AAAAAAAABw4/g8L7UQ53k50__IrUDXc7IwOyNvZfopISACEwYBhgL/s1600/image005.png)
5.  Check Active Directory replication: **repadmin /showrepl** . You need to ensure that AD replication is OK.  
    [![]({{ site.url }}/images/2017-06-23/2017-06-23-image007.png)](https://3.bp.blogspot.com/-5Z8f1mAYOGA/WU0FJgaM2MI/AAAAAAAABw4/kIqMyHvu_OUteXmRcv_WLY2ov5Xde7GlQCEwYBhgL/s1600/image007.png)
6.  Check if SYSVOL is shared/ready on every domain controller: **net share**  
    [![]({{ site.url }}/images/2017-06-23/2017-06-23-image009.png)](https://1.bp.blogspot.com/-nM9wzs4nkFI/WU0FJs0KJVI/AAAAAAAABw4/EhcrPRcvO8c01hGlsrK6NLCW8tnQ8KinwCEwYBhgL/s1600/image009.png)
7.  Check every Domain Controller if SYSVOL is ready:  **HKLM\\System\\CurrentControlSet\\Services\\Netlogon\\Parameters\\Sysvol \[SysvolReady\] = 1**  
    [![]({{ site.url }}/images/2017-06-23/2017-06-23-image011.png)](https://3.bp.blogspot.com/-1Exu5I_UxEE/WU0FJ97RhsI/AAAAAAAABw4/GYSVc03JCNsTpf0j_E9HzQMOM-izgIfCACEwYBhgL/s1600/image011.png)
8.  Check the current DFSR status and ensure you are not half way a DFSR migration already. **Dfsrmig /getMigrationState**  
    [![]({{ site.url }}/images/2017-06-23/2017-06-23-image013.png)](https://2.bp.blogspot.com/-3ws_zoLQbEw/WU0FKZXZ_7I/AAAAAAAABw4/Pa6oo8qOVpYf8EuaSdwaAR0gVqx8XlpJQCEwYBhgL/s1600/image013.png)
9.  Check if the DFS Service is installed on every domain controller:[![]({{ site.url }}/images/2017-06-23/2017-06-23-image015.png)](https://1.bp.blogspot.com/-A-EWXuPaI4Q/WU0FKvYlzbI/AAAAAAAABw4/-L0F9xGOHksc51FqPUINNco40ZRG8R5ugCEwYBhgL/s1600/image015.png)

Phase 1: Migrated to “PREPARED” state
-------------------------------------
If you have ensured done the preparation and you can fallback to a backup. Your good to go and start the FRS to DFSR migration as follows.  
  
**INFO:** The prepared state is a state where the DFSR Replication will be introduced parallel of the FRS Replication. When in DFSMig state “PREPARED”, the FRS Replication is leading.  
  
**WARNING:** The SYSVOL to SYSVOL\_DFSR copy is done once, so changes to SYSVOL are not replicated to SYSVOL\_DFSR!  
  
\=============================================  
_DFSR has successfully migrated the Domain Controller PBO-DC-01 to the 'PREPARED' state.  
  
TO CONTINUE MIGRATION: If you choose to continue the migration process and proceed to the 'REDIRECTED' state, please note that any changes made henceforth to the SYSVOL share located at C:\\Windows\\SYSVOL (which is under NTFRS replication) will not be updated in the SYSVOL\_DFSR folder located at C:\\Windows\\SYSVOL\_DFSR (which is under DFSR replication). **To avoid this possibility of data loss, please make sure no file system changes on the SYSVOL share occur while DCs are migrating from 'PREPARED' to 'REDIRECTED' state.**  
  
TO ROLLBACK MIGRATION: If you choose to rollback the migration process and return to the 'START' state, please note that DFSR will no longer be replicating the SYSVOL\_DFSR folder and all DFSR information will be removed from the Active Directory.  
  
Additional Information:  
Sysvol NTFRS folder: C:\\Windows\\SYSVOL  
Sysvol DFSR folder: C:\\Windows\\SYSVOL\_DFSR  
Domain Controller: PBO-DC-01_  
\=============================================  

1.  Execute command **dfsrmig /setGlobalState 1**  
    [![]({{ site.url }}/images/2017-06-23/2017-06-23-image017.png)](https://1.bp.blogspot.com/-ZchRnRrpzSk/WU0FKyp4SgI/AAAAAAAABw4/_Jgb8Ke9piYQYW09I1jm_GcSIaDZ7TIZwCEwYBhgL/s1600/image017.png)
2.  Check the progress with command: **dfsrmig /getMigrationState**[![]({{ site.url }}/images/2017-06-23/2017-06-23-image019.png)](https://4.bp.blogspot.com/-j7bez--swWU/WU0FK7AwsVI/AAAAAAAABw4/jOGzLzeaDt8LbBTpRddGwtzMitkMPvM8gCEwYBhgL/s1600/image019.png)

Output – not completed:  
[![]({{ site.url }}/images/2017-06-23/2017-06-23-image019.png)](https://4.bp.blogspot.com/-j7bez--swWU/WU0FK7AwsVI/AAAAAAAABw4/jOGzLzeaDt8LbBTpRddGwtzMitkMPvM8gCEwYBhgL/s1600/image019.png)  
Output – Completed:  

[![]({{ site.url }}/images/2017-06-23/2017-06-23-image021.png)](https://3.bp.blogspot.com/-YOYd0506dpM/WU0FLLmMYzI/AAAAAAAABw4/IgHOh0FIYEY7icrENKyfRXoivB8LURg5QCEwYBhgL/s1600/image021.png)

Phase 2: Migrate to “REDIRECTED” state
--------------------------------------
**INFO:** In a redirected state the DFSR will become primary. In this state you still have the opportunity to rollback to FRS replication.  
  
**WARNING:** There could be data loss when rolling back to FRS replication:  
\=============================================  
_DFSR has successfully migrated the Domain Controller PBO-DC-01 to the 'REDIRECTED' state. DFSR is now replicating the SYSVOL\_DFSR folder located at C:\\Windows\\SYSVOL\_DFSR.  
  
TO CONTINUE MIGRATION: If you choose to continue the migration process and proceed to the 'ELIMINATED' state, please note that it will not be possible to revert the migration process. Once migration reaches the 'ELIMINATED' state, the SYSVOL folder located at C:\\Windows\\SYSVOL will be deleted and NTFRS will no longer replicate it. All the NTFRS related information in the Active Directory will be deleted. After that, DFSR will be solely responsible for the SYSVOL share replication process on the Domain Controller PBO-DC-01.  
  
TO ROLLBACK MIGRATION: If you choose to rollback the migration process to the 'PREPARED' state, any changes made after moving to the 'REDIRECTED' state to the SYSVOL\_DFSR folder located at C:\\Windows\\SYSVOL\_DFSR (which is currently under DFSR replication) will not be updated in the SYSVOL share located at C:\\Windows\\SYSVOL (which is under NTFRS replication). **To avoid this possibility of data loss post rollback, please make sure that no file system changes on the SYSVOL share  occur while DCs are migrating from 'REDIRECTED' to 'PREPARED' state.**_  
_  
After rollback, NTFRS will replicate the SYSVOL share located at C:\\Windows\\SYSVOL and DFSR will continue replicating the SYSVOL\_DFSR folder located at C:\\Windows\\SYSVOL\_DFSR. However, NTFRS will still primarily be responsible for the SYSVOL share replication process on the Domain Controller PBO-DC-01.  
  
Additional Information:  
Sysvol NTFRS folder: C:\\Windows\\SYSVOL  
Sysvol DFSR folder: C:\\Windows\\SYSVOL\_DFSR  
  
Domain Controller: PBO-DC-01_  
\=============================================  
1.  Check if SYSVOL folder is still shared: **net share**[![]({{ site.url }}/images/2017-06-23/2017-06-23-image023.png)](https://2.bp.blogspot.com/-7MAqg9LSYJo/WU0FLQmcK6I/AAAAAAAABw4/MGZIZ2hV3BcvcdkcKEpnt68bpe5lxjTTgCEwYBhgL/s1600/image023.png)
2.  Check if SysvolReady has value 1[![]({{ site.url }}/images/2017-06-23/2017-06-23-image024.png)](https://2.bp.blogspot.com/--9msN8L_I3E/WU0FLR7QCgI/AAAAAAAABw4/anrbl-aCZyY1brm043YlCvo56nLwcoyWgCEwYBhgL/s1600/image024.png)
3.  Check if the AD replication is still OK: **repadmin /showrepl or repadmin /replsum**  
    [![]({{ site.url }}/images/2017-06-23/2017-06-23-image026.png)](https://1.bp.blogspot.com/-I9m8fmid8IM/WU0FL2mnmfI/AAAAAAAABw4/wJMfRnOhXgwX4g6fNUCLKC-41lqMdMQHgCEwYBhgL/s1600/image026.png)
4.  Execute the following command to get to the REDIRECTED state: **dfsrmig /setGlobalState 2**  
    [![]({{ site.url }}/images/2017-06-23/2017-06-23-image028.png)](https://4.bp.blogspot.com/-k6-IH0tbu2c/WU0FL4svNeI/AAAAAAAABw4/iAVSUPmvRrYxMOXaNfURFFGR-xyoMTRfgCEwYBhgL/s1600/image028.png)
5.  Check the progress with command **dfsrmig /getMigrationState**

Output – not completed:  
[![]({{ site.url }}/images/2017-06-23/2017-06-23-image030.png)](https://2.bp.blogspot.com/-LqS-UWVhilE/WU0FMUgd80I/AAAAAAAABw4/0UHSp4YCgg8cecOrGXiPyqtjyeiGOOoxACEwYBhgL/s1600/image030.png)Output – Completed:  

[![]({{ site.url }}/images/2017-06-23/2017-06-23-image032.png)](https://4.bp.blogspot.com/-1kd-puutWVo/WU0FMcTHA-I/AAAAAAAABw4/ZhxhcvPQdhMtw4l-o1hLB1GNVX-6pNKagCEwYBhgL/s1600/image032.png)

Phase 3: Migrate to “ELIMINATED” state
--------------------------------------
**WARNING:** There is no rollback possibility to FRS when migrated to the ELIMINATED state.  

1.  Check if SYSVOL folder is still shared: **net share**[![]({{ site.url }}/images/2017-06-23/2017-06-23-image033.png)](https://4.bp.blogspot.com/-5usmJiu520c/WU0FMvDZG0I/AAAAAAAABw4/h1f1qk7A2kQ-IVxvQ56q28YF8yqzv57DQCEwYBhgL/s1600/image033.png)
2.  Check if SysvolReady has value 1[![]({{ site.url }}/images/2017-06-23/2017-06-23-image035.png)](https://1.bp.blogspot.com/-XUmuKN7FhpY/WU0FM4vHg2I/AAAAAAAABw4/uBiRPmTbpj8ugbryu06dB247NkklxF2WwCEwYBhgL/s1600/image035.png)
3.  Check if the AD replication is still OK: **repadmin /showrepl or repadmin /replsum**  
    [![]({{ site.url }}/images/2017-06-23/2017-06-23-image026.png)](https://1.bp.blogspot.com/-I9m8fmid8IM/WU0FL2mnmfI/AAAAAAAABw4/wJMfRnOhXgwX4g6fNUCLKC-41lqMdMQHgCEwYBhgL/s1600/image026.png)
4.  Execute the following command to get to the ELIMINATED state: **dfsrmig /setGlobalState 3**  
    [![]({{ site.url }}/images/2017-06-23/2017-06-23-image037.png)](https://2.bp.blogspot.com/-dW63A_tcjZE/WU0FNNRcRYI/AAAAAAAABw4/dhgJOAWeG0AIXFtk_PKNp9Txxii90WRXwCEwYBhgL/s1600/image037.png)
5.  Check the progress with command **dfsrmig /getMigrationState**[![]({{ site.url }}/images/2017-06-23/2017-06-23-image041.png)](https://3.bp.blogspot.com/-aSk3sdIZMCg/WU0FNa49EwI/AAAAAAAABw4/2MvrCemCKwg84ApZTPqLP8R9Ef9wSccZgCEwYBhgL/s1600/image041.png)

Output – Not completed:  
[![]({{ site.url }}/images/2017-06-23/2017-06-23-image039.png)](https://2.bp.blogspot.com/-yFRyy06k1KE/WU0FNKPYUUI/AAAAAAAABw4/GErSGbXY8vsrmftVUxjS0HYUHSZZCvyeACEwYBhgL/s1600/image039.png)  
Output – Completed:  

[![]({{ site.url }}/images/2017-06-23/2017-06-23-image041.png)](https://3.bp.blogspot.com/-aSk3sdIZMCg/WU0FNa49EwI/AAAAAAAABw4/2MvrCemCKwg84ApZTPqLP8R9Ef9wSccZgCEwYBhgL/s1600/image041.png)
  
When done, the following events are in the event viewer:  

[![]({{ site.url }}/images/2017-06-23/2017-06-23-image043.png)](https://2.bp.blogspot.com/-CqM7dBQxA2A/WU0FNpOpXrI/AAAAAAAABw4/8zw0SfK_Mi4mEpNAjDS-nj9xWPFoCLRfgCEwYBhgL/s1600/image043.png)

[![]({{ site.url }}/images/2017-06-23/2017-06-23-image045.png)](https://3.bp.blogspot.com/-M4PePAi-Mek/WU0FN55XmOI/AAAAAAAABw8/HKkW-V88dmoZ0CeXRenB-o8AZkIn81rOQCEwYBhgL/s1600/image045.png)