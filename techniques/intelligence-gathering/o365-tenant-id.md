# O365 Tenant ID

### Enumerate tenant ID from domain name

```
curl https://login.microsoftonline.com/[domain]/.well-known/openid-configuration
```

### Tenant ID to domain name

Get part of tenant domain name - doesn't recover full domain

```
https://login.microsoftonline.com/[tenantID]/oauth2/devicecode?client_id=x
```



### Determine if a machine is joined to AzureAd

`HKLM:/SYSTEM/CurrentControlSet/Control/CloudDomainJoin/JoinInfo/{Guid}`

Underneath the key, the following keys can be found:\
– TenantId\
– UserEmail



### Determine if a user is joined to an AzureAd Domain

`HKCU:/SOFTWARE/Microsoft/Windows NT/CurrentVersion/WorkplaceJoin/AADNGC/{Guid}`

Underneath the key, the following keys can be found:\
– TenantDomain\
– UserId
