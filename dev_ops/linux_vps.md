1. SSH

- Gen ssh key:

```bash
ssh-keygen -t ed25519 -C "devuser@company"  # -C: Comment, who owner this key
   /home/<username>/.ssh/id_rsa_<server_name>
ls -l ~/.ssh/
cat /home/<username>/.ssh/id_rsa_<server_name>.pub

chmod 600 /paht_to_private_key # If need

# Update config
nano ~/.ssh/config
Host <alias_name>
    HostName xxx.xxx.xx.xxx
    User <username>
    Port xxx
ssh <alias_name>
```

```bash
Dùng ssh-copy-id (đơn giản nhất):
ssh-copy-id username@ip_vps

# Xem nội dung public key
cat ~/.ssh/id_ed25519.pub

# SSH vào VPS rồi paste vào file authorized_keys
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
# Paste nội dung public key vào, lưu lại

# Phân quyền trên VPS
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

- Add person:

```
sudo adduser new_user
groups new_user

ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub

mkdir /home/new_user/.ssh
sudo nano /home/new_user/.ssh/authorized_keys

sudo chown -R new_user:new_user /home/new_user/.ssh
sudo chmod 700 /home/new_user/.ssh
sudo chmod 600 /home/new_user/.ssh/authorized_keys

sudo usermod -aG docker new_user # Add full perrmisson on docker, near to root

sudo groupadd workspace
sudo usermod -aG workspace new_user
sudo chown -R root:workspace /workspace
sudo chmod -R 770 /workspace

sudo chmod g+s /workspace # 👉 Khi devuser tạo file mới: file tự động thuộc group workspace, tránh lỗi “người khác không sửa được”
```

- Update security
  
```bash
sudo nano /etc/ssh/sshd_config
```

```bash
# Thay đổi cổng SSH mặc định (22) sang cổng khác để giảm scan/bruteforce
Port 123456

# Tắt đăng nhập SSH bằng mật khẩu cho TẤT CẢ user
# Chỉ cho phép đăng nhập bằng SSH key
# Lưu ý: các file trong /etc/ssh/sshd_config.d/*.conf có thể override setting này
PasswordAuthentication no

# Không cho root đăng nhập bằng mật khẩu
# Root chỉ được đăng nhập bằng SSH key
PermitRootLogin prohibit-password

# Giới hạn số lần thử đăng nhập sai tối đa là 3 lần
# Có thể kết hợp với Fail2ban để tự động ban IP
MaxAuthTries 3

# Tắt X11 Forwarding để giảm bề mặt tấn công (không dùng GUI qua SSH)
X11Forwarding no

# Bật xác thực bằng public key (SSH key)
PubkeyAuthentication yes

# SSH keepalive: disconnect client sau  300 x 2 = 600 giây = 10 phút idle
ClientAliveInterval 300
ClientAliveCountMax 2

# Chỉ cho phép SSH:
# - root: từ mọi IP
# - junior_dev: chỉ từ dải ip xx.xx.*.*
AllowUsers root junior_dev@xx.xx.*.*

# Chỉ cho phép user root và user junior_dev đăng nhập SSH
# junior_dev chỉ được đăng nhập từ dải IP nội bộ 10.0.*.*
AllowUsers root junior_dev@10.0.*.*
```

```bash
sudo service sshd restart
```

- SSH Tunnel

```bash
ssh -R <server_port>:localhost:<local_port> -i ~/.ssh/xxx -p xxx root@xxx.xxx.xxx.xxx
```

2. User

```
# Create a user named 'deploy' (or any name you prefer)
sudo adduser --uid 10100 deploy
# useradd là lệnh gốc
# ❌ không tạo home directory
# ❌ không hỏi password
# ❌ không copy file .bashrc
# ❌ không hỏi thông tin user
# chỉ tạo record trong /etc/passwd

# Add the user to the sudo group if needed
usermod -aG sudo deploy

# Change ownership to the new user
chown -R deploy:deploy /path

# Changer user
su - deploy

# Thêm user hiện tại vào group docker
sudo usermod -aG docker $USER

# Kích hoạt thay đổi group (không cần logout)
newgrp docker

# Xoá user
sudo userdel deploy # Low-level utility
sudo deluser deploy # High-level wrapper
```

3. Nginx

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
# Socket io
sudo nano /etc/nginx/nginx.conf
http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Add new this one
    map $http_upgrade $connection_upgrade {
        default upgrade;
        ''      close;
    }
    # Add new this one
	
    sendfile on;
    keepalive_timeout 65;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}

sudo nano /etc/nginx/sites-available/default
server {
    listen 80;
    access_log off; # if don't want get logs
    server_name abc.com;

    location /socket.io/ {
        proxy_pass http://127.0.0.1:3001/socket.io/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Accept-Encoding "";
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }

    location / {
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:3001;
        proxy_redirect off;
    }
}

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

4. Ufw

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

5. Docker

```bash
# Thêm user hiện tại vào group docker
sudo usermod -aG docker $USER

# Kích hoạt thay đổi group (không cần logout)
newgrp docker
```

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
  # "iptables": false,
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

None root user
```
ARG UID=10100
ARG GID=10100

RUN addgroup -g $GID appgroup && adduser -u $UID -G appgroup -S appuser

