#  Linux配置Maven环境

##### 下载`apache-maven-3.6.3-bin.tar.gz`压缩包放于 `/usr/local/maven`并解压

```bash
$ cd /usr/local/java
$ tar -xzvf jdk-8u241-linux-x64.tar.gz
$ rm -f jdk-8u241-linux-x64.tar.gz
```

### 配置环境变量

#### 配置用户环境变量

```bash
$ vi /etc/profile
```

#### 添加如下语句

```text
export MAVEN_HOME=/usr/local/maven/apache-maven-3.6.3
export PATH=$PATH:$MAVEN_HOME/bin
```

#### 使用户环境变量生效

```bash
$ source /etc/profile
```

#### 测试是否安装成功

```bash
$ mvn -version
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /usr/local/maven/apache-maven-3.6.3
Java version: 1.8.0_241, vendor: Oracle Corporation, runtime: /usr/local/java/jdk1.8.0_241/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.4.0-142-generic", arch: "amd64", family: "unix"
```

#### 增加Nexus私服配置

```bash
$ vi /usr/local/maven/apache-maven-3.6.3/conf
# 增加
        <server>
			<id>nexus-releases</id>
			<username>admin</username>
			<password>admin</password>
		</server>
		
		<server>
			<id>nexus-snapshots</id>
			<username>admin</username>
			<password>admin</password>
		</server>
		<server>
			<id>nexus-public</id>
			<username>admin</username>
			<password>admin</password>
		</server>
```

