# Nginx vs Apache

- static assets served by apache require that the server asks php for the file. nginx allows you to bypass fpm, or php for static assets never asking fpm for a txt, json or css file
- apache requests are file location first, nginx requests are uri location first.
- having uri locations allows nginx to serve not only as a web server, but a load balancer or even a mail server.

# Compiling NGINX from source 
This will install with SSL.

- View this documentation [building nginx from source](http://nginx.org/en/docs/configure.html)
- [nginx modules reference](https://nginx.org/en/docs/)
- first download the latest nginx source code from [nginx.org](http://nginx.org)
- `tar -zxvf nginx-X.XX.X.tar.gz`
- `apt install build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev`
- `./configure`
- Once you are happy with the configuration, run
- `make`
- `make install`
- Now check your nginx install exists:
- `ls -l /etc/nginx`
- `nginx -v`
- Now start nginx
- `nginx`
- `ps aux | grep nginx`

## Adding nginx as a systemd service
This will allow nginx to restart automatically.

- Add the [systemd init script](https://www.nginx.com/resources/wiki/start/topics/examples/systemd/)
- Keep in mind on ubuntu 20 that systemd services are located in `/etc/systemd/system`
- Modify the configuration as you see fit.
- `systemctl start nginx`
- `systemctl status nginx`
- To enable startup on boot `sytemctl enable nginx`

### Standard web server with PCRE for regex, SSL,  and using standard log paths. Has a custom PID file
```bash
./configure --sbin-path=/usr/bin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.long --with-pcre --pid-path=/var/run/nginx.pid --with-http_ssl_module
```

# Configuring NGINX
- the main configuration file must have an events and an http stanza
- the server name can be a domain name with a wildcard or an ip address
- the server root specifies where the server name will search for files from. By default this is roots home directory
- `nginx -t` to check nginx configuration
- `systemctl reload nginx` to restart gracefully
- to get CSS, JS, and HTML To work, you must include the mime types in a `types` stanza.
- by default you can include the `/etc/nginx/mime.types` that will include most of the basic mime types


# Location blocks
- `location /greet` prefix match will match any urls starting with the path
- `location =/greet` exact match - match the path to be exactly this
- `location ~ /greet[0-9]` regex match = will only match /greet1, greet2, will not match /greet. This is case sensitive
- `location *~ /greet[0-9]` same as above, but case insensitive
- regex location blocks take preference over a prefix match
- `location ^ /greet`  preferential prefix match takes preference over the regex - it is meant as being more important.
- order of preference is
  - exact match `=`
  - preferential prefix `^~`
  - regex match `~*`
  - prefix match `/`

# NGINX variables
- default variables can be seen [here](http://nginx.org/en/docs/varindex.html)
- the `$arg` variable returns query params.
- each query param is accessible like so: `?foo=bar` == `$arg_foo`  
- do not abuse this [if is evil](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/)

# Redirects
- return statements take a status code an a response string
- rewrite statements take a status code and redirect to a uri 
- the below example will take requests from /user and return the location from /greet
- if the /user/john is passed we will say a special greeting to him
```
http {
  server {
    rewrite ^/user/(\w+) /greet/$1 last;
    location /greet {
      return 200 "Hello stranger";
    }
    location /greet/john {
      return 20- "Hello John";
    }
  }
}
```

# Try Files
- in the below example nginx will check if `/sites/demo/thumb.png` exists, if it does, it will serve that file and only that file.

```
http {
  server {
    listen 80;

    root /sites/demo;

    try_files thumb.png /greet;

    location /greet {
      return 200 "Hello";
    }
  }
}

```

- A usage example of this would be to serve the main app file for a web app that intercepts requests to render content.
- if the file does not exist, the nginx will continue routing to `/greet`
- try files is typically used with request variables
- You can pass multiple arguments to try files, only the last argument is the rewrite
## named locations
- similar to try files, you can use an `@` to set a location as your routing.

```

try_files $uri @friendly_404;

location @friendly_404 {
  return 404 "404 not found"
}
```

# Logging 
- logging is enabled by default
- log locations are `/var/log/access.log`,`/var/log/error.log`
- you can define custom error and access logs with the `error_log` and `access_log` directives, this will also work inside of a location block so you can have custom logs for certain routes.
- you can disable access logging for specific locations by setting `access_log off`. This is particularly useful for huge servers. I would not recommend this as it prevents the ability to search the site for malicious attacks.

# Inheritance & Directive Types
- The three nginx directive types are the Standard directive, the array directive and the action directive

```
# Array Directives
# Directives likes this can be specified multiple times without overriding a previous setting.
# These directives get inherited by all child contexts. 
# Child context can also override array contexts
access_log /var/log/nginx-access-custom.log
access_log /var-log/overridden-nginx-access-custom.log

server {
  
  # Standard Directive
  # Can only be declared once. A second declaration overrides the first
  # Gets inherited by all child contexts
  # Child context can override inheritance by redeclaring directive
  root /sites/blog;

  location /secret {
    # Action Directive
    # invokes an action such as a rewrite or a redirect
    # Inheritance does not apply as the request is either stopped (redirect/respose) ore reevaluated (rewrite)
  }
}
```

# PHP Processing
- to host a php site we use `php-fpm` which will host the php app on a port, the nginx will act as a reverse proxy, typically serving on port 80
- In the example below we will serve the php app through the fastcgi proxy.
- The index.php is the main file for the app, otherwise we serve index.html
- Any request to a php file is given to fastcgi via fastcgi.conf
- the `fastcgi_pass` tells nginx to pass the request to a unix socket
- You must also allow nginx to run as the `www-data` user so that php requests are served correctly.
```
$ ps aux | grep php
root       21364  0.0  1.8 195476 18764 ?        Ss   17:11   0:00 php-fpm: master process (/etc/php/7.4/fpm/php-fpm.conf)
www-data   21365  0.0  0.6 195856  6536 ?        S    17:11   0:00 php-fpm: pool www
www-data   21366  0.0  1.2 195856 12152 ?        S    17:11   0:00 php-fpm: pool www
root       21635  0.0  0.3   8616  3536 pts/0    S+   17:41   0:00 /bin/bash -c  ps aux | grep php-fpm
root       21637  0.0  0.0   8120   736 pts/0    S+   17:41   0:00 grep php-fpm
```
Here is the nginx configuration

```

events {}
user www-data;

http {
  include mime.types;
  server {
    listen 80; 
    server_name some-host;

    root /sites/blog;
    # By default serve index.php, if it does not exist serve index.html
    index index.php index.html;
    
    location / { 
      # given the uri, try serving it otherwise it is a 404
      try_files $uri $uri/ =404;
    }   

    # All php files are served via fpm
    location ~\.php$ {
      # Fast CGI is the proxy
      include fastcgi.conf;
      # This tells nginx to use the unix socket where fpm is running
      fastcgi_pass unix:/run/php/php7.4-fpm.sock;

    }   
  }
}
```
# Worker Processes
Notice that there is a master process and a worker process below

```
root@host:/sites/blog# systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2020-12-31 15:01:30 UTC; 2h 51min ago
       Docs: man:nginx(8)
    Process: 21647 ExecReload=/usr/sbin/nginx -g daemon on; master_process on; -s reload (code=exited, status=0/SUCCESS)
   Main PID: 2448 (nginx)
      Tasks: 2 (limit: 1137)
     Memory: 5.2M
     CGroup: /system.slice/nginx.service
             ├─ 2448 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             └─21648 nginx: worker process
```

The master process is `nginx` itself. `nginx` will spawn workers which will listen to client requests. The default amount of workers for an nginx is 1.


To change the number of workers, you can specify it in the nginx.conf.

```
worker_processes 2;
```

Note that increasing the amount of workers does not increase performance. Workers handle requests as fast as the hardware and code can.

However when we have more than 1 cpu core, they cannot share processes. A nginx worker can not run on more than 1 core. So with that being said, if you have an 8 core server - set the workers to 8.

You can also se the

```
worker_processes auto;
```

To allow the workers to automatically scale per core of the machine.


### Worker connections

Given this example

```
events {
  worker_connections 1024;
}
```

We are specifying the amount of files that the nginx server can be open at one time.

You can check this amount by running `ulimit -n` Given that my server returns `1024` I will set the number to this.

### Max Connections

You can determine the maximum amount of connections a server can handle by multiplying the worker processes by the worker connections.


# Buffer Sizes
Buffering is when a process read data into memory. If the buffer is too large some of it is written to disk (swap)

A timeout will prevent a client from sending an endless stream of data and breaking the server.

```
# Buffer size for POST submissions
client_body_buffer_size 10K;
client_max_body_size 8m;

# Buffer size for headers
client_header_buffer_size 1k;

# Max time to receive client headers/body
client_body_timeout 12;
client_header_timeout 12;

# Max time to keep a connection open for
keepalive_timeout 15;

# Max time for the client accept/receive a response
send_timeout 10;

# Skip buffering for static files
sendfile on;

# Optimize sendfile packets
tcp_nopush on;
```

# Dynamic modules
- run `nginx -V` to see the existing configuration
- Show existing dynamic modules with `./configure --help | grep dynamic`
- Recompile nginx
- Run `make`
- Run `make install`
- Run `systemctl reload nginx`
- Run `systemctl status nginx`


# Performance
The below code will tell the client browser to cache any css, jpg, js, or png for 1 hour after being requested.
```
    location ~* \.(css|js|jpg|png) {
        access_log off;
       add_header Cache-Control public;
        add_header Pragma public;
        add_header Vary Accept-Encoding;
        expires 60m;
      image_filter rotate 180;
    }
```

# Compressing responses with gzip

Enable gzip compressiong with the directive 

- A compression level above 5 is basically useless. 3 or 4 are good for most projects.

```
gzip on;
gzip_comp_level 3;
gzip_types text/css text/javascript;
```

Note that you must return the header to your location. This occurs by default on my nginx configuration, but you should check your headers for `Vary: Accept-Encoding`

```
add_header Vary Accept=Encoding
```

# Caching

You can specify the format of a cache like so
```
http {
  include mime.types;

  fastcgi_cache_path /tmp/nginx_cache levels=1:2 keys_zone=MYCACHE:100m inactive=60m;
  fastcgi_cache_key "$scheme$request_method$host$request_uri";
```
This will create MD5 hashed cache keys consisting of the request URL with the host.


You can return a header on whether or not it was a cache hit with 

```
add_header X-Cache $upstream_cache_status;
```

### Tracking cache exceptions
The below example will skip the fastcgi cache whenever a `skipcache=1` argument is passed to the server.

```
set $no_cache 0;

if ($arg_skipcache = 1) {
  set $noCache 1;
}

location ~\.php {
  fastcgi_cache_bypass $no_cache;
  fastcgi_no_cache $no_cache;
}
```


# HTTP2

To enable, compile nginx with the `--with-http_v2_module` parameter.

- http2 is a binary protocol vs htt1 being a textual protocol
- http compresses headers
- http2 uses persistent, multiplexed connections
- http2 allows for server push events on the client side

to enable http2

```
server {
  listen 443 ssl http2;
  ssl_certificate /some/certificate/path;
  ssl_certificate_key /some/certificate/key
}
```

# Server Push

Server push allows you to return multiple files with a single response.

You can test server push with this command.
```
nghttp -nysa somehost
```

Without the Configuration this is what you would see when running the command.
```
sorted by 'complete'

id  responseEnd requestStart  process code size request path
 13   +166.23ms        +81us 166.14ms  200  11K /

```


Now given this configuration.

```

location / {
 http2_push /style.css;
 http2_push /thumb.png;
}
```

```

id  responseEnd requestStart  process code size request path
 13      +398us        +57us    341us  200   4K /
  2      +518us *     +279us    238us  200  980 /style.css
  4     +2.05ms *     +295us   1.75ms  200  12K /thumb.png

```

# SSL Notes

- You should always redirect https to http. This can be acheived like so.

```

http {
  server {
    listen 80;
    server_name localhost;
    return 301 https://$host$request_uri;
  }
  server {
    listen 443;

# Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

# Optimize cipher suites
    ssl_prefer_ciphers on;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADS:!AECHD:!MD5;

# DH Params.
# To generate DH Params, run 
# openssl dhparam 2048 -out /etc/nginx/ssl/dhparam.pem
# Note that the size must match that of your SSL certificate
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;

# Enable HSTS, a header to prevent the browser from loading anything over https
    add_header Strict-Transport-Security "max-age=3153600" alwaysl

# SSL Session Cache
    ssl_session_cache shared:SSL:40m;
    ssl_session_timeout 4h;
    ssl_session_tickets on;
  }
}
```

# Rate Limiting

You can test your rate limiting with `siege`

```
http {
# Rate Limit per request (bad)
  limit_req_zone $server_name zone=RATELIMIT:10m rate=1/s;

# Rate limit per user IP address per user. Good for preventing brute force.
  limit_req_zone $binary_remote_addr zone=RATELIMIT:10m rate=1/s;

# Rate limit for the specific request
  limit_req_zone $request_uri zone=RATELIMIT:10m rate=1/s;

# You do not need to include both server and location blocks, I am simply showing that they both work.
  server {
    limit_req zone=RATELIMIT;
  }

  location {
    limit_req zone=RATELIMIT;
  }
}
```
### Burst rates

You are also able to set a burst rate on either a `limit_req_zone` or a `limit_req` directive.

This will alow the number of additional requests, but they are not preffered - or slowed down.

```
limit_req_zone $binary_remote_addr zone=MYZONE:10m rate=1/s burst=5;
```
Would allow 6 connections total, but the remaining 5 after hitting rate limit would be "queued" perse.

### Nodelay

Nodelay allows bursted requests to be served as quickly as possible.

```
limit_req_zone $binary_remote_addr zone=MYZONE:10m rate=1/s burst=5 nodelay;
```


# hardening nginx
- you should keep server packages up to date

```
http {
# disable nginx server tokens displaying what version number you are using
  server_tokens off;

  server {
# disallow "clickjacking"
  add_header X-Frame-Options "SAMEORIGIN";
  }
}
```

It should be added that nginx should be compiled without necessary modules, such as 

```
--without-http_autoindex_module
```

# Reverse Proxy

```
http {
  server {
    listen 8888:

    location / {
      return 200 "hello from nginx";
    }

    location /somepath {
# To serve some project that is on port 9999
      proxy_pass 'http://localhost:9999';
    }
    location /nginxorg {
# All requests to /nginxorg go to nginx.org
      proxy_pass 'http://nginx.org';
    }
# another use of a reverse proxy is to add headers to a request
    location /somepath2 {
# send the header to the proxy
      set_header foo yes;
# Send the header to client from proxy
      proxy_set_header proxied nginx;
      proxy_pass 'http://localhost:9999/';
    }
  }
}
```


# Load Balancer
```
http {
  upstream server_block {
# Enable sticky session - keep a users ip on the same server.
    ip_hash;

# As opposed to ip_hash, this setting would direct traffic to the lead connected.
    least_conn;
    server someservice:1001;
    server someservice:1002;
    server someservice:1003;
  }

  server {
    listen 80;
    location / {
      proxy_pass http://server_block;

    }
  }

}
```
# GEOIP
- [geoip documentation](https://github.com/leev/ngx_http_geoip2_module)
First you must enable the module `--with-http_geoip_module`
Now download the maxmind geoip database and install it following the documentation linked above.
- Once the files are extracted

```
http {
  geoip_county /etc/nginx/geoip/GeoIP.dat;
  geoip_city /etc/nginx/geoip/GeoLiteCity.dat;
  
  location /geo_country {
    return 200 "visiting from $geo_country_name";
  }
}

```
# Video Streaming

```
http {
  server {
    location ~ \.mp4$ {
      root /sites/downloads/;
      mp4;
      mp4_buffer_size 4M;
      mp4_max_buffer_size 10M;
    }
  }
}

```
