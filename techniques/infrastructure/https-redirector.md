---
description: Quick steps to set up basic HTTPS redirector
---

# HTTPS Redirector

Example basic HTTP\(S\) redirector. Ideally add additional filtering for specific URIs etc.

```text
cat /etc/nginx/sites-available/default
##
#### Port 80 fwd to 443
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name _;
        
        #Allow local web server to handle requests made to LetsEncrypt files. 
        #This will esure LetsEncrypt/Certbot can update TLS certificated, and requests won't be forwarded to proxied sites. 
       
        location ^~ /.well-known/acme-challenge/ {
        default_type "text/plain";
        #root         /var/www/letsencrypt; #If you use Certbot's "nginx" option, this doesnt need to be added, as certbot will place files in your web root. 
        }

        #Return 404 if users try to access acme-challenge folder
        location = /.well-known/acme-challenge/ {
        return 404;
        }
        
        
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

