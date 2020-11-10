# Wi-Fi passwords

To extract wifi PSKs:

```text
netsh wlan show profiles
netsh wlan show profile name="Profilename" key=clear
```

The following oneliner can be used to run through all profiles:

```text
foreach($profil in (netsh wlan show profiles | Select-String ':')){foreach($line in (netsh wlan show profiles name=($profil.toString().split(':')[1].trim()) key=clear)){$line+';'}};
```

