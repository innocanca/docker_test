# Docker仓库

仓库（Repository）是集中存放镜像的地方

一个容易混淆的概念是注册服务器（Registry) 。实际上注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。从这个方面来说，仓库可以被认为是一个具体的项目或目录，例如对仓库地址docker.io/ubuntu来说，doceker.io是注册服务器地址，ubuntu是仓库名。

大部分时候，并不需要严格区分这两者概念

## 1. Docker Hub

目前Docker官方维护了一个公共仓库Docker Hub，其中包括了数量超过2650000的镜像，大部分需求都可以通过DOcker Hub中直接下载镜像实现 

### 1.1 注册

你可以在 https://hub.docker.com 免费注册一个 Docker 账号。

### 1.2 登陆

可以通过执行 docker login 命令交互式的输入用户名及密码来完成在命令行界面登录 Docker Hub。

你可以通过 docker logout 退出登录。

### 1.3 拉取镜像

你可以通过docker search 命令来查找**官方仓库**中的镜像，并利用 docker pull 命令来将它下载到本地 。

```
    $ docker search centos
    NAME                               DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
    centos                             The official build of CentOS.                   6449      [OK]
    ansible/centos7-ansible            Ansible on Centos7                              132                  [OK]
    consol/centos-xfce-vnc             Centos container with "headless" VNC session…   126                  [OK]
    jdeathe/centos-ssh                 OpenSSH / Supervisor / EPEL/IUS/SCL Repos - …   117                  [OK]
    centos/systemd                     systemd enabled base container.                 96                   [OK]
```

根据是否是官方提供，可将镜像分为两类。

一种是类似centos这样的镜像，被称为基础镜像或根镜像，这些基础镜像由Docker公司创建、验证、支持、提供。这样的镜像往往使用单哥单词作为名字

还有一种类型，比如ansible/centos7-ansible镜像，它是由Docker Hub的注册用户创建并维护的，往往带有用户名称前缀，可以通过前缀 **username/** 来制定使用某个用户提供的镜像

另外，查找的时候通过 --filter=stars=N 参数可以指定仅显示收藏数量为 N 以上的镜像。

下载镜像

```
    $ docker pull centos
    Using default tag: latest
    latest: Pulling from library/centos
    7a0437f04f83: Pull complete
    Digest: sha256:5528e8b1b1719d34604c87e11dcd1c0a20bedf46e83b5632cdeac91b8c04efc1
    Status: Downloaded newer image for centos:latest
    docker.io/library/centos:latest
```

### 1.4 推送镜像

用户也可以在 ***登陆后*** 通过docker push 命令来将自己的镜像推送到Docker Hub

以下命令中的 username 请替换为你的 Docker账号用户名

```
$ docker tag ubuntu:18.04 username/ubuntu:18.04

$ docker image ls

    REPOSITORY                                               TAG                    IMAGE ID            CREATED             SIZE
    ubuntu                                                   18.04                  275d79972a86        6 days ago          94.6MB
    username/ubuntu                                          18.04                  275d79972a86        6 days ago          94.6MB

    $ docker push username/ubuntu:18.04

    $ docker search username

    NAME                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
    username/ubuntu
```

### 1.5 自动构建（后续研究）

自动构建（Automated Builds）功能对于需要经常需要升级镜像的程序来说，十分方便

有时候，用户构建了镜像，安装了某个软件，当软件发布新版本则需要手动更新镜像。

而自动构建运行用户通过Docker Hub指定跟踪一个目标网站（支持 GitHub (opens new window)或 BitBucket (opens new window)）上的项目，一旦项目发生新的提交（commit）或者创建了新的标签（tag），Docker Hub会自动构建镜像并推送到Docker Hub中


## 2. 私有仓库（后续研究）

有时候使用 Docker Hub 这样的公共仓库可能不方便，用户可以创建一个本地仓库供私人使用。

docker-registry (opens new window)是官方提供的工具，可以用于构建私有的镜像仓库。本文内容基于 docker-registry (opens new window)v2.x 版本。