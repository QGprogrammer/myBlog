#  基于docker安装运行Registry

#### 概述

官方的 Docker Hub 是一个用于管理公共镜像的地方，我们可以在上面找到我们想要的镜像，也可以把我们自己的镜像推送上去。但是，有时候我们的服务器无法访问互联网，或者你不希望将自己的镜像放到公网当中，那么你就需要 Docker Registry，它可以用来存储和管理自己的镜像。

#### 安装

使用 `docker-compose` 来安装，配置如下：

```text
version: '3.1'
services:
  registry:
    image: registry
    restart: always
    container_name: registry
    ports:
      - 5000:5000
    volumes:
      - /usr/local/docker/registry/data:/var/lib/registry
```

#### 测试

启动成功后需要测试服务端是否能够正常提供服务，有两种方式：

- 浏览器端访问

http://ip:5000/v2/

- 终端访问

```text
curl http://ip:5000/v2/
```



#### WebUi的安装

[docker-registry-web官网](https://hub.docker.com/r/hyper/docker-registry-web/)

使用`docker-compose.yml`启动一直失败，先用bash启动吧。

```bash
$ docker run -it -d -p 8082:8080 \
	-e REGISTRY_URL=http://registry-srv:5000/v2 \
	-e REGISTRY_NAME=localhost:5000 \
	hyper/docker-registry-web 
```



#### 配置客户端

在 `/etc/docker/daemon.json` 中增加如下内容（如果文件不存在请新建该文件）

```text
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ],
  "insecure-registries": [
    "ip:5000"
  ]
}
```

之后重新启动服务。

```bash
$ sudo systemctl restart docker
```