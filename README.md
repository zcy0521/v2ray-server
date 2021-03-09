# V2Ray Server

## V2Ray(tcp+kcptun)

### 安装[V2Ray](https://github.com/v2fly/fhs-install-v2ray)

```shell
sudo -i
apt update
apt install curl

bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh)
```

### 配置V2Ray服务

- 编辑`/usr/local/etc/v2ray/config.json`

```json

```

- 运行V2Ray

```shell
sudo systemctl enable v2ray
sudo systemctl start v2ray
sudo systemctl status v2ray

sudo systemctl restart v2ray
```

### 安装[Kcptun](https://github.com/xtaci/kcptun)

```shell
wget https://github.com/xtaci/kcptun/releases/download/v20201010/kcptun-linux-amd64-20201010.tar.gz
tar -xzf kcptun-linux-amd64-20201010.tar.gz
sudo cp server_linux_amd64 /usr/bin/kcptun
sudo chmod +x /usr/bin/kcptun
```

- 编辑`~/.bashrc`并执行`ulimit -n 65535`

```
ulimit -n 65535
```

- 编辑`/etc/sysctl.conf`并执行`sudo sysctl -p`

```
net.core.rmem_max=26214400
net.core.rmem_default=26214400
net.core.wmem_max=26214400
net.core.wmem_default=26214400
net.core.netdev_max_backlog=2048
```

### 配置Kcptun服务

- 编辑[`/etc/kcptun/config.json`](https://github.com/xtaci/kcptun/issues/251)

```json
{
  "listen": ":4000",
  "target": "127.0.0.1:10000",
  "key": "HelloKcptun!",
  "crypt": "aes-128",
  "mode": "fast3",
  "mtu": 1400,
  "sndwnd": 4096,
  "rcvwnd": 4096,
  "datashard": 30,
  "parityshard": 15,
  "dscp": 46,
  "nocomp": true,
  "acknodelay": false,
  "nodelay": 0,
  "interval": 20,
  "resend": 2,
  "nc": 1,
  "sockbuf": 4194304,
  "keepalive": 10,
  "log": "/var/log/kcptun.log"
}
```

- 编辑`/usr/lib/systemd/system/kcptun.service`

```
Description=kcptun

Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
Environment=GOGC=20
ExecStart=/usr/bin/kcptun -c /etc/kcptun/config.json
Restart=on-failure
RestartSec=10
KillMode=process
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

- 运行kcptun

```shell
sudo systemctl enable kcptun
sudo systemctl start kcptun
sudo systemctl status kcptun

sudo systemctl restart kcptun
```

## V2Ray(ws+tls)

### 安装[V2Ray](https://github.com/v2fly/fhs-install-v2ray)

```shell
sudo -i
apt update
apt install curl

bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh)
```

### 配置V2Ray服务

- 编辑`/usr/local/etc/v2ray/config.json`

```json

```

- 运行V2Ray

```shell
sudo systemctl enable v2ray
sudo systemctl start v2ray
sudo systemctl status v2ray

sudo systemctl restart v2ray
```

### 安装[Nginx](http://nginx.org/en/linux_packages.html#Debian)

- 安装依赖

```shell
sudo apt install curl gnupg2 ca-certificates lsb-release
```

- 添加源

```shell
echo "deb http://nginx.org/packages/debian `lsb_release -cs` nginx" sudo tee /etc/apt/sources.list.d/nginx.list
curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
sudo apt-key fingerprint ABF5BD827BD9BF62
```

- 安装Nginx 

```shell
sudo apt update
sudo apt install nginx
```

### 配置SSL证书

- 上传ssl证书

```shell
scp -P 8022 [cert-file-name].pem [cert-file-name].key root@SERVER_IP:/root

sudo mkdir /etc/nginx/cert
cp [cert-file-name].pem [cert-file-name].key /etc/nginx/cert/
```

- 编辑`/etc/nginx/nginx.conf` `http{}`

```
http {
        server {
                listen 80;
                server_name [yourdomain.com];
                rewrite ^(.*)$ https://$host$1;
                location / {
                        index index.html index.htm;
                }
        }

        server {
                listen 443 ssl;
                server_name [yourdomain.com];
                root html;
                index index.html index.htm;
                ssl_certificate cert/[cert-file-name].pem;
                ssl_certificate_key cert/[cert-file-name].key;
                ssl_session_timeout 5m;
                ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
                ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
                ssl_prefer_server_ciphers on;

                location / {
                        root html;
                        index index.html index.htm;
                }

                location /ray {
                        if ($http_upgrade != "websocket") {
                                return 404;
                        }
                        proxy_redirect off;
                        proxy_pass http://127.0.0.1:10001/ray;
                        proxy_http_version 1.1;
                        proxy_set_header Upgrade $http_upgrade;
                        proxy_set_header Connection "upgrade";
                        proxy_set_header Host $host;
                        # Show real IP in v2ray access.log
                        proxy_set_header X-Real-IP $remote_addr;
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                }
        }
}
```

- 运行Nginx

```shell
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx

sudo systemctl restart nginx
```

## 可选配置

### 修改SSH端口

- 编辑`/etc/ssh/sshd_config`并执行`sudo systemctl restart ssh`

```shell
Port = 8022
```

### 配置[BBR](https://github.com/google/bbr)

- 编辑`/etc/sysctl.conf`并执行`sudo sysctl -p`

```shell
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```
