# ssl-for-sass-caddy

### setup server basics Ubuntu 18.04
```bash
apt install -y vim tree sudo zip unzip sqlite members libcap2-bin
```
### Install Caddy
```bash
curl https://getcaddy.com | bash -s personal http.cache,http.cors,http.expires,http.geoip,http.git,http.ipfilter,http.locale,http.nobots,http.ratelimit,http.realip
```

Caddy will run with its own user, we won't log in with it so we create it without shell.

```bash
sudo useradd -M -s /bin/false caddy
```

Caddy will need some folder with the correct rights to store configuration and logs.

```bash
sudo mkdir -p /etc/caddy
sudo mkdir -p /var/log/caddy
sudo chown -R caddy:root /etc/caddy /var/log/caddy
```
Caddy will need to listen on port 80 and 443, which are the standard port for HTTP and HTTPS. There is one easy way to give a program the rights to bind to port less than 1024 without being root. In the very first step we installed libcap2-bin, it's time to use it.

```bash
sudo setcap cap_net_bind_service=+ep /usr/local/bin/caddy
```

Copy a the content of `Caddyfile` from this repository. Make a caddy file and paste the content. 
```bash
sudo touch /etc/caddy/Caddyfile
sudo nano /etc/caddy/Caddyfile
```

#### Run Caddy as a deamon

We'll use system.d to run caddy. It makes it easy to run with the correct user, and easy to restart.

Create a caddy deamon definition in `/etc/systemd/system/caddy.service` . If you decided to store your logs or configuration somewhere else, remember the change the different paths in this file.
```bash
[Unit]
Description=Caddy HTTP/2 web server

[Service]
User=caddy
Group=caddy
Environment=CADDYPATH=/etc/caddy
ExecStart=/usr/local/bin/caddy -agree=true -log=/var/log/caddy/caddy.log -conf=/etc/caddy/Caddyfile -root=/dev/null
ExecReload=/bin/kill -USR1 $MAINPID
LimitNOFILE=1048576
LimitNPROC=64

[Install]
WantedBy=multi-user.target
```

Now, reload the configuration, start the service and check its status.
```bash
sudo systemctl daemon-reload
sudo systemctl start caddy
sudo systemctl status caddy
```

Make sure the deamon is started with the server is rebooted:

```bash
sudo systemctl enable caddy
```

:) Copied from https://www.sigerr.org/article/install-set-up-caddy-server-php-fpm-with-ssl-ubuntu