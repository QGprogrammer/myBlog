#  使用VMware安装Ubantu

### 大致流程如下：

* **下载并安装VMware**

* **下载Ubantu镜像文件**

* **使用VM创建虚拟机**

* **更换下载源**

  >APT(Advanced Packaging Tool) 是 Debian/Ubuntu 类 Linux 系统中的软件包管理程序, 使用它可以找到想要的软件包, 而且安装、卸载、更新都很简便；也可以用来对 Ubuntu 进行升级; APT 的源文件为 `/etc/apt/` 目录下的 `sources.list` 文件。

  #### 查看系统版本

  ```
  lsb_release -a
  ```

  输出结果为：

  ```
  No LSB modules are available.
  Distributor ID:	Ubuntu
  Description:	Ubuntu 16.04 LTS
  Release:	16.04
  Codename:	xenial
  ```

  **注意：** Codename 为 `xenial`，该名称为我们 Ubuntu 系统的名称，修改数据源需用到该名称

  #### 编辑数据源

  ```
  vi /etc/apt/sources.list
  ```

  删除全部内容并修改为：

  ```
  deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
  ```

  ####  更新数据源

  ```
  apt-get update
  ```

  #### 常用 APT 命令

  #### 安装软件包

  ```text
  apt-get install packagename
  ```

  #### 删除软件包

  ```text
  apt-get remove packagename
  ```

  #### 更新软件包列表

  ```text
  apt-get update
  ```

  #### 升级有可用更新的系统（慎用）

  ```text
  apt-get upgrade
  ```