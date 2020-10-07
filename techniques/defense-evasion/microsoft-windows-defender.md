# Microsoft Windows Defender

Check if AV running:

```bash
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List | more
Get-MpComputerStatus 
```

Disable Defender

```text
sc config WinDefend start= disabled
sc stop WinDefend
# Powershell
Set-MpPreference -DisableRealtimeMonitoring $true
# Remove definitions
"%Program Files%\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All
```

