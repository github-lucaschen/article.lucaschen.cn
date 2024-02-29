---
author: Lucas Chen
title: 'Nginx 基础(Nginx Basic)'
date: 2023-01-04
description: >-
  How to use nginx easily.
tags:
  - server
  - nginx
categories:
  - server
  - en-US
series:
  - nginx
---

How to use nginx easily.

The following sections are included:

1. Install
2. Configuration
3. Virtual host
4. Reverse proxy
5. Load balancing
6. Dynamic-Static Separation
7. UrlRewrite

<!--more-->

## Install Nginx

```shell
# https://nginx.org/download/
./configure
make && make install
```

### Start the nginx

```shell
cd /usr/local/nginx/sbin
# start nginx
./nginx
# quick stop
./nginx -s stop
# complete accepted connection requests before exiting
./nginx -s quit
# reload the configuration file
./nginx -s reload
```

### Installation serves the system

#### Step 1: Creating a service script

```shell
vi /usr/lib/systemd/system/nginx.service
```

#### Step 2: The content of the script

```txt
[Unit]
Description=nginx - web server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
ExecQuit=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

#### Step 3: Reload the system service

```shell
systemctl daemon-reload
systemctl start nginx
systemctl status nginx
```

#### Step 4: Set to boot upon startup

```shell
systemctl enable nginx.service
```

## Basic configuration files for nginx

```txt
# The default value is 1, indicating that a service process is started
# Configure how many worker_processes be created
# The number of cores for each CPU should be one
# e.g., 1c = 1, 2c = 2
worker_processes  1;

events {
    # Number of connections acceptable for a single service process
    worker_connections  1024;
}

http {
    # Reference to other files
    # mime.types: Identifies the file type of the HTTP header
    include       mime.types;
    # Set the default file type
    # application/octet-stream: Stream, will download
    default_type  application/octet-stream;

    # Efficient network transfer using Linux  sendfile(socket, file, len)
    # i.e., zero copy of data
    sendfile        on;

    keepalive_timeout  65;

    # Virtual host
    server {
        # The listening port number
        listen       80;

        # Domain name or host name
        server_name  localhost;

        # Matching URI paths
        location / {
            # File root
            # Relative path, relative to the nginx home directory
            root   html;
            # Default page name
            index  index.html index.htm;
        }

        # Error code corresponding page
        error_page   500 502 503 504  /50x.html;


        location = /50x.html {
            root   html;
        }
    }
}
```

## Virtual host

Originally a server can only correspond to a site, through the
virtual host technology can be virtualized into multiple sites to provide services externally at the same time

### Servername Matching rule

Servername matches are in the order in which they are written. The matches that are written in front of it don't go any further.

#### Rule 1: Full match

We can match multiple domain names in the same servername.
Separate domain names with Spaces.

```shell
server_name vod.lucaschen.cn www.lucaschen.cn;
```

#### Rule 2: Wildcard match

```shell
server_name *.lucaschen.com
```

#### Rule 3: Wildcard ends match

```shell
server_name vod.*;
```

#### Rule 4: Regular match

```shell
server_name ~^[0-9]+\.lucaschen\.cn$;
```

## Reverse proxy

```shell
location / {
-    root   html;
-    index  index.html index.htm;
+    proxy_pass http://lucaschen.cn/;
}
```

You have to choose between `root` and `proxy_pass`,You can't have both.

## Load balancing

```shell
upstream httpd {
    server 192.168.100.201:80;
    server 192.168.100.202:80;
}

server {
    listen       80;
    server_name  localhost;

    location / {
        # Same name as defined by upstream
        proxy_pass http://httpd;
    }
}
```

### Strategy

#### Strategy 1: polling(default)

By default, polling is used, forwarding one by one, which is suitable for stateless requests.

#### Strategy 2: weight

Specifies the polling probability, weight proportional to the access ratio, for cases where back-end server performance is uneven.

```shell
upstream httpd {
    server 192.168.100.201:80 weight=10 down;
    server 192.168.100.202:80 weight=1;
    server 192.168.100.203:80 weight=1 backup;
}
```

- down: Indicates that the current server does not participate in the load
- weight: The default value is 1. The greater the weight, the greater the weight of the load
- backup: If all other non-backup machines are down or busy, request the backup machine

#### Strategy 3: ip_hash

The IP address of the client is forwarded to the same server, you can keep calling back.

#### Strategy 4: least_conn

Minimum connection access.

#### Strategy 5: url_hash

Forwards requests based on the url accessed by the user.

#### Strategy 6: fair

Forward the request based on the backend server response time.

## Dynamic-Static Separation

```shell
location / {
    proxy_pass http://lucaschen.cn/;
}

location /css {
    root /usr/local/nginx/static;
}

location /images {
    root /usr/local/nginx/static;
}

location /js {
    root /usr/local/nginx/static;
}
```

### Location prefix

- `/`: Universal match, any request will be matched
- `=`: Accurate matching, not starting with a specified pattern
- `~`: Regular matching, case sensitive
- `~*`: Regular matching, case insensitive
- `^~`: Non-regular matches that match a location that begins with a specified pattern

### Location matching order

- Multiple regular locations are matched directly in written order, and no further matches are made after success
- A normal(non-regular) location will go down until it finds the best match(maximum prefix match)
- When both location(non-regular) and regular location exist, if the regular match is successful, location(no-regular) match is performed
- If all types of location exist, `=` > `^~` > Regular Match > Normal(Maximum prefix match)

### Using regular expressions

```shell
location ~*/(css|img|js) {
    root /usr/local/nginx/static;
}
```

### alias and root

```shell
location /css {
    alias /usr/local/nginx/static/css;
}
```

Root sets the root directory, and alias accepts requests without a location on the path.

1. alias specifies an exact directory. That is, the files in the path directory accessed by location are directly found in the alias directory
2. The directory specified by root is the upper level of the path directory matched by location. The path directory must exist in the root directory
3. rewrite's break cannot be used in directory blocks that use the alias tag (for unknown reasons), alias must be followed by a "/"
4. In the alias virtual directory configuration, if the path directory matched by location does not contain a slash (/), the access is unaffected by whether the path directory is followed by a slash (/). When accessing the URL, the url automatically adds a slash (/). However, if location matches the path directory followed by a "/", then the path directory must be followed by a "/" in the url accessed. It will not automatically be followed by a "/". If you don't add a "/", the access will fail
5. In the root directory configuration, whether the path directory matching location is followed by a slash (/) or not does not affect access

## UrlRewrite

```shell
rewrite <regex> <replacement> [flag];
e.g.
rewrite ^/([0-9]+).html$ /index.jsp?pageNum=$1 break;
```

Rewrite is the key directive to implement URL rewriting, redirecting to replacement according to the regex (regular expression) part and ending with flag.

- rewrite: The keyword error_log cannot be changed
- regex: Perl is compatible with regular expression statements for rule matching
- replacement: Replace the content of the regex match with replacement
- flag: Rewrite supports flag flags

Those tag segment positions can exist with the rewrite argument:

- server
- location
- if

Description of the flags:

- last: After the matching of this rule is complete, new location URI rules are further matched. The last matched rule takes effect
- break: This rule terminates when the match is complete and does not match any subsequent rules
- redirect: Return 302 temporary redirect, browser address bar will display the URL after jump
- permanent: Returns the 301 permanent redirect, browser address bar will display the URL after jump
