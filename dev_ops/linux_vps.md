1. SSH

- On local:

```
ls -l ~/.ssh/id*
mkdir ~/.ssh
chmod 700 ~/.ssh
ssh-keygen
ssh-keygen -b 4096 -t rsa
   /home/<username>/.ssh/id_rsa_<server_name>
ls -l ~/.ssh/
cat /home/<username>/.ssh/id_rsa_<server_name>.pub

nano ~/.ssh/config
Host <alias_name>
    HostName xxx.xxx.xx.xxx
    User <username>
    Port xxx
ssh <alias_name>
```

- On server:

```bash
# Change port:

nano /etc/ssh/sshd_config
Port 123456
systemctl restart sshd
or
sudo systemctl restart ssh

# Test port
ss -tlnp | grep sshd
LISTEN 0      128          0.0.0.0:123456      0.0.0.0:*    users:(("sshd",pid=347278,fd=3))
LISTEN 0      128             [::]:123456         [::]:*    users:(("sshd",pid=347278,fd=4))

# Disable login with password for all accout
PasswordAuthentication no
nano /etc/ssh/sshd_config.d/50-cloud-init.conf  # It will mix with ssh/sshd_config
If see PasswordAuthentication yes, set it to no

If not working, read more about
ChallengeResponseAuthentication no

# Login with key:
sudo useradd -m -s /usr/bin/zsh <username>
sudo passwd <username>
ls -l ~/.ssh/
mkdir ~/.ssh
chmod 700 ~/.ssh
touch authorized_keys
chmod 600 ~/.ssh/authorized_keys
nano ~/.ssh/authorized_keys
```

2. nginx

```
sudo nano /etc/nginx/sites-available/default
server {
    listen 80;
    access_log off; # if don't want get logs
    server_name abc.com;

    location / {
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:5000;
        proxy_redirect off;
    }
}
```
```
sudo apt-get install apache2-utils
sudo htpasswd -c /etc/nginx/.htpasswd username
location /protected {
    auth_basic "Restricted Area"; # Message displayed in the auth prompt
    auth_basic_user_file /etc/nginx/.htpasswd; # Path to the password file
}
```

```
/etc/nginx/nginx.conf

events {
    worker_connections 20000;
}
```

```
sudo nano /etc/nginx/nginx.conf
client_max_body_size 100M;
```

```
sudo rm /var/log/nginx/access.log
sudo service nginx reload // after reload nginx, automatic recreate access.log and error.log
sudo truncate --size 0 /var/log/nginx/access.log // truncate to 0 kb and increase from 0 kb
```

```
Note: Remember config sticky session
https://github.com/socketio/socket.io/issues/4239#issuecomment-1011912700
https://socketio.bootcss.com/docs/using-multiple-nodes/#NginX-configuration
http {
  	server {
		listen 3000;
		server_name io.yourhost.com;

		location / {
			proxy_set_header X- Forwarded - For $proxy_add_x_forwarded_for;
			proxy_set_header Host $host;

			proxy_pass http://nodes;

			# enable WebSockets
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "upgrade";
		}
}

	upstream nodes {
	    # enable sticky session based on IP;
	    ip_hash;

	    server app01: 3000;
	    server app02: 3000;
	    server app03: 3000;
	}
}
```

3. ufw

```
sudo nano /etc/default/ufw
IPV6=yes

sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow ssh
ufw allow 'Nginx HTTP'
sudo ufw allow 1234/tcp

sudo ufw logging on

sudo ufw status
sudo ufw status numbered
sudo ufw delete 3
sudo ufw delete allow 22/tcp

sudo ufw status verbose (list)
sudo ufw reload

SSH
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
```

4. docker

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```
mkdir /etc/docker
sudo nano /etc/docker/daemon.json
{
  "iptables": false,
  "log-driver": "json-file", # none
  "log-opts": {
    "max-size": "1m",
    "max-file": "3"
  }
}
sudo service docker restart
sudo systemctl stop docker.socket // If can not restart and restart again
```
[sudo permission ](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)

5. scp

```
scp -P 2290 public/uploads.zip root@159.223.64.220:
scp -i ~/.ssh/id_rsa_server  -P 2290 public/uploads.zip root@159.223.64.220:
scp user@server:/path/to/remotefile.zip /Local/Target/Destination
```

6. npm

```
apt install npm
```

7. git

```
Multi ssh for diffrent project
ssh-keygen -t rsa -b 4096 -C "bamboo@gmail.com"
/root/.ssh/id_rsa_<app_name>
cat ~/.ssh/id_rsa_<app_name>.pub

nano ~/.ssh/config
Host github.com_<app_name>
	HostName github.com
	User git
	IdentityFile ~/.ssh/id_rsa_<app_name>

git clone git@github.com_<app_name>:xxx/xxx.git
```

8. SSL

