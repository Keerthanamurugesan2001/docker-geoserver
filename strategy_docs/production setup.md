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

