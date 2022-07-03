---
description: LOLBINs that can be used to execute Windows executables.
---

# Executables - LOLBIN Execution

### ForFiles

Privileges required: User \
OS: Windows vista, Windows 7, Windows 8, Windows 8.1, Windows 10\
Mitre:[T1218](https://attack.mitre.org/wiki/Technique/T1218)

Executes payload.exe since there is a match for notepad.exe in the c:\windows\System32 folder.

```
forfiles /p c:\windows\system32 /m notepad.exe /c C:\Tools\payload.exe
/p - path to search
/m - search mask
/c - command to run
```

Findfiles can also be used to execute ADS.\
Mitre: [T1096](https://attack.mitre.org/techniques/T1096/)

```
forfiles /p c:\windows\system32 /m notepad.exe /c "C:\Tools\notpayload.exe:payload.exe"
```

### Program Compatibility Assistant

Privileges required: User \
OS: Windows 7, Windows 8, Windows 8.1, Windows 10\
Mitre: [T1202](https://attack.mitre.org/techniques/T1202/)\
\
Used for running programs with incompatabilities with the installed version of Windows. Can be used to run a binary in a new process tree.&#x20;

```
pcalua.exe -a C:\Tools\payload.exe
```

### SyncAppvPublishingServer

Privileges required: User \
OS: Windows 10\
Mitre: [T1218](https://attack.mitre.org/techniques/T1218/)

The SyncAppvPublishingServer initiates the Microsoft application virtualization (App-V) publishing refresh operation. However it can be used as a non-directly method to execute commands for evasion. In the example below the execution occurs from PowerShell and the “Start-Process” cmdlet is used to run the executable.

```
SyncAppvPublishingServer.vbs "n; Start-Process C:\Tools\payload.exe"
```

This technicque can be used to run any powershell commands without using powershell.exe.

```
SyncAppvPublishingServer.vbs "n; <Powershell command>"
```

&#x20;
