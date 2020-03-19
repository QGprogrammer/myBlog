#  Linux配置Java环境

##### 下载`jdk-8u241-linux-x64.tar.gz`压缩包放于 `/usr/local/java`并解压

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
export JAVA_HOME=/usr/local/java/jdk1.8.0_241
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib
export PATH=$JAVA_HOME/bin:$PATH
```

#### 使用户环境变量生效

```bash
$ source /etc/profile
```

#### 测试是否安装成功

```bash
$ java -version
java version "1.8.0_152"
Java(TM) SE Runtime Environment (build 1.8.0_152-b16)
Java HotSpot(TM) 64-Bit Server VM (build 25.152-b16, mixed mode)

$ echo $PATH
$ echo $CLASSPATH
```