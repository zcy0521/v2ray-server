# V2Ray Server

## 配置BBR

https://github.com/google/bbr

```shell script
$ sudo nano /etc/sysctl.conf
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
$ sudo sysctl -p
```

## V2Ray

### 安装

https://github.com/v2fly/fhs-install-v2ray

```shell script
sudo -i
apt update
apt install curl

bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh)

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

### 配置 ws+kcp

```shell script
$ sudo nano /usr/local/etc/v2ray/config.json
```

```json
{
  "log": {
    "loglevel": "warning",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
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

### 运行

```shell script
sudo systemctl enable v2ray
sudo systemctl start v2ray
sudo systemctl restart v2ray
sudo systemctl status v2ray
```

## Nginx

### 安装

http://nginx.org/en/linux_packages.html#Debian

```shell script
sudo apt install curl gnupg2 ca-certificates lsb-release

echo "deb http://nginx.org/packages/debian `lsb_release -cs` nginx" sudo tee /etc/apt/sources.list.d/nginx.list
    
curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
sudo apt-key fingerprint ABF5BD827BD9BF62
sudo apt update
sudo apt install nginx
```

### 配置 ssl

- 上传证书

```shell scrpit
$ unzip <DOMAIN_NAME>.zip
$ scp -P 8022 <DOMAIN_NAME>.pem <DOMAIN_NAME>.key admin@<SERVER_IP>:/home/admin
```

- 服务器创建证书目录

```shell scrpit
$ sudo mkdir /etc/nginx/cert
$ cp /home/admin/<DOMAIN_NAME>.pem /etc/nginx/cert/<DOMAIN_NAME>.pem
$ cp /home/admin/<DOMAIN_NAME>.key /etc/nginx/cert/<DOMAIN_NAME>.key
```

- 配置https证书 **替换4处`<DOMAIN_NAME>`**

```shell scrpit
$ sudo touch /etc/nginx/conf.d/<DOMAIN_NAME>.conf
$ sudo nano /etc/nginx/conf.d/<DOMAIN_NAME>.conf
```

```
server {
    listen       80;
    server_name  <DOMAIN_NAME>;
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
    server_name <DOMAIN_NAME>;

    ssl_certificate /etc/nginx/cert/<DOMAIN_NAME>.pem;
    ssl_certificate_key /etc/nginx/cert/<DOMAIN_NAME>.key;
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
        proxy_pass http://127.0.0.1:10000/ray;
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

### 运行

```shell script
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
sudo systemctl restart nginx
```
