#  基于docker安装运行Nexus

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
  nexus:
    restart: always
    image: sonatype/nexus3
    container_name: nexus
    ports:
      - 8081:8081
    volumes:
      - /usr/local/docker/nexus/data:/nexus-data
```

***注： 启动时如果出现权限问题可以使用：`chmod 777 /usr/local/docker/nexus/data` 赋予数据卷目录可读可写的权限***



#### 登录控制台验证安装

#### 地址：http://ip:port/ 用户名：admin 密码：

#### Your **admin** user password is located in
**/nexus-data/admin.password** on the server.