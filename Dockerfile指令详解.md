
## 1. Dockerfile指令

## 2. FROM指定基础镜像

定制镜像：以一个镜像为基础，在其上进行定制

基础镜像必须指定，**FROM**就是指定基础镜像，因此一个Dockerfile中FROM是必备的指令，并且***必须是第一条指令***

基础镜像：
+ nginx
+ redis
+ mongo
+ mysql
+ httpd
+ tomcat
+ python
+ golang
+ ubuntu
+ centos

Docker提供一个特殊的镜像，名为**scratch** ,这个镜像是虚拟的概念，空白镜像

```
对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 FROM scratch 会让镜像体积更加小巧，使用 Go 语言 (opens new window)开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一
```

## 3. RUN执行命令

Run指令是用来执行命令的，由于命令行的强大能力，RUN指令在定制镜像时是最常用的指令之一。

格式：
+ shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 RUN 指令就是这种格式。
```
    RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

+ exec格式：RUN【"可执行文件","参数1","参数2"】，这更像是函数调用中的格式

Dockerfile中每一个指令都会建立一层，RUN也不例外，每一个RUN的行为，就和手工构建镜像过程一样，新建立一层，在其上执行这些命令，执行结束后，commit这层的修改，构成新的镜像。

Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。


## 4. 镜像构建上下文

```
    Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 Docker Remote API (opens new window)，而如 docker 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 docker 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举
```

docker build 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？

这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

## 5. COPY复制文件
格式
+ COPY [--chown=<user>:<group>] <源路径>... <目标路径>
+ COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]

COPY 指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置。比如：
```
    COPY package.json /usr/src/app/
```

<源路径> 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 filepath.Match (opens new window)规则
```
    COPY hom* /mydir/
    COPY hom?.txt /mydir/
```

<目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。***目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录***。

在使用该指令的时候还可以加上 --chown=<user>:<group> 选项来改变文件的所属用户及所属组。

如果源路径为文件夹，复制的时候不是直接复制该文件夹，而是将文件夹中的内容复制到目标路径。(疑问：复制到目标路径的文件内容是原目录结构吗？)


## 6. ADD更高级的复制文件

所有的文件复制均使用 COPY 指令，仅在需要自动解压缩的场合使用 ADD。

在某些情况下，这个自动解压缩的功能非常有用，比如官方镜像 ubuntu 中：
```
    FROM scratch
    ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
    ...
```

## 7. CMD容器启动命令（不理解，后续研究）

+ shell格式：CMD命令
+ exec格式：CMD ["可执行文件", "参数1", "参数2"...]

Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令的。

比如，ubuntu 镜像默认的 CMD 是 /bin/bash，如果我们直接 docker run -it ubuntu 的话，会直接进入 bash。我们也可以在运行时指定运行别的命令，如 docker run -it ubuntu cat /etc/os-release。这就是用 cat /etc/os-release 命令替换了默认的 /bin/bash 命令了，输出了系统版本信息。

在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 "，而不要使用单引号。

## 8. ENTRYPOINT 入口点（不理解，后续研究）

## 9. ENV 设置环境变量

+ ENV <key> <value>
+ ENV <key1>=<value1> <key2>=<value2>...

无论是后面的其它指令，如 RUN，还是运行时的应用，都可以直接使用这里定义的环境变量。

```
    ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```
```
    ENV NODE_VERSION 7.2.0

    RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
    && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
    && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```

## 10. ARG 构建参数

格式：ARG <参数名>[=<默认值>]

构建参数和 ENV 的效果一样，都是设置环境变量。所不同的是，ARG 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 ARG 保存密码之类的信息，因为 docker history 还是可以看到所有值的。

ARG 指令有生效范围，如果在 FROM 指令之前指定，那么只能用于 FROM 指令中。

```
    ARG DOCKER_USERNAME=library

    FROM ${DOCKER_USERNAME}/alpine

    RUN set -x ; echo ${DOCKER_USERNAME}
```

使用上述 Dockerfile 会发现无法输出 ${DOCKER_USERNAME} 变量的值，要想正常输出，你必须在 FROM 之后再次指定 ARG

```
    # 只在 FROM 中生效
    ARG DOCKER_USERNAME=library

    FROM ${DOCKER_USERNAME}/alpine

    # 要想在 FROM 之后使用，必须再次指定
    ARG DOCKER_USERNAME=library

    RUN set -x ; echo ${DOCKER_USERNAME}

```

## 11. VOLUME 定义匿名卷(后续研究)

+ VOLUME ["<路径1>", "<路径2>"...]
+ VOLUME <路径>

容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中.，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

```
    VOLUME /data
```

这里的 /data 目录就会在容器运行时自动挂载为匿名卷，任何向 /data 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行容器时可以覆盖这个挂载设置。比如
```
    $ docker run -d -v mydata:/data xxxx
```
在这行命令中，就使用了 mydata 这个命名卷挂载到了 /data 这个位置，替代了 Dockerfile 中定义的匿名卷的挂载配置。

## 12. EXPOSE 声明端口

+ EXPOSE <端口1> [<端口2>...]

EXPOSE 指令是声明容器运行时提供服务的端口，这只是一个声明，在容器运行时并不会因为这个声明应用就会开启这个端口的服务。

声明好处：
+ 在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；
+ 另一个用处则是在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

要将 EXPOSE 和在运行时使用 -p <宿主端口>:<容器端口> 区分开来。-p，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

## 13. WORKDIR 指定工作目录

格式为 WORKDIR <工作目录路径>。

使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，WORKDIR 会帮你建立目录。

⚠️ 之前提到一些初学者常犯的错误是把 Dockerfile 等同于 Shell 脚本来书写，这种错误的理解还可能会导致出现下面这样的错误：

```
    RUN cd /app
    RUN echo "hello" > world.txt
```
如果将这个 Dockerfile 进行构建镜像运行后，会发现找不到 /app/world.txt 文件，或者其内容不是 hello。

在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；而在 Dockerfile 中，这两行 RUN 命令的执行环境根本不同，是两个完全不同的容器。这就是对 Dockerfile 构建分层存储的概念不了解所导致的错误。

之前说过每一个 RUN 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 RUN cd /app 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

因此如果需要改变以后各层的工作目录的位置，那么应该使用 WORKDIR 指令。

```
    WORKDIR /app

    RUN echo "hello" > world.txt
```

如果你的 WORKDIR 指令使用的相对路径，那么所切换的路径与之前的 WORKDIR 有关：

```
    WORKDIR /a
    WORKDIR b
    WORKDIR c

    RUN pwd
```