WORKDIR /app
COPY --from=builder /app/cache/dist .
COPY --from=builder /app/cache/node_modules ./node_modules
COPY --from=builder /app/cache/package.json ./package.json
COPY --from=builder /app/cache/ecosystem.prod.config.cjs ./ecosystem.prod.config.cjs
COPY --from=builder /app/cache/kysely/seeds/data ./kysely/seeds/data

RUN chown -R appuser:appgroup /app

USER appuser

CMD ["pm2-runtime", "start", "ecosystem.prod.config.cjs"]

EXPOSE 3000
```

```root@ai-school-prod-dash-ubuntu-s-2vcpu-2gb-90gb-intel-nyc3-01:/workspace/production/ai_school/backend/public# ls -la
total 6992
drwxr-xr-x 3 root root    4096 Jun  7 04:24 .
drwxr-xr-x 9 root root    4096 Jul 26 04:13 ..
-rw-r--r-- 1 root root       0 Jun  6 00:15 README.md
drwxr-xr-x 5 root root    4096 Jun  6 10:49 assets
-rw-r--r-- 1 root root 7143791 Jun  6 11:03 assets.zip
root@ai-school-prod-dash-ubuntu-s-2vcpu-2gb-90gb-intel-nyc3-01:/workspace/production/ai_school/backend/public#
root@ai-school-prod-dash-ubuntu-s-2vcpu-2gb-90gb-intel-nyc3-01:/workspace/production/ai_school/backend/public#
root@ai-school-prod-dash-ubuntu-s-2vcpu-2gb-90gb-intel-nyc3-01:/workspace/production/ai_school/backend/public# sudo chown -R 10100:10100 assets
root@ai-school-prod-dash-ubuntu-s-2vcpu-2gb-90gb-intel-nyc3-01:/workspace/production/ai_school/backend/public# ls -la
total 6992
drwxr-xr-x 3 root  root     4096 Jun  7 04:24 .
drwxr-xr-x 9 root  root     4096 Jul 26 04:13 ..
-rw-r--r-- 1 root  root        0 Jun  6 00:15 README.md
drwxr-xr-x 5 10100 10100    4096 Jun  6 10:49 assets
-rw-r--r-- 1 root  root  7143791 Jun  6 11:03 assets.zip
```

6. Github Action self-host runner
```
# Install the service
sudo ./svc.sh install

# Start the service
sudo ./svc.sh start

# Check status
sudo ./svc.sh status

# Other useful commands:
# sudo ./svc.sh stop
# sudo ./svc.sh uninstall
```

6. Scp

```
# Upload
scp -i ~/.ssh/id_rsa_server -P 23 public/uploads.zip root@159.223.64.220:

# Download
scp -i ~/.ssh/id_rsa_...... -P 23 user@server:/path/to/remotefile.zip /Local/Target/Destination
```

7. Git

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

*. Check connect to VPS
   
```bash
sudo lastb # Lịch sử truy cập thất bại

# ssh:notty
# ssh: Đăng nhập qua giao thức SSH
# notty: "no TTY" = không có terminal thật (pseudo-terminal)
# Nghĩa là: Đây là các lần thử đăng nhập SSH tự động (bằng script/bot), không phải người dùng thật ngồi gõ lệnh
```

```bash
sudo last # Lịch sử truy cập thành công
```

```bash
last reboot -F
```

9. Fail2ban

```bash
sudo apt update
sudo apt install fail2ban -y
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

# Kiểm tra
sudo fail2ban-client status sshd
```

```bash
sudo nano /etc/fail2ban/jail.local

[DEFAULT]
# Tăng thời gian ban lên
bantime = 24h          # Ban 24 giờ thay vì 10 phút
findtime = 1h          # Theo dõi trong 1 giờ thay vì 10 phút
maxretry = 3           # Giảm xuống 3 lần thay vì 5

# Ban vĩnh viễn sau nhiều lần tái phạm
bantime.increment = true
bantime.multipliers = 1 5 10 50 100 500
bantime.maxtime = 5w   # Tối đa ban 5 tuần

[sshd]
enabled = true
port = 22123456789
logpath = /var/log/auth.log
maxretry = 3
findtime = 1h
bantime = 24h


sudo systemctl restart fail2ban
```

10. SSL

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

11. Monitoring

```
iotop: Disk I/O
htop: CPU / RAM
???: Bandwidth
```

12. Redis

```
echo 1 >/proc/sys/vm/overcommit_memory
```

13. Socket Statistics

```
ss # Apps nào đang listen port nào

Netid  State   Recv-Q  Send-Q  Local Address:Port    Peer Address:Port  Process
tcp    LISTEN  0       128     0.0.0.0:80           0.0.0.0:*          users:(("nginx",pid=1234))
tcp    LISTEN  0       128     127.0.0.1:3306       0.0.0.0:*          users:(("mysqld",pid=5678))
tcp    LISTEN  0       128     [::]:22              [::]:*             users:(("sshd",pid=910))
udp    UNCONN  0       0       0.0.0.0:53           0.0.0.0:*          users:(("systemd-resolve",pid=800))
```


<img width="767" height="340" alt="image" src="https://github.com/user-attachments/assets/86b5207e-7b5e-4ec9-92da-2e1eca164f74" />

<img width="760" height="350" alt="image" src="https://github.com/user-attachments/assets/0b1bdbc3-e088-4b1a-b405-f06576294d1c" />


```
sudo ss -tulpn state listening # Hay dùng
```


14. Nginx & Telegraf & Prometheus & Grafana

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
