---
layout: post
title: Docker笔记
date: 2017-11-04 23:51:38 +08:00
author: "izumo"
tags:
    - Docker
    - Linux
---

>> 本文章目前为未完成状态，随时会加入新内容。

## Docker命令

### run

大部分run（不仅限于run）命令本身都会返回容器的id。

使用指定镜像创建并启动容器

    docker run abc/xxx[:tag]

后台运行

    docker run -d

设置容器名字

    docker run --name xxx

交互式容器，`-i`保持标准输入对容器开放，`-t`为容器分配虚拟终端（使用`Ctrl+P`和`Q`分离终端）。

    docker run -it

依赖其它容器（容器名:容器内部别名）

    docker run --link xxx:yyy

运行容器并执行命令（举例）

    docker run -d --name hello busybox:latest /bin/sh -c "echo 'hello world'"

创建只读容器

    docker run --read-only

为容器添加环境变量

    docker run -e e1=aaa -e e2= bbb

为容器添加挂载点（内部路径 或 主机路径:内部路径）（查看[存储卷](#ccj)）

    docker run -v /path/to/dir1 -v /path/to/dir2
    docker tun -v /path/to/dir/in/host:/path/to/dir/in/container

为容器添加开放端口（同上）

    docker run -p 80
    docker run -p 8080:80

设定容器入口（相当于 `cat command`）

    docker run --entrypoint=“cat” xxx command

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

    docker inspect --format "{{json .State.Running}}" xxx

### port

查看容器端口映射

    docker port xxx

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

        docker run --export 1111 -P

### Joined容器

    docker run --name aaa //被连接的容器
    docker run --net container:aaa --name bbb 

### Open容器

    docker run --net host

