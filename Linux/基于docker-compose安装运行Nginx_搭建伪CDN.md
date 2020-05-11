#  基于docker安装运行Nginx_搭建伪CDN

*如未安装nginx，参考**[Nginx的安装](https://github.com/QGprogrammer/myBlog/blob/master/Linux/基于docker安装运行Nginx_虚拟主机.md)*

#### 何为CDN？

>**Content Delivery Net**
>
>将源站内容分发至最接近用户的节点，使用户可就近取得所需内容，提高用户访问的响应速度和成功率。
>解决因分布、带宽、服务器性能带来的访问延迟问题，适用于站点加速、点播、直播等场景。

考虑成本问题，一般个人或公司不会(zuo si)自行搭建CDN，可以购买一些大厂的CDN加速服务即可。



**下文展示了基于Nginx搭建伪CDN。**

这样就可以将一些静态资源放到这里，而不用写进项目中，省去了耗时的编译打包。



*如未安装nginx，参考[Nginx的安装](https://github.com/QGprogrammer/myBlog/blob/master/Linux/基于docker安装运行Nginx_虚拟主机.md)*



修改 /usr/local/docker/nginx/conf 目录下的 nginx.conf 配置文件：

```xml

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
        server_name  192.168.116.139;
        location / {
            root   /usr/share/nginx/wwwroot/cdn;
            index  index.html index.htm;
        }

    }
}

```

然后将静态资源文件放到cdn目录下即可。