---
description: >-
  Identification of a targets internal domain name using public facing web
  applications that support NTLM SSP authentication, such as Exchange.
---

# Internal domain enumeration

## NTLM SSP challenge response decoding for internal domain

Knowing the internal domain name of a target can be highly useful for a range of activities including finding similar public domain names, designing phishing lures or keying malware to ensure it only runs in the target environment. 

### Web applications

Web applications that support NTLM authentication can be leveraged to access this information very easily. Fortunately, a very common web application supports NTLM authentication, Exchange servers. 

Two URLs that Exchange servers support for NTLM authentication are shown below. This authentication type is not exclusive to exchange, and this internal domain infromation disclosure is possible on any applicaitions that support it. 

```text
https://[host]/EWS
https://autodiscover.[host]/autodiscover/autodiscover.xml 
```

By intiating an NTLM authentication request with null credentials the server will respond with an NTLM challenge response. This is encoded with a known key and can be decoded enumerate NetBIOS name, DNS, and OS build version information.

In many cases web applications that do not offer NTLM SSO authentication during normal usage will respond with a NTLM challenge response if a GET request is made including the Authorization header with null credential values.

```http
Authorization: NTLM TlRMTVNTUAABAAAAB4IIAAAAAAAAAAAAAAAAAAAAAAA=
```

This can be automated using the nmap script http-ntlm-info.nse

```http
nmap -p 443 --script http-ntlm-info [host]
```

### Other services

NTLM SSO is supported by a wide array of other services.

The nmap script discussed above \(http-ntlm-info.nse\) has alternatives for a number of other protocols.

These include SMTP, IMAP, POP3, MS-SQL, TELNET, NNTP, and RDP \(eg rdp-ntlm-info.nse\). To use all of these scripts:

```http
nmap --script=*-ntlm-info --script-timeout=60s [host]
```

## Tools:

### Nmap script: [http-ntlm-info.nse](https://nmap.org/nsedoc/scripts/http-ntlm-info.html)

This is included the nmap default script repository.

```bash
nmap -p 443 --script http-ntlm-info [host]

PORT    STATE SERVICE
443/tcp open  https
| http-ntlm-info:
|   Target_Name: DOMAIN
|   NetBIOS_Domain_Name:  DOMAIN
|   NetBIOS_Computer_Name: Hostname
|   DNS_Domain_Name: Domain.Parent_Domainl
|   DNS_Computer_Name: Hostname.Domain.Parent_Domain
|   DNS_Tree_Name: Parent_Domain
|_  Product_Version: 10.0.14393

```

By default the script includes the Authorization header with null credentials in its request to the web server; where the NTLM authentication is not accessable at the root of the web applciation, specify the path using the script argument "http-ntlm-info.root".  

Examples for the two Exchange server paths above:

```text
nmap -p 443 --script http-ntlm-info --script-args http-ntlm-info.root=/EWS [host]
nmap -p 443 --script http-ntlm-info --script-args http-ntlm-info.root=/autodiscover/autodiscover.xml autodiscover.[host]
```

See also:

* smpt-ntlm-info.nse
* imap-ntlm-info.nse
* pop3-ntlm-info.nse
* ms-sql-ntlm-info.nse
* telnet-ntlm-info.nse
* nntp-ntlm-info.nse
* rdp-ntlm-info.nse

### Burp plugin: [**ntlm-challenge-decoder**](https://github.com/PortSwigger/ntlm-challenge-decoder)

The burp suite extension ‘NTLM Challenge Decoder’, is also available. This adds an additional tab when you view HTTP responses that include NTLM challenge responses.

It will also convert Windows version number into their standard product names.

![](../../.gitbook/assets/image%20%282%29.png)

Sample HTTP request & response

```http
Sample request:
GET /ews HTTP/1.1
Host: ***.***.***.***
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Authorization: NTLM TlRMTVNTUAABAAAAB4IIogAAAAAAAAAAAAAAAAAAAAAKANc6AAAADw==
 
Sample response:
HTTP/1.1 401 Unauthorized
Server: Microsoft-IIS/7.5
WWW-Authenticate: NTLM [NTLM Response]
WWW-Authenticate: Negotiate
WWW-Authenticate: Basic realm="***.***.***.***"
X-Powered-By: ASP.NET
Date: Wed, 14 Aug 2019 12:29:13 GMT
Connection: close

Content-Length: 0
```

## Resources

* [https://github.com/PortSwigger/ntlm-challenge-decoder](https://github.com/PortSwigger/ntlm-challenge-decoder)
* [https://nmap.org/nsedoc/scripts/http-ntlm-info.html](https://nmap.org/nsedoc/scripts/http-ntlm-info.html)

