---
layout: post
title: Docker笔记
date: 2017-11-04 23:51:38 +08:00
author: "izumo"
tags:
    - Docker
    - Linux
---

> 本文章目前为未完成状态，随时会加入新内容。

## Docker命令

### run

大部分run（不仅限于run）命令本身都会返回容器的id。

使用指定镜像创建并启动容器

    docker run abc/xxx[:tag]

后台运行

    docker run -d

设置容器名字

    docker run --name xxx

交互式容器，`-i`保持标准输入对容器开放，`-t`为容器分配虚拟终端（使用`Ctrl+P`和`Ctrl+Q`分离终端）。

    docker run -it

依赖其它容器（容器名:容器内部别名）（查看[跨容器依赖](#krqyl)）

    docker run --link xxx:yyy

运行容器并执行命令（举例）

    docker run -d --name hello busybox:latest /bin/sh -c "echo 'hello world'"

创建只读容器

    docker run --read-only

为容器添加环境变量

    docker run -e e1=aaa -e e2=bbb

为容器添加挂载点（内部路径 或 主机路径:内部路径）（查看[存储卷](#ccj)）

    docker run -v /path/to/dir1 -v /path/to/dir2
    docker tun -v /path/to/dir/in/host:/path/to/dir/in/container

为容器添加开放端口（同上）

    docker run -p 80
    docker run -p 8080:80

设定容器入口（相当于 `cat command`）

    docker run --entrypoint="cat" xxx command

容器退出后自动删除

    docker --rm

### create

创建（不启动）容器，返回创建容器的cid

    docker create xxx
    
    CID=$(docker create xxx)
    echo $CID

创建容器，并将容器cid输出到文件中

    docker create --cidfile /path/to/file.cid xxx

### ps

列出运行中的容器

    docker ps

列出所有容器

    docker ps -a

获得最后创建的容器cid

    CID=$(docker ps -l -q)
    echo $CID

### start

启动容器（容器必须已经创建）

    docker start xxx

### restart

重启容器

    docker restart xxx

### stop

关闭容器

    docker stop xxx

### logs

查看容器运行日志

    docker logs xxx

### exec

让已启动的容器执行命令

    docker exec container command

### top

查看容器内部运行的进程

docker top xxx

### rm

删除容器

    docker rm xxx

删除容器，同时删除容器挂载点（查看[存储卷](#ccj)）

    docker rm -vf xxx

### rmi

删除镜像（该镜像不存在实例）

    docker rmi xxx

### images

列出本地镜像

    docker images

### pull

拉取镜像

    docker pull abc/xxx[:tag]

### search

从Docker Hub搜索镜像

    docker search xxx

### save

保存镜像（注意不是容器）（字面意思，不会丢失分层信息）（不使用`-o`，生成的文件将输出到终端）

    docker save -o xx.tar xxx

### load

加载镜像（注意不是容器）（不使用`-i`，则从终端加载）

    docker load -i xx.tar

### inspect

查看容器元数据（大括号内为查询表达式，类似jQuery）

{% raw %}

    docker inspect --format "{{json .State.Running}}" xxx

{% endraw %}

### port

查看容器端口映射

    docker port xxx

### commit

提交新镜像（由已存在容器生成新镜像）（查看[手动构建镜像](#sdgjxjx)）

    docker commit [-a "@author" -m "message"] src_container new_image

### diff

查看文件系统的改动（查看[手动构建镜像](#sdgjxjx)）

    docker diff xxx

### tag

为镜像添加标签（默认的标签为`latest`）（镜像名称可以不同）

    docker tag aaa:tag1 aaa:tag2
    docker tag aaa:tag1 bbb:tag2
    docker tag aaa:tag1 aaa[:latest]

### cp

主机与容器间文件的复制（将主机路径替换为`-`，可以将文件输出到终端）

    docker cp /file/in/host container_name:/file/in/container
    docker cp container_name:/file/in/container /file/in/host

### export

将扁平的文件系统的所有内容导出到标准输出或一个压缩文件上（查看[AUFS](#aufs)）

    docker export -o yyy.tar xxx
    docker export xxx 

### import

将压缩文件的内容导入到一个新镜像中（新镜像只有一层）（查看[AUFS](#aufs)）

        docker import [-c "<Dockerfile instruction>"] yyy.tar xxx
        docker import [-c "<Dockerfile instruction>"] - xxx < yyy.tar

### build

从Dockerfile生成镜像（文件名默认为`Dockerfile`）（使用`-f`可以设置Dockerfile文件的名字）（查看[Dockerfile](#dockerfile)）

    docker build -t xxx[:tag] /path/to/dockerfile
    docker build -t xxx[:tag] -f dockerfilename.df /path/to/dockerfile

不使用缓存

    docker build --no-cache

<span id="ccj"></span>

## 存储卷

存储卷用于将容器内部的路径映射到主机上，使得这些路径内的文件可以存储在主机上。

### 存储卷类型

#### 绑定挂载存储卷

将容器内路径直接和主机上的特定路径进行绑定，适合将容器内数据分享给容器外的进程，或者将容器外数据分享给容器内应用。（可以用于容器间共享数据，将主机路径设为相同即可，但不是非常推荐）（必须使用绝对路径）（可以绑定单个文件，文件必须存在，否则Docker会认为你想绑定目录）

    docker tun -v /path/to/dir/in/host:/path/to/dir/in/container

在路径末尾添加`:ro` 使得路径对容器只读（容器不可修改路径内文件）

    docker tun -v /path/to/dir/in/host:/path/to/dir/in/container:ro


#### Docker管理存储卷

指定容器内需要绑定到主机的路径，Docker将自动为该路径在主机系统中创建目录（一般是在主机的`/var/lib/docker/vfs/dir/container_cid`下）

    docker run -v /path/to/dir

### 共享存储卷

将不同容器的存储卷路径绑定到相同的主机路径（不推荐）

    docker run -v /path/to/dir/in/host:/path/to/dir/in/container1 xxx1
    docker run -v /path/to/dir/in/host:/path/to/dir/in/container2 xxx2

使用`volume-from`

复制容器的卷定义

    docker run --volumes-from other_container 

1. 卷路径同时对源容器和目标容器可见；
2. 使用`docker rm -vf`删除目标容器时，不会删除源容器的卷定义；
3. 不能和`-v`共存，即不能添加新的绑定（暂时存疑）；
4. 可以复制多个源容器卷定义，但源容器内的路径不能重复；
5. 容器的访问权限（如`:ro`）也会被复制。

### 存储卷的清理

删除容器同时删除存储卷（`-f`为强制删除运行中的容器，先关闭，再删除）

    docker rm -v xxx
    docker rm -vf xxx

1. 删除容器绑定的所有存储卷；
2. 任何有其它容器绑定的存储卷将被忽略，但引用计数器递减。

## 网络访问

### Closed容器

    docker run --net none

1. 仅提供回环接口；
2. 容器内部不能访问容器外的网络；
3. 容器外（包括主机和其它容器）进程不能访问容器内部。

### Bridged容器

    docker run --net bridge
    docker run （省略）

1. Docker网络的默认选项；
2. 容器内部可以访问容器外的网络；
3. 如果不使用`-p`开放容器端口，则容器外（包括主机和其它容器）进程不能访问容器内部。

自定义命名解析，会将容器的ip地址映射到该命名，该命名只有容器内部可见

    docker run --hostname yyy

自定义DNS服务器

    docker run --dns a.b.c.d

为容器添加hosts（添加至`/etc/hosts`）

    docker run --add-host host_name_1:a.b.c.d --add-host host_name_2:e.f.g.h

#### 开放容器端口

1. 将容器指定端口（1234）绑定到所有主机（0.0.0.0）的一个动态（随机）端口上：

        docker run -p 1234
    
2. 将容器指定端口（1234）绑定到所有主机（0.0.0.0）的一个指定端口（5678）上：

        docker run -p 5678:1234

3. 将容器指定端口（1234）绑定到指定主机（192.168.0.2）的一个动态（随机）端口上：

        docker run -p 192.168.0.2::1234

4. 将容器指定端口（1234）绑定到指定主机（192.168.0.2）的一个指定端口（5678）上：

        docker run -p 192.168.0.2:5678:1234

5. 将容器的所有**开放**端口暴露出去，效果同（1）：

        docker run -P

6. 设置想要开放的端口，一般和`-P`命令配合使用，常用于`-P`选项没有开放想要的端口的情况：

        docker run --expose 1111-1115
        docker run --expose 1111 -P

### Joined容器

    docker run --name aaa //被连接的容器
    docker run --net container:aaa --name bbb 

两个容器共享它们开放的端口。适合容器想要互相通信，但不想赋予对方对自己资源的直接访问权的情况。

### Open容器

    docker run --net host

无任何网络隔离，容器对主机网络有完全的访问权，不建议使用。

<span id="krqyl"></span>

### 跨容器依赖

将新容器和另外一个容器相链接

    docker run --link aaa:aab bbb

1. aaa为被链接容器，bbb为新容器，aab为容器aaa在容器bbb内的别名（映射为容器aaa的ip）；
2. 如果跨容器通信被禁止，Docker会添加特定的防火墙规则来允许被链接容器间的通信；
3. 链接是单向的，即容器aaa没有容器bbb的链接信息；
4. 链接不具有传递性，即链接了容器bbb的容器ccc没有容器aaa的链接信息。

## 隔离与资源分配

### 内存

限制容器中进程能够使用的内存大小（单位可用b、k、m、g）

    docker run --memory 256m

### CPU

指定容器的**相对**权重（数值），只有在CPU时间上存在竞争时才会起作用

    docker run --cpu-shares 1024

限制容器只能在指定的CPU核集合中执行，以减少上下文切换

    docker run --cpuset-cpus 0
    docker run --cpuset-cpus 0, 1, 2
    docker run --cpuset-cpus 0-2

### 设备

挂载设备至容器（摄像头，光驱等）（主机路径:容器内路径）

    docker run --device /dev/video0:/dev/video0

### 共享内存

跨进程通信（IPC），容器bbb可以访问容器aaa相同的内存位置

    docker --ipc container:aaa bbb

### 开放内存

将容器和主机运行在同一个命名空间（一般情况下不推荐使用）

    docker run --ipc host

### 指定用户（run-as）

指定容器使用的用户

    docker -u uname
    docker -u uid:gid

1. uname必须在容器内存在
2. uid、gid在容器内可以不存在
3. 容器内的uname和主机中的同名uname具有相同的权限（主要影响卷`-v`）。除非想要主机的文件能够被容器访问，否则不要将文件以卷的形式挂载到容器上。

### 特权容器

    docker run --privileged

<span id="sdgjxjx"></span>

## 手动构建镜像

创建新镜像（默认的标签为`latest`）

    docker commit [-a "@author" -m "message"] src_container new_image[:tag]

以下内容会被记录进新镜像：

+ 所有的环境变量
+ 工作目录
+ 开放端口集合
+ 所有卷的定义
+ 容器入口点
+ 命令和参数

查看文件系统的改动（A：添加，C：修改，D：删除）

    docker diff xxx

<span id="aufs"></span>

## AUFS（文件系统）

+ AUFS由多个层组成。每当对AUFS改动一次，改动会被记录到一个新的层中，这个新层放置于所有层的最上面。容器（和用户）访问文件系统所看到的，就是所有这些层的“联合”，或者说是自上而下的观察角度。
+ 当你从AUFS读取一个文件时，系统会从存在该文件的、最上面的一层中读取。如果文件没有在最顶层被创建或改动，那么读取操作就会沿着层不断向下找，知道找到这个文件的层。
+ 使用`diff`命令可以查看容器文件相对于源镜像的改动情况。
+ 使用`commit`命令会在源镜像层结构的基础上再添加一层（在使用`commit`命令之前所进行的所有操作，都只算作一层）。
+ Dockerfile的每一次RUN操作，都会添加一层（因此可以将多个RUN合成一个RUN以减少镜像的层数）。
+ AUFS具有层数限制，一般为42。
+ 可以通过导出镜像，再导入镜像来获得扁平镜像，减小镜像的大小。但**不推荐**，因为会丢失改动的历史信息，更好的做法是创建分支。

### 导出和导入扁平文件系统

将扁平的文件系统的所有内容导出到标准输出或一个压缩文件上

    docker export -o yyy.tar xxx
    docker export xxx 

将压缩文件的内容导入到一个新镜像中（新镜像只有一层）

    docker import [-c "<Dockerfile instruction>"] yyy.tar xxx
    docker import [-c "<Dockerfile instruction>"] - xxx < yyy.tar 

<span id="dockerfile"></span>

## Dockerfile

几乎所有的Dockerfile命令都会使镜像添加一层，因此同一个类型的命令应尽可能的少出现。

### FROM

设置基础镜像（一般情况下，所有的Dockerfile均以`FROM`开始）

    FROM ubuntu:latest

### MAINTAINER

设置镜像维护者的名字和邮箱

    MAINTAINER user name here "username@email.com"

### RUN

运行命令（每个`RUN`命令都会使文件系统增加一层）

    RUN apt install -y git
    RUN apt purge git

优化方法

    RUN apt install -y git && apt purge git

### ENTRYPOINT

容器入口（等同于`--entrypoint`）

    ENTRYPOINT ["git"]

+ 一般Dockerfile只有一个`ENTRYPOINT`,如果设置多个，只执行最后一个
+ `ENTRYPOINT`命令有两种格式：shell格式和exec格式
- shell格式类似于shell命令，其中的参数以空格为边界。如：`ENTRYPOINT exec ./mailer.sh`，相当于`/bin/sh -c 'exec ./mailer.sh'`。
- exec格式是一个字符串数组，其中第一个字符串是要执行的命令，剩余的字符串是参数。如：`ENTRYPOINT ["ls", "-l"]`。
+ 使用shell格式时，`CMD`命令的所有参数以及`docker run`指定的额外参数将会被忽略。

### CMD

设置默认命令，在生成容器时执行（构建镜像时不执行）

+ 如果执行`docker run xxx`时没有提供命令，则默认执行`CMD`命令
+ 一般和`ENTRYPOINT`配合使用，此时`ENTRYPOINT`保留命令，`CMD`保留参数。如`ENTRYPOINT ["/bin/bash"]`，以下

    CMD ["echo", "hello world"]
    CMD "echo" "hello world"

### ENV

为容器添加环境变量（相当于`docker run -e`）

    ENV APPPORT="/app" \
        APP="mailer.sh" \
        VERSION="0.6"

+ 添加的环境变量对Dockerfile文件本身也有效
+ 每个`ENV`命令都会使文件系统增加一层
+ 可以使用多个`ENV`，但这样会创建更多的层

### ADD

从主机复制文件到容器

    ADD ["/path/to/dir/in/host" "/path/to/dir/in/container"]

+ 数组最后一个元素为容器目录，之前的元素为主机目录
+ 复制到容器的文件和文件夹权限为755，uid和gid为0（即root）
+ 如果文件为压缩格式（tar、gzip、bzip2等），则docker会自动解压缩（URL下载不能使用解压缩特性，会直接复制压缩包）
+ 主机文件必须在`docker build <PATH>`的`<PATH>`下
+ 要复制的主机文件和容器的目的文件名字不同时，将文件改名
+ 容器的目的为路径时（以`/`结尾），将主机文件复制到容器的指定路径下
+ 由于`ADD`命令会造成**歧义**（不知道需不需要解压缩），因此大多数情况下应使用`COPY`命令替代

### COPY

从主机复制文件到容器

    COPY ["/path/to/dir/in/host/1", "/path/to/dir/in/host/2", "/path/to/dir/in/container"]

+ 数组最后一个元素为容器目录，之前的元素为主机目录
+ 不会执行解压缩
+ 支持shell模式和exec模式，但如果参数包含空格，则必须使用exec模式（上面的例子即为exec模式）

### LABEL

定义键值对，这些键值对被记录为镜像或者容器的额外元数据（相当于`docker run --label`）（此处使用了`ENV`命令添加的环境变量）

    LABEL base.name="App Name" \
          base.version="${VERSION}"

### WORKDIR

设置容器的工作路径（等同于`docker run -w`），对`RUN`、`CMD`、`ENTRYPOINT`命令生效

    WORKDIR /path/to/workdir

### USER

设置用户（等同于`docker run -u`），对接下来的Dockerfile命令生效

    USER uname[:gname]
    USER uid[:gid]

### EXPOSE

设置对外开放端口（等同于`docker run --expose`）

    EXPOSE 80 8080

+ 执行`docker run -P`命令，会将`EXPOSE`命令列出的所有端口开放
+ 不能使用端口范围作为参数，如：2000-3000

### VOLUME

添加挂载点（等同于`docker run -v`）

    VOLUME ["/var/log", "/var/www/html"]

+ 无法创建只读卷（即`:ro`）
+ 无法创建绑定-挂载卷（即`/path/to/dir/in/host:/path/to/dir/in/container`）

### ONBUILD

    ONBUILD COPY [".", "/var/myapp"]
    ONBUILD RUN ls -l /var/myapp

+ 如果生成的镜像被作为另一个构建的基础镜像，则`ONBUILD`命令定义了需要被执行的命令
+ 这些命令不会在包含它们的Dockerfile被构建时被执行
+ `ONBUILD`命令不具有传递性，即镜像A的`ONBUILD`可以被基于镜像A的镜像B执行，但不会被基于镜像B的镜像C执行