```
sudo apt install certbot
sudo apt install python3-certbot-dns-cloudflare

cat ~/.secrets/certbot/cloudflare.ini
dns_cloudflare_api_token = cewqd2sqxsxs

sudo chmod 600 ~/.secrets/certbot/cloudflare.ini

certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini \
  -d abc.com \
  -d *.abc.com

sudo nano /etc/nginx/sites-available/default
server {
    listen 80;
    #access_log off;
    server_name abc.com;

    listen 443 ssl;

    # RSA certificate
    ssl_certificate /etc/letsencrypt/live/abc.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/abc.com/privkey.pem;

    location / {
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:4000;
        proxy_redirect off;
    }
}

sudo certbot renew --dry-run
sudo certbot renew --reuse-key
sudo certbot renew --reuse-key --dry-run

sudo crontab -e
0 1,13 * * * sudo certbot renew --reuse-key && sudo service nginx reload >> /var/log/letsencrypt/renew.log

ls -la /etc/letsencrypt/live/abc.com
```

9. Monitoring

```
iotop: Disk I/O
htop: CPU / RAM
???: Bandwidth
```

10. Redis

```
echo 1 >/proc/sys/vm/overcommit_memory
```

11. Nginx & Telegraf & Prometheus & Grafana

- Nginx

  Add nginx-module-vts module

  ```
  sudo apt-get update
  sudo apt-get install build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev
  nginx -v
  wget http://nginx.org/download/nginx-1.21.4.tar.gz
  tar -xzvf nginx-1.21.4.tar.gz
  git clone https://github.com/vozlt/nginx-module-vts.git

  cd nginx-1.21.4
  ./configure --add-module=../nginx-module-vts --with-http_ssl_module --with-stream --with-http_v2_module
  make
  # Config from /etc/nginx/nginx.conf is applyed when run nginx which install from apt
  sudo cp /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.backup
  sudo make install

  sudo nano /usr/local/nginx/conf/nginx.conf

  http {
    vhost_traffic_status_zone;

    server {
      listen 80;

      location /status {
        vhost_traffic_status_display;
        vhost_traffic_status_display_format prometheus;
        }
      }

      log_format metrics '$remote_addr - $remote_user [$time_local] '
                          '"$request" $status $body_bytes_sent '
                          '"$http_referer" "$http_user_agent" '
                          '$request_time';
  }

  sudo systemctl stop nginx
  sudo touch /usr/local/nginx/logs/nginx.pid
  sudo /usr/local/nginx/sbin/nginx
  sudo /usr/local/nginx/sbin/nginx -s reload
  http://<NGINX_IP>/status
  /usr/local/nginx/sbin/nginx -V 2>&1 | grep --color -o vts
  ```

  Add nginx_status modules

  ```
  ./configure --with-http_stub_status_module

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';

          location /nginx_status {
              stub_status;
              allow 127.0.0.1;
              deny all;
          }

  ```

- Telegraf

  ```
  sudo nano /etc/telegraf/telegraf.conf
  sudo apt remove telegraf
  sudo apt install telegraf

  sudo systemctl start telegraf
  sudo journalctl -u telegraf -n 30

  sudo systemctl daemon-reload
  sudo systemctl restart telegraf
  sudo systemctl status telegraf

  sudo telegraf --config /etc/telegraf/telegraf.conf --test


  sudo nano /etc/systemd/system/telegraf.service

  [Service]
  Environment="TELEGRAF_OPTS="
  ExecStart=/usr/bin/telegraf --config /etc/telegraf/telegraf.conf


  sudo nano /etc/telegraf/telegraf.conf
  [global_tags]

  [agent]
    interval = "10s"
    round_interval = true
    metric_batch_size = 1000
    metric_buffer_limit = 10000
    collection_jitter = "0s"
    flush_interval = "10s"
    flush_jitter = "0s"
    precision = ""
    debug = false
    quiet = false
    logfile = "/var/log/telegraf/telegraf.log"

  ###############################################################################
  #                                  INPUTS                                     #
  ###############################################################################

  [[inputs.nginx]]
    urls = ["http://localhost/nginx_status"]
    response_timeout = "5s"
  [[inputs.tail]]
    name_override = "nginxlog"
    files = ["/var/log/nginx/access.log"]
    from_beginning = true # Be careful
    pipe = false
    data_format = "grok"
    grok_patterns = ["%{COMBINED_LOG_FORMAT}"]
  [[inputs.cpu]]
    percpu = true
  [[inputs.disk]]
  [[inputs.diskio]]
  [[inputs.net]]
  [[inputs.mem]]
  [[inputs.system]]


  ###############################################################################
  #                                  OUTPUTS                                    #
  ###############################################################################

  [[outputs.prometheus_client]]
    listen = ":9273"
    # path = "/metrics" # This is default

  # Configuration for sending metrics to InfluxDB
  #[[outputs.influxdb]]
  # urls = ["http://localhost:8086"]
  # database = "telegraf"
  # username = "user_123"
  # password = "password_123"

  # Uncomment to send metrics directly to Grafana Loki
  # [[outputs.loki]]
  #   urls = ["http://localhost:3100/loki/api/v1/push"]

  # Configuration for sending metrics to Prometheus
  # [[outputs.prometheus_client]]
  #   listen = ":9273"


  sudo systemctl unmask telegraf.service
  sudo systemctl daemon-reload
  sudo systemctl restart telegraf.service
  ```

- Prometheus

  ```
  prothemius
  nano /etc/prometheus/prometheus.yml
  scrape_configs:
    - job_name: 'telegraf'
      static_configs:
        - targets: ["telegraf_export_cross_3.ecomobileapp.com"]
      scheme: https

  sudo systemctl restart prometheus

  ```
