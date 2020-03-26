#  基于docker安装运行Nginx_负载均衡

*如未安装nginx，参考[Nginx的安装](https://github.com/QGprogrammer/myBlog/blob/master/Linux/基于docker安装运行Nginx_虚拟主机.md)*

*tomcat的部署，参考[基于docker安装运行Nginx_反向代理](https://github.com/QGprogrammer/myBlog/blob/master/Linux/基于docker安装运行Nginx_反向代理.md)*

修改 `/usr/local/docker/nginx/conf` 目录下的 nginx.conf 配置文件：

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
	upstream group {
		server 192.168.116.144:8080 weight=10;
		server 192.168.116.144:8081 weight=10;
	}

	server {
		listen 80;
		server_name admin.ven.com;
		location / {
			proxy_pass http://group;
			index index.jsp index.html index.htm;
		}
	}
}
```

- `upstream`：每个设备的状态:
- `down`：表示当前的 `server` 暂时不参与负载
- `weight`：默认为 1 `weight` 越大，负载的权重就越大。
- `max_fails`：允许请求失败的次数默认为 1 当超过最大次数时，返回 `proxy_next_upstream` 模块定义的错误
- `fail_timeout`:`max_fails` 次失败后，暂停的时间。
- `backup`：其它所有的非 `backup` 机器 `down` 或者忙的时候，请求 `backup` 机器。所以这台机器压力会最轻