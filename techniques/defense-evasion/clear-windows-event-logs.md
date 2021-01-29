---
description: not so subtle...
---

# Clear windows event logs

The event logs can be cleared with the following utility commands:

### Wevtutil

```text
Wevtutil cl system
Wevtutil cl application
Wevtutil cl security
```

### Powershell

```text
$logs = Get-EventLog -List | ForEach-Object {$_.Log}
$logs | ForEach-Object {Clear-EventLog -LogName $_ }
Get-EventLog -list
```



