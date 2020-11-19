---
description: >-
  Windows clipboard can be used for interaction with password managers & general
  sensitive infromation
---

# Clipboard

Options to read the clipboard:

### Powershell - Get-Clipboard

```text
echo Get-Process | clip
Get-Clipboard | iex
```

### SharpClipboard

Register in Clipboard chain for notification of clipboard changes - allowing for reading. SharpClipboard supports running in CobaltStrike's execute-assembly. Kill with job kill when done. 



