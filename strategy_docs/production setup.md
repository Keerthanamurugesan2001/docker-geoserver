# GeoServer Deployment with Docker, Nginx, and Cerbot

This document outlines the process of deploying the Kartoza GeoServer Docker container with Nginx on a bare metal server.
It also addresses common issues encountered with login redirects over HTTPS and provides solutions for `j_spring_security_check` and CSRF issues.

## Prerequisites

- [Docker](https://docs.docker.com/engine/install/ubuntu/) & Docker Compose installed on your server.
- [Nginx](https://docs.vultr.com/how-to-install-nginx-web-server-on-ubuntu-24-04?ref=9141994&utm_source=performance-max-apac&utm_medium=paidmedia&obility_id=16876059738&&utm_campaign=APAC_-_India_-_Performance_Max_-_1001&utm_term=&utm_content=&ref=9141994&gad_source=1&gclid=Cj0KCQiA0--6BhCBARIsADYqyL8qSDD8yKeGT1Nxk3vMHSp7t2meQCy6NYQyOlZwluYTaDS0EzVH-DEaAse1EALw_wcB) installed locally on the server for reverse proxy.
- [Git]
- Certbot.

## Step-by-Step Setup


### Step 1:
clone your geoserver docker, please fork the `https://github.com/kartoza/docker-geoserver`
and then clone your forked repo
Ex:

```
git clone https://github.com/manikandanmohan10/docker-geoserver.git
git checkout dev

```


### Step 2. change docker-compose.yml

Change `docker-compose.yml` file according to the below file, It is required for production setup. If you are in `dev` branch this correction not required leave it.

[click here](https://github.com/manikandanmohan10/docker-geoserver/blob/develop/docker-compose.yml)


### Step 3. Run  the docker cmd

```
docker-compose up -d
```


### setp 3. Nginx Configuration

**Set Up NGINX:**

Install NGINX locally on your machine or server.
Place the nginx.conf in the** /etc/nginx/conf.d/** directory or as specified in your local configuration.
Restart NGINX to apply changes: `sudo systemctl restart nginx`.

```
sudo nano /etc/nginx/conf.d/geoserver.conf
```


```

server {
    listen 443 ssl; # managed by Certbot
    server_name geoserver.groupstrategy7.com;  # Your actual domain

    ssl_certificate /etc/letsencrypt/live/geoserver.groupstrategy7.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/geoserver.groupstrategy7.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    location / {
        proxy_pass http://127.0.0.1:8000;  # Adjust the backend service address
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;  # Important for HTTPS
        client_max_body_size 250M;
    }

    location /geoserver {
        proxy_redirect http://127.0.0.1:8000/geoserver https://$host/geoserver;

        proxy_pass http://127.0.0.1:8000/geoserver;  # Adjust the backend service address
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;  # Important for HTTPS
        proxy_set_header X-Script-Name /geoserver;  # Important for GeoServer to recognize the context
        client_max_body_size 250M;
}

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
}

server {
    listen 80;
    server_name geoserver.groupstrategy7.com;
    return 301 https://$host$request_uri; # Redirect to HTTPS
}
```

**Start NGINX**

```
sudo systemctl start nginx
```

**Enable NGINX to Start on Boot**

```
sudo systemctl enable nginx
```

**see status**

```
sudo systemctl status nginx
```

### 4. Fixing the Login Redirect Issue (j_spring_security_check)
When using HTTPS, some users may face an issue where the login page redirects from HTTPS to HTTP, causing login failures. This issue is related to the j_spring_security_check handler in GeoServer.

Solution: Ensure the following is set in your Nginx configuration to preserve the protocol:

nginx
```
proxy_set_header X-Forwarded-Proto https;
```

This ensures that GeoServer knows it is running behind a proxy and doesn't redirect back to HTTP.

Reference Solution:
Faced issue on redirect https to https while login
[Reddit discussion on login redirect issue](https://www.reddit.com/r/javahelp/comments/1fz25xh/geoserver_j_spring_security_check_on_login_keep/)


[docker container](https://hub.docker.com/r/kartoza/geoserver)
[csrf](https://docs.geoserver.org/stable/en/user/security/webadmin/csrf.html)
[proxy](https://stackoverflow.com/questions/68783126/issue-with-geoserver-login-with-ssl)

