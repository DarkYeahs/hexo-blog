---
title: docker初步学习
date: 2019-06-24 15:45:22
tags: Docker
categories: 笔记
---
#### 定义
Docker属于linux容器的一种封装，提供简单易用的容器使用接口。是目前最为流行的Linux容器解决方案。

Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。

总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。


#### 用途
Docker主要用途目前有三大类：
+ 提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
+ 提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
+ 组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

#### 安装
这里以centos 7环境安装为例，Docker分两个版本，一个是社区版本 Docker CE，一个是企业版本Docker EE（部分收费）。这里讲述的是社区版本的安装。

##### 要求
1. 需要开启centos-extras，centos 7是默认开启centos-extras的，但如果你之前手动关闭centos-extras，你需要开启centos-extras再进行安装。
2. 推荐使用overlay2存储驱动程序

##### 安装步骤
1. 先卸载旧版本的Docker。旧版本的Docker被称为docker或者docker-engine，需要先移除他们以及他们的依赖
```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
2. 安装Docker有三种方式，这里使用从仓库下载安装的方式。
```shell
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
    yum-utils提供yum-config-manager的命令，接下来使用yum-config-manager添加Docker的安装库
```shell
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
    然后在执行安装命令
```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io
```


3. 启动Docker
```shell
$ sudo systemctl start docker
```

4. Docker权限

    需要用户具有 sudo 权限，为了避免每次命令都输入sudo，可以把用户加入 Docker 用户组
    ```shell
    $ sudo groupadd docker
    $ sudo usermod -aG docker $USER
    ```

#### 运行
##### image文件
image是二进制文件，Docker根据Dockerfile文件将应用程序及依赖生成一个二进制文件，即image文件。然后Docker根据image文件生成容器实例，同一个image文件可以生成多个容器实例。
##### 容器文件
Docker根据image文件生成的容器实例，本身也是一个文件。容器实例生成的时候，会同时存在两个文件：容器文件以及image文件。并且关闭容器不会删除容器文件，只是容器停止运行而已。
##### 例子：hello world
从官方库拉取一个hello world镜像到本地
```shell
docker image pull library/hello-world
```
拉取成功后可以运行以下命令常看image文件
```shell
$ docker image ls
```
运行该image文件
```image
$ docker container run hello-world
```
docker container run命令会根据hello-world生成正在运行的容器实例（容器文件），运行成功后会有以下输出
```shell
Hello from Docker!
This message shows that your installation appears to be working correctly.
```
