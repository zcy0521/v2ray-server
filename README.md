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

- 配置 V2Ray `/etc/v2ray/config.json`

- 运行 V2Ray `service v2ray start|stop|status|reload|restart|force-reload`

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
