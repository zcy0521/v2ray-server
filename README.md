# v2ray-server

## 安装

### Linux

[Github](https://github.com/v2ray/v2ray-core)

[Linux 安装脚本](https://www.v2ray.com/chapter_00/install.html#linuxscript)

- 安装 V2Ray

```shell script
sudo apt update
sudo apt install curl
sudo su
bash <(curl -L -s https://install.direct/go.sh)
```

- 配置 V2Ray

```shell script
sudo vim /etc/v2ray/config.json
{
  "inbounds": [{
    "port": 18993,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "9c04e2fb-107a-4bba-81be-9afeca38b20a",
          "level": 1,
          "alterId": 64
        }
      ]
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```

- 运行 V2Ray

```shell script
service v2ray start
service v2ray stop
service v2ray status
service v2ray reload
service v2ray restart
service v2ray force-reload
```

### Docker

[DockerHub](https://hub.docker.com/r/v2fly/v2fly-core)

[Docker](https://www.v2ray.com/chapter_00/install.html#docker)

- 下载镜像

```shell script
sudo docker pull v2fly/v2fly-core
```

- 运行容器

```shell script
git clone https://github.com/zcy0521/v2ray-server.git
cd v2ray-server
sudo docker-compose up -d
```

## 服务器优化

### [bbr](https://github.com/google/bbr)

```shell script
sudo vi /etc/sysctl.conf
net.ipv4.tcp_congestion_control=bbr
net.core.default_qdisc=fq

sudo sysctl --system
```
