#  基于docker安装运行GitLab Runner实现持续集成

### 概述

互联网软件的开发和发布，已经形成了一套标准流程，最重要的组成部分就是持续集成（Continuous integration，简称CI）。

持续集成指的是，频繁地（一天多次）将代码集成到主干。它的好处主要有两个：

- 快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易。
- 防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。

Martin Fowler 说过，"持续集成并不能消除 Bug，而是让它们非常容易发现和改正。"

持续集成强调开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。

与持续集成相关的，还有两个概念，分别是持续交付和持续部署。

### 持续交付

持续交付（Continuous delivery）指的是，频繁地将软件的新版本，交付给质量团队或者用户，以供评审。如果评审通过，代码就进入生产阶段。

持续交付可以看作持续集成的下一步。它强调的是，不管怎么更新，软件是随时随地可以交付的。

持续交付在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的「类生产环境」（production-like environments）中。比如，我们完成单元测试后，可以把代码部署到连接数据库的 Staging 环境中更多的测试。如果代码没有问题，可以继续手动部署到生产环境中。

### 持续部署

持续部署（continuous deployment）是持续交付的下一步，指的是代码通过评审以后，自动部署到生产环境。

持续部署的目标是，代码在任何时刻都是可部署的，可以进入生产阶段。

持续部署的前提是能自动化完成测试、构建、部署等步骤。



理解了上面的基本概念之后，有没有觉得少了些什么东西 —— 由谁来执行这些构建任务呢？ 答案就是 GitLab Runner 了！

想问为什么不是 GitLab CI 来运行那些构建任务？

一般来说，构建任务都会占用很多的系统资源 (譬如编译代码)，而 GitLab CI 又是 GitLab 的一部分，如果由 GitLab CI 来运行构建任务的话，在执行构建任务的时候，GitLab 的性能会大幅下降。

GitLab CI 最大的作用是管理各个项目的构建状态，因此，运行构建任务这种浪费资源的事情就交给 GitLab Runner 来做拉！

因为 GitLab Runner 可以安装到不同的机器上，所以在构建任务运行期间并不会影响到 GitLab 的性能

### 安装

- 使用Dockerfile自定义一个配齐环境的镜像

  ```
  FROM gitlab/gitlab-runner
  
  # 修改软件源
  RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse' > /etc/apt/sources.list && \
      echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse' >> /etc/apt/sources.list && \
      echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse' >> /etc/apt/sources.list && \
      echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse' >> /etc/apt/sources.list && \
      apt-get update -y && \
      apt-get clean
  
  # 安装 Docker
  RUN curl -fsSL get.docker.com -o get-docker.sh && \
  	sh get-docker.sh --mirror Aliyun
  COPY daemon.json /etc/docker/daemon.json
  
  # 安装 Docker Compose
  WORKDIR /usr/local/bin
  RUN wget https://raw.githubusercontent.com/topsale/resources/master/docker/docker-compose
  RUN chmod +x docker-compose
  
  # 安装 Java
  RUN mkdir -p /usr/local/java
  WORKDIR /usr/local/java
  COPY jdk-8u152-linux-x64.tar.gz /usr/local/java
  RUN tar -zxvf jdk-8u152-linux-x64.tar.gz && \
      rm -fr jdk-8u152-linux-x64.tar.gz
  
  # 安装 Maven
  RUN mkdir -p /usr/local/maven
  WORKDIR /usr/local/maven
  RUN wget https://raw.githubusercontent.com/topsale/resources/master/maven/apache-maven-3.5.3-bin.tar.gz
  # COPY apache-maven-3.5.3-bin.tar.gz /usr/local/maven
  RUN tar -zxvf apache-maven-3.5.3-bin.tar.gz && \
      rm -fr apache-maven-3.5.3-bin.tar.gz
  # COPY settings.xml /usr/local/maven/apache-maven-3.5.3/conf/settings.xml
  
  # 配置环境变量
  ENV JAVA_HOME /usr/local/java/jdk1.8.0_152
  ENV MAVEN_HOME /usr/local/maven/apache-maven-3.5.3
  ENV PATH $PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin
  
  WORKDIR /
  ```

  使用docker-compose 启动

  ```
  version: '3.1'
  services:
    gitlab-runner:
      build: environment
      restart: always
      container_name: gitlab-runner
      privileged: true
      volumes:
        - /usr/local/docker/runner/config:/etc/gitlab-runner
        - /var/run/docker.sock:/var/run/docker.sock
  ```

  

### 注册 Runner

安装好 GitLab Runner 之后，我们只要启动 Runner 然后和 GitLab CI 绑定：

```bash
$ docker exec -it gitlab-runner gitlab-runner register

# 输入 GitLab 地址
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://192.168.75.146:8080/

# 输入 GitLab Token
Please enter the gitlab-ci token for this runner:
1Lxq_f1NRfCfeNbE5WRh

# 输入 Runner 的说明
Please enter the gitlab-ci description for this runner:
可以为空

# 设置 Tag，可以用于指定在构建规定的 tag 时触发 ci
Please enter the gitlab-ci tags for this runner (comma separated):bash
deploy

# 这里选择 true ，可以用于代码上传后直接执行
Whether to run untagged builds [true/false]:
true

# 这里选择 false，可以直接回车，默认为 false
Whether to lock Runner to current project [true/false]:
false

# 选择 runner 执行器，这里我们选择的是 shell
Please enter the executor: virtualbox, docker+machine, parallels, shell, ssh, docker-ssh+machine, kubernetes, docker, docker-ssh:
shell
```

### .gitlab-ci.yml

在项目工程下编写 .gitlab-ci.yml 配置文件：

```text
stages:
  - install_deps
  - test
  - build
  - deploy_test
  - deploy_production

cache:
  key: ${CI_BUILD_REF_NAME}
  paths:
    - node_modules/
    - dist/

# 安装依赖
install_deps:
  stage: install_deps
  only:
    - develop
    - master
  script:
    - npm install

# 运行测试用例
test:
  stage: test
  only:
    - develop
    - master
  script:
    - npm run test

# 编译
build:
  stage: build
  only:
    - develop
    - master
  script:
    - npm run clean
    - npm run build:client
    - npm run build:server

# 部署测试服务器
deploy_test:
  stage: deploy_test
  only:
    - develop
  script:
    - pm2 delete app || true
    - pm2 start app.js --name app

# 部署生产服务器
deploy_production:
  stage: deploy_production
  only:
    - master
  script:
    - bash scripts/deploy/deploy.sh
```

上面的配置把一次 Pipeline 分成五个阶段：

- 安装依赖(install_deps)
- 运行测试(test)
- 编译(build)
- 部署测试服务器(deploy_test)
- 部署生产服务器(deploy_production)

设置 Job.only 后，只有当 develop 分支和 master 分支有提交的时候才会触发相关的 Jobs。

节点说明：

- stages：定义构建阶段，这里只有一个阶段 deploy
- deploy：构建阶段 deploy 的详细配置也就是任务配置
- script：需要执行的 shell 脚本
- only：这里的 master 指在提交到 master 时执行
- tags：与注册 runner 时的 tag 匹配



