# VBA

## Auto run macros

Auto execute - Word

```text
Sub AutoOpen()
Run
End Sub
```

Auto execute - Excel

```text
Sub Auto_Open()
Run
End Sub
```

## Bypass keying for testing

```text
set USERDNSDOMAIN=<the_target_domain_name> & macrodoc.xslm
```

