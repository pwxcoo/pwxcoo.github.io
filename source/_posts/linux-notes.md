---
title: linux notes
date: 2018-12-28 16:30:58
categories: memo
tags:
- memo
- linux
---

a collection of frequently-used linux operations.

## server configure ssh service

- `useradd -m [username]`, `-m` will automatically create new user folder in `/home`
- `passwd [username]`, configure password
- `usermod -s /bin/bash [username]`, change default login shell
    - `cat /etc/passwd` list all users
    - `cat /etc/shells` list all available shells
- `usermod -aG sudo pwxcoo`，grant `sudo` privileges
    - `groups` check group of current user
- in client，`scp id_rsa.pub [username]@[server address]:/home/[username]` transfer public key to server
- back to server，`mv id_rsa.pub .ssh/authorized_keys`  will be ok

## nginx

- `sudo apt update`
- `sudo apt install nginx`
- `cd /etc/nginx/conf.d` (NGINX site-specific configuration files are kept in `/etc/nginx/conf.d`)
- simple template. `example.com.conf`
    ```conf
    server {
        listen       80;
        server_name  example.com;

        #charset koi8-r;
        #access_log  /var/log/nginx/host.access.log  main;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
    ```
- test nginx configration is ok
    - `sudo nginx -t`
    - `sudo nginx -s reload`