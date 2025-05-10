## 2025/05/06

## Docker インストール

https://www.server-world.info/query?os=Ubuntu_24.04&p=docker&f=1

sudo apt -y install docker.io
docker version

## Docker Composer インストール

https://www.server-world.info/query?os=Ubuntu_24.04&p=docker&f=7

```
$ sudo apt install curl
$ sudo curl -L https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
$ sudo -s
# chmod 755 /usr/local/bin/docker-compose
# /usr/local/bin/docker-compose --version
```

インストールはしたけど、Podman使うので、Dockerは使わないかな…。