# Below is my configuration and setup for a docker swarm host.

This guide is for docker swarm setup on a fresh ubuntu host. It consists of one manager and two node workers.

## Server Info

#### Docker cluster

For this setup, I have a $10/month droplet as my worker and swarm manager, and two $5 month droplets as nodes

# Setup

1. [Follow the guide](https://docs.docker.com/engine/install/ubuntu/) below to install docker.
2. [Enable swarm mode](https://docs.docker.com/engine/swarm/swarm-mode/), on your master node.

```
docker swarm init --advertise-addr <node-ip-address>
```

3. On worker node(s) install docker, and then join the swarm using the token from the previous step.

4. On your swarm manager, open the following ports

```
ufw allow 22/tcp
ufw allow 2376/tcp
ufw allow 2377/tcp
ufw allow 7946/tcp
ufw allow 7946/udp
ufw allow 4789/udp
```

5. Restart docker on swarm manager

```
systemctl restart docker
```

6. Open ports on each worker

```
ufw allow 22/tcp
ufw allow 2376/tcp
ufw allow 7946/tcp
ufw allow 7946/udp
ufw allow 4789/udp
```

7. Restart docker on each worker.

8. On your swarm manager, you should now be able to display your nodes

```bash
root@swarm-manager:~ docker node ls
ID                            HOSTNAME        STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
uq9p2h38zf9i4k7i483chi8ur *   swarm-manager   Ready     Active         Leader           20.10.2
1vshlhq3uli6lk2yewc5jjm0y     swarm-node-1    Ready     Active                          20.10.2
018s4qa9tggfdy5lk7foa5v3m     swarm-node-2    Ready     Active                          20.10.2

```

9. On the manager, install the [gitlab runner](https://docs.gitlab.com/runner/install/linux-repository.html)

10. Here are the configuration settings i used

```
Enter the GitLab instance URL (for example, https://gitlab.com/):
https://gitlab.com
Enter the registration token:
REDACTED
Enter a description for the runner:
[swarm-manager]:
Enter tags for the runner (comma-separated):

Registering runner... succeeded                     runner=sAzPNGab
Enter an executor: custom, docker-ssh, docker+machine, docker-ssh+machine, docker, parallels, shell, ssh, virtualbox, kubernetes:
docker
Enter the default Docker image (for example, ruby:2.6):
docker:latest
```

11. Create a deployer user

12. [Allow docker usage as non root user](https://docs.docker.com/engine/install/linux-postinstall/)

13. [Create an ssh key pair](https://docs.gitlab.com/ee/ssh/README.html#ed25519-ssh-keys) so Gitlab CI can communicate with our swarm manager.

```
ssh-keygen -t ed25519 -C "<comment>"
```

14. Take your _private_ ssh key and install it as a pipeline variable, file type, cannot be protected or obfuscated.

```
cat ~/.ssh/id_ed25519
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

Use Lets Encrypt via certbot. This adds nginx_ensite which is not ideal, but I will tend to keep vhost configurations in the `sites_enabled` and them add the host in the main `/etc/nginx/nginx.conf`

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

```

# Resources

- https://www.digitalocean.com/community/tutorials/how-to-configure-the-linux-firewall-for-docker-swarm-on-ubuntu-16-04
```
