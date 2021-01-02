# Below is my configuration and setup for a docker swarm host.


## Server Info

* Ubuntu 20
* NGINX 1.19
* LetsEncrypt
* Gitlab CI/CD Deploys




# Setup
```
mkdir ~/install_files

```
## UFW

Basic UFW configuration guide can be found [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-20-04)

## Swarm Worker

I have a Gitlab group configuration as the runner, so follow the steps in `Groups -> <your-group> -> Settings -> CI/CD -> Group Runners`.

## NGINX

Nginx configuration is a good default setup with autoindexing disabled, http2 and SSL support. Make sure you append the existing configuration output seen from `nginx -V`!

You can clear the nginx cache by running

```
sudo clear_nginx_cache
```

```
--with-cc-opt='-g -O2 -fdebug-prefix-map=/build/nginx-5J5hor/nginx-1.18.0=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-compat --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-mail=dynamic --with-mail_ssl_module
```
```
cd /home/root/install_files/nginx-xxx
nginx -V
./configure <existing-configuration-output>
make
make install
nginx -t
systemctl reload nginx
```

NGINX is installed as a systemd service. It is enabled on restart.

```
/etc/systemd/system/nginx.service
```

## SSL

Use Lets Encrypt via certbot. This adds nginx_ensite which is not ideal, but I will tend to keep vhost configurations in the `sites_enabled` and them add the host in the  main `/etc/nginx/nginx.conf`

Before adding SSL Certificates, make sure there is an `A` DNS record for @ and www, as well as any subdomains.

### Installing Certbot

```
sudo apt install certbot python3-certbot-nginx
```

### Updating certificates, Creating Certificates

```
sudo certbot --nginx -d somedomain.com -d someotherdomain.com
```

### Check certbot autorenew status

```
sudo systemctl status certbot.timer
sudo certbot renew --dry-run
```

