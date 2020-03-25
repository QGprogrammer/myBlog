#  基于docker安装运行Nginx_反向代理

#### 何为代理服务器

客户端向服务器发起请求时，不直接访问目标服务器，而是中间服务器，该台服务器能向服务期进行通信，获取客户端所需资源，同时也可做一些额外的代理功能。

这种代理亦称为***正向代理***。

科学上网就是典型的正向代理。



#### 何为反向代理服务器

反向代理服务器顾名思义，代理对象为服务器端。客户端还是照常发起请求给代理机，至于最终会访问到哪台服务器的资源，由这个代理决定，它的作用的就是给***服务端做代理***。



#### 使用Nginx反向代理两台tomcat服务器

***编写docker-compose.yml***

```
version: '3'
services:
  tomcat1:
    image: tomcat
    container_name: tomcat1
    ports:
      - 8080:8080
```

***配置反向代理***

修改 /usr/local/docker/nginx/conf 目录下的 nginx.conf 配置文件：

[Nginx的安装](https://github.com/QGprogrammer/myBlog/blob/master/Linux/基于docker安装运行Nginx_虚拟主机.md)

```
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
	
	# 配置tomcat1代理服务器
	upstream tomcat1 {
		server 192.168.116.144:8080;
	}

	# 配置一个虚拟主机
	server {
		listen 80;
		server_name admin.ven.com;
		location / {
				# 域名 admin.ven.com 的请求全部转发到  tomcat1 服务上
				proxy_pass http://tomcat1;
				# 欢迎页面，按照从左到右的顺序查找页面
				index index.jsp index.html index.htm;
		}
	}
}
```

记得修改Hosts文件，把`admin.ven.com`域名映射到nginx服务器上。

这样客户端访问`admin.ven.com`的时候，就转发到tomcat上了。