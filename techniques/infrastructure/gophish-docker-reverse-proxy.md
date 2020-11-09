# Gophish Docker reverse proxy

```text
#Declaring Public GoPhish Web Server
upstream GoPhishPublic  {
     server gophish:443; 
    }
#Declaring GoPhish Management location
upstream GoPhishMgmt {
     server gophish:3333;
     
     

# Forwards all traffic to port 80 to 443 (HTTP > HTTPS)
server {												
            listen 80 default_server;
            listen [::]:80 default_server;
            server_name _;
            
    location /.well-known/acme-challenge/ {
        proxy_pass http://certbot
    }

    location / {
        return 301 https://$host$request_uri;
    }

}

server{
            listen 443 ssl default_server;
            listen [::]:443 ssl default_server;
            ssl_certificate /etc/letsencrypt/live/[[DOMAIN_NAME]]/fullchain.pem;	
            ssl_certificate_key /etc/letsencrypt/live/[[DOMAIN_NAME]]/privkey.pem;
            ssl_protocols   TLSv1 TLSv1.1 TLSv1.2;
            ssl_ciphers     HIGH:!aNULL:!MD5;

            server_name _;
 
    }
 
        ## config ##
        location / {
                proxy_set_header        Accept-Encoding   "";
                proxy_set_header        Host              $http_host;
                proxy_set_header        X-Forwarded-By    $server_addr:$server_port;
                proxy_set_header        X-Forwarded-For   $remote_addr;
                proxy_set_header        X-Forwarded-Proto $scheme;
                proxy_set_header        X-Real-IP         $remote_addr;

#Send all to GoPhish public web server with proxy headers including originating IP
                proxy_pass  https://GoPhishPublic;
               
# Reverse proxy Lab traffic to GoPhish Admin insterface
                if ( $remote_addr ~* [YOUR IP] ) {
                        proxy_pass https://GoPhishMgmt;
                }
                proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        }
        }
 
```

