---
description: Enumeration following gaining of a foothold by exploit or other means.
---

# Initial Enumeration

## System Enumeration

### Meterpreter

Get username and sytem info

```text
> getuid
Server username: WINXP-E95CE571A1\User
> sysinfo
Computer        : WINXP-E95CE571A1
OS              : Windows XP (Build 2600, Service Pack 3).
Architecture    : x86
System Language : en_US
Meterpreter     : x86/win32
```

N.B. drop into CMD from meterpreter using "`shell`"

### CMD

System info

```text
>systeminfo
Host Name:                 XENOV
OS Name:                   Microsoft Windows 10 Enterprise
OS Version:                10.0.18363 N/A Build 18363
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
[SNIP for brevity]

>systeminfo | findstr /B /C:"OS Name" /C:"OS Version" C:"System Type"
OS Name:                   Microsoft Windows 10 Enterprise
OS Version:                10.0.18363 N/A Build 18363
System Type:               x64-based PC
```

Patching

```text
>wmi qfe
Caption                                     CSName  Description      FixComments  HotFixID   InstallDate  InstalledBy          InstalledOn  Name  ServicePackInEffect  Status
http://support.microsoft.com/?kbid=4565633  XENOV   Update                        KB4565633               NT AUTHORITY\SYSTEM  7/20/2020
http://support.microsoft.com/?kbid=4517245  XENOV   Update                        KB4517245               NT AUTHORITY\SYSTEM  5/6/2020
http://support.microsoft.com/?kbid=4537759  XENOV   Security Update               KB4537759               NT AUTHORITY\SYSTEM  5/11/2020
http://support.microsoft.com/?kbid=4552152  XENOV   Security Update               KB4552152               NT AUTHORITY\SYSTEM  5/6/2020
http://support.microsoft.com/?kbid=4560959  XENOV   Security Update               KB4560959               NT AUTHORITY\SYSTEM  6/15/2020
http://support.microsoft.com/?kbid=4561600  XENOV   Security Update               KB4561600               NT AUTHORITY\SYSTEM  6/11/2020
http://support.microsoft.com/?kbid=4565554  XENOV   Security Update               KB4565554               NT AUTHORITY\SYSTEM  7/20/2020
http://support.microsoft.com/?kbid=4565483  XENOV   Update                        KB4565483               NT AUTHORITY\SYSTEM  7/20/2020
```

List drives

```text
> wmic logicaldisk get caption,description,providername
Caption  Description       ProviderName
C:       Local Fixed Disk
```

## User Enumeration

Current user and their privileges

```text
>whoami
host\administrator

>whoami /priv
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled

>whoami /groups
Group Name                                                    Type             SID          Attributes
============================================================= ================ ============ ==================================================
Everyone                                                      Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account and member of Administrators group Well-known group S-1-5-114    Group used for deny only
BUILTIN\Administrators                                        Alias            S-1-5-32-544 Group used for deny only
BUILTIN\Performance Log Users                                 Alias            S-1-5-32-559 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                                                 Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE                                      Well-known group S-1-5-4      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                                                 Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users                              Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization                                Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account                                    Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
LOCAL                                                         Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication                              Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level                        Label            S-1-16-8192
```

List users on system

```text
> net user
User accounts for \\XENOV

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
WDAGUtilityAccount
The command completed successfully.

> Net user DefualtAccount
User name                    DefaultAccount
Full Name
Comment                      A user account managed by the system.
User's comment
Country/region code          000 (System Default)
Account active               No
Account expires              Never

Password last set            ‎10/‎03/‎2020 14:43:28
Password expires             Never
Password changeable          ‎10/‎03/‎2020 14:43:28
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   Never

Logon hours allowed          All

Local Group Memberships      *System Managed Group
Global Group memberships     *None
The command completed successfully.

```

List local groups

```text
> net localgroup
-------------------------------------------------------------------------------
*__vmware__
*Access Control Assistance Operators
*Administrators
*System Managed Accounts Group
*Users
The command completed successfully.
```

## Network Enumeration

Networking details

```text
> ipconfig /all
[Lists basic networking details for the host]

> arp -a 
[Lists ARP table]

>route print
[routing information]

netstat -ano
[Lists listening ports and established connections]
```



