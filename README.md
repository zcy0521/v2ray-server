# V2Ray Server

## 修改SSH

```shell script
$ sudo nano /etc/ssh/sshd_config
Port = 8022

$ sudo systemctl restart ssh
```

## 配置BBR

https://github.com/google/bbr

```shell script
$ sudo nano /etc/sysctl.conf

net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

$ sudo sysctl -p
```

## V2Ray

### 安装V2Ray

https://github.com/v2fly/fhs-install-v2ray

```shell script
$ sudo -i
$ apt update
$ apt install curl

$ bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
$ bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh)
installed: /usr/local/bin/v2ray
installed: /usr/local/bin/v2ctl
installed: /usr/local/share/v2ray/geoip.dat
installed: /usr/local/share/v2ray/geosite.dat
installed: /usr/local/etc/v2ray/config.json
installed: /var/log/v2ray/
installed: /var/log/v2ray/access.log
installed: /var/log/v2ray/error.log
installed: /etc/systemd/system/v2ray.service
installed: /etc/systemd/system/v2ray@.service
```

### 配置V2Ray(vmess+kcp+ws+shadowsocks)

```shell script
sudo nano /usr/local/etc/v2ray/config.json
```

```json
{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "protocol": [
          "bittorrent"
        ],
        "outboundTag": "blocked"
      }
    ]
  },
  "inbounds": [
    {
      "port": 10000,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811"
          }
        ]
      },
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      }
    },
    {
      "port": 10001,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811"
          }
        ]
      },
      "streamSettings": {
        "network": "kcp",
        "kcpSettings": {
          "mtu": 1350,
          "tti": 10,
          "uplinkCapacity": 200,
          "downlinkCapacity": 200,
          "congestion": false,
          "readBufferSize": 2,
          "writeBufferSize": 2,
          "header": {
            "type": "none"
          }
        }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      }
    },
    {
      "port": 10002,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811"
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/ray"
        }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      }
    },
    {
      "port": 20000,
      "protocol": "shadowsocks",
      "settings": {
        "method": "chacha20-poly1305",
        "password": "IamAGoodMan",
        "network": "tcp,udp"
      },
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ]
}
```

### 运行V2Ray

```shell script
sudo systemctl enable v2ray
sudo systemctl start v2ray
sudo systemctl status v2ray

sudo systemctl restart v2ray
```

## Kcptun

### 参数优化

https://github.com/xtaci/kcptun

- 执行 ulimit -n 65535

```shell script
$ sudo nano ~/.bashrc

ulimit -n 65535
```

- 编辑 /etc/sysctl.conf

```shell script
$ sudo nano /etc/sysctl.conf

net.core.rmem_max = 26214400
net.core.rmem_default = 26214400
net.core.wmem_max = 26214400
net.core.wmem_default = 26214400
net.core.netdev_max_backlog = 2048

$ sudo sysctl -p
```

### 下载Kcptun

https://github.com/xtaci/kcptun/releases/

```shell script
wget https://github.com/xtaci/kcptun/releases/download/v20201010/kcptun-linux-amd64-20201010.tar.gz

tar -xzf kcptun-linux-amd64-20201010.tar.gz
```

### 配置Kcptun

https://github.com/xtaci/kcptun/issues/251

- Kcptun to V2Ray

```shell script
sudo mkdir /etc/kcptun
sudo nano /etc/kcptun/config.v2ray.json
```

```json
{
  "listen": ":4000",
  "target": "127.0.0.1:10000",
  "key": "KcptunPassword",
  "crypt": "aes-128",
  "mode": "fast3",
  "mtu": 1400,
  "sndwnd": 5120,
  "rcvwnd": 5120,
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
  "log": "/var/log/kcptun.v2ray.log"
}
```

- Kcptun to Shadowsocks

```shell script
sudo nano /etc/kcptun/config.ss.json
```

```json
{
  "listen": ":4001",
  "target": "127.0.0.1:20000",
  "key": "KcptunPassword",
  "crypt": "aes-128",
  "mode": "fast3",
  "mtu": 1400,
  "sndwnd": 5120,
  "rcvwnd": 5120,
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
  "log": "/var/log/kcptun.ss.log"
}
```

### 运行Kcptun

- Kcptun to V2Ray

```shell script
sudo nano /usr/lib/systemd/system/kcptun.v2ray.service
```

```
Description=kcptun.v2ray

Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
Environment=GOGC=20
ExecStart=/usr/bin/kcptun -c /etc/kcptun/config.v2ray.json
Restart=on-failure
RestartSec=10
KillMode=process
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

- Kcptun to Shadowsocks

```shell script
sudo nano /usr/lib/systemd/system/kcptun.ss.service
```

```
Description=kcptun.ss

Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
Environment=GOGC=20
ExecStart=/usr/bin/kcptun -c /etc/kcptun/config.ss.json
Restart=on-failure
RestartSec=10
KillMode=process
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

- 运行

```shell script
sudo cp server_linux_amd64 /usr/bin/kcptun

sudo chmod +x /usr/bin/kcptun /usr/lib/systemd/system/kcptun.v2ray.service /usr/lib/systemd/system/kcptun.ss.service

sudo systemctl enable kcptun.v2ray kcptun.ss
sudo systemctl start kcptun.v2ray kcptun.ss
sudo systemctl status kcptun.v2ray kcptun.ss

sudo systemctl restart kcptun.v2ray kcptun.ss
```

## Nginx

### 安装Nginx

http://nginx.org/en/linux_packages.html#Debian

```shell script
sudo apt install curl gnupg2 ca-certificates lsb-release

echo "deb http://nginx.org/packages/debian `lsb_release -cs` nginx" sudo tee /etc/apt/sources.list.d/nginx.list
curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
sudo apt-key fingerprint ABF5BD827BD9BF62

sudo apt update
sudo apt install nginx
```

### 配置Nginx ssl

- 上传证书

```shell scrpit
unzip DOMAIN_NAME.zip
scp -P 8022 DOMAIN_NAME.pem DOMAIN_NAME.key admin@SERVER_IP:/home/admin
```

- 服务器创建证书目录

```shell scrpit
sudo mkdir /etc/nginx/cert
cp DOMAIN_NAME.pem DOMAIN_NAME.key /etc/nginx/cert/
```

- 配置https证书 **替换4处`DOMAIN_NAME`**

```shell scrpit
sudo nano /etc/nginx/conf.d/DOMAIN_NAME.conf
```

```
server {
    listen       80;
    server_name  [DOMAIN_NAME];
    rewrite ^(.*)$ https://$host$1 permanent;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

# HTTPS server
server {
    listen 443 ssl;
    server_name [DOMAIN_NAME];

    ssl_certificate /etc/nginx/cert/[DOMAIN_NAME].pem;
    ssl_certificate_key /etc/nginx/cert/[DOMAIN_NAME].key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    
    location /fly {
        if ($http_upgrade != "websocket") {
            return 404;
        }
        proxy_redirect off;
        proxy_pass http://127.0.0.1:10002/ray;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        # Show real IP in v2ray access.log
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### 运行Nginx

```shell script
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx

sudo systemctl restart nginx
```
