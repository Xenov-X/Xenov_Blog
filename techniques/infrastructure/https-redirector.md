---
description: Quick steps to set up basic HTTPS redirector
---

# HTTPS Redirector

c

c

```text
cat /etc/nginx/sites-available/default
##
#### Port 80 fwd to 443
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name _;
        return 301 https://$host$request_uri;

}

#### SSL configuration
server{
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;
        ssl_certificate /etc/letsencrypt/live/<Domain>/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/<Domain>/privkey.pem;
        ssl_protocols   TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers     HIGH:!aNULL:!MD5;

        server_name _;
        
#### If user agent isn't a match, forward to spoofed site
        location / { 
                if ($http_user_agent != 123) {
                        return 301 https://www.<Legit-Domain>.com;
                       break;
        }
#### If user agent matches, set up transparent proxy to team  server
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_pass https://<TeamServer>;   ###Team Server IP
  }
}
```

