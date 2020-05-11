#  基于docker安装运行Nginx_虚拟主机

在 `/usr/local`目录下创建目录`docker`

```bash
$ cd /usr/local
$ mkdir docker
$ cd /docker
```

创建`docker-compose.yml`文件

```text
version: '3.1'
services:
  nginx:
    restart: always
    image: nginx
    container_name: nginx
    ports:
      - 80:80
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./wwwroot:/usr/share/nginx/wwwroot
```

虚拟主机是一种特殊的软硬件技术，它可以将网络上的每一台计算机分成多个虚拟主机，每个虚拟主机可以独立对外提供 www 服务，这样就可以实现一台主机对外提供多个 web 服务，每个虚拟主机之间是独立的，互不影响的。

通过 Nginx 可以实现虚拟主机的配置，Nginx 支持三种类型的虚拟主机配置

- 基于 IP 的虚拟主机
- 基于域名的虚拟主机
- 基于端口的虚拟主机



#### Nginx配置文件结构

```xml
events {
	...
}

http {
	# 虚拟主机1
	server {
		...
	}
	#虚拟主机2
	server{
		...
	}
}
```



#### 基于端口的虚拟主机配置

在 `/usr/local/docker/nginx/wwwroot` 目录下创建 `server1` 和 `server2` 两个目录，并分辨创建两个 `index.html` 文件

修改 `/usr/local/docker/nginx/conf` 目录下的 `nginx.conf` 配置文件：

```text
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    # 配置虚拟主机1
    server {
	# 监听端口
        listen       80;
	# 虚拟主机ip地址
        server_name  192.168.116.139;
	# location后跟请求的匹配 / 表示全部请求
        location / {
	    # root 指定虚拟主机网页目录
            root   /usr/share/nginx/wwwroot/server1;
	    # 指定欢迎页面，按从左到右顺序查找
            index  index.html index.htm;
        }

    }
    # 配置虚拟主机2
    server {
        listen       8080;
        server_name  192.168.116.114;
        location / {
            root   /usr/share/nginx/wwwroot/server2;
            index  index.html index.htm;
        }
    }
}
```



#### 基于域名的虚拟主机配置

- 配置 Windows Hosts 文件

- 通过 host 文件指定 admin.ven.com 和 oa.ven.com 对应 192.168.116.144 虚拟机：

在 `/usr/local/docker/nginx/wwwroot` 目录下创建 admin和 oa两个目录，并分辨创建两个 index.html 文件

```text
user  nginx;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  admin.ven.com;
        location / {
            root   /usr/share/nginx/wwwroot/admin;
            index  index.html index.htm;
        }

    }

    server {
        listen       80;
        server_name  oa.ven.com;
        location / {
            root   /usr/share/nginx/wwwroot/oa;
            index  index.html index.htm;
        }
    }
}
```