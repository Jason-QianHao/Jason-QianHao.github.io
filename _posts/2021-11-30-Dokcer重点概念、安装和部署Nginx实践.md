---
title: Dokcer重点概念、安装和部署Nginx实践
tags: Docker
---

# Dokcer重点概念、安装和部署Nginx实践

# Docker重点概念

Docker 包括三个基本概念:

- **镜像（Image）**：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。

- **容器（Container）**：镜像（Image）和容器（Container）的关系，就像是**面向对象程序设计中的类和实例**一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- **仓库（Repository）**：仓库可看成一个代码控制中心，用来保存镜像。

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。

Docker 容器通过 Docker 镜像来创建。

## 镜像

```
Docker 提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变

镜像的构建是分层的，不同于ios那样的打包系统，docker的镜像只是一个虚拟的概念，实际上它并不是由一个文件组成的而是由一组文件组成的。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。

在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉
```

镜像常见操作：

- docker search [镜像名称]：搜索Docker Hub(镜像仓库)上的镜像；
- docker pull [镜像名称]：从仓库中获取镜像；

- docker images：列出本地所有的镜像（非隐藏，可以加入-a参数显示所有)；
- docker build：通过 Dockerfile build 出镜像；
- docker commit：将容器中的所有改动生成新的镜像；
- docker history：查看镜像的历史；
- docker save：将镜像保存成 tar 包；
- docker import：通过 tar 包导入新的镜像；
- docker load：通过 tar 包或者标志输入导入镜像；
- docker rmi：删除本地镜像；
- docker tag：给镜像打 tag

## 容器

```
Docker 利用容器来运行应用。

容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

镜像（ Image ）和容器（ Container ）的关系，就像是面向对象程序设计中的 类 和 实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。

镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为"容器存储层"。容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 数据卷（Volume）、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。
```

容器常见操作：

- docker run [OPTIONS] IMAGE [COMMAND] [ARG...]：利用镜像创建并启动一个容器

  - 常用参数：

    ```shell
    --interactive 等同于 -i ，接受 stdin 的输入，（即使没有连接，也要保持STDIN打开）
    
    --tty 等同于 -t，分配一个 tty,也就是分配虚拟终端，一般和 i 一起使用；
    
    --name 给容器设置一个名字，如果没有指定将会随机产生一个名称
    
    -d, --detach       在后台运行容器并打印出容器ID 
    ```

  - demo

    ```shell
     docker run --name nginx-test -p 8080:80 -d nginx
    ```

- docker exec -it containerID /bin/bash：以bash方式进入容器

- docker container start/stop/restart：启动/停止/重启停止状态下的容器

- docker rm [容器名称]/[容器ID] ：删除已关闭的容器

- docker inspect [容器id]：查看容器全部信息

- docker container ls -a / docker ps -a：查看所有已经创建的包括终止状态的容器

## 仓库

```
镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry 就是这样的服务。一个 Docker Registry 中可以包含多个仓库（ Repository ）；每个仓库可以包含多个标签（ Tag ）

仓库分为公开仓库（Public）和私有仓库（Private）两种形式。

最大的公开仓库是 Docker Hub，存放了数量庞大的镜像供用户下载。

docker官方镜像仓库：https://hub.docker.com
```

仓库常见操作：

- docker login: 登录镜像仓库；
- docker logout: 登出镜像仓库；
- docker pull: 从镜像仓库拉取镜像 ；
- docker push: 向镜像仓库 push 镜像，需要先 login。

## 底层实现

Docker 底层的核心技术包括 Linux 上的命名空间（Namespaces）、控制组（Control groups）、Union 文件系统（Union file systems）和容器格式（Container format）。

我们知道，传统的虚拟机通过在宿主主机中运行 hypervisor 来模拟一整套完整的硬件环境提供给虚拟机的操作系统。虚拟机系统看到的环境是可限制的，也是彼此隔离的。 这种直接的做法实现了对资源最完整的封装，但很多时候往往意味着系统资源的浪费。 例如，以宿主机和虚拟机系统都为 Linux 系统为例，虚拟机中运行的应用其实可以利用宿主机系统中的运行环境。

我们知道，在操作系统中，包括内核、文件系统、网络、PID、UID、IPC、内存、硬盘、CPU 等等，所有的资源都是应用进程直接共享的。 要想实现虚拟化，除了要实现对内存、CPU、网络IO、硬盘IO、存储空间等的限制外，还要实现文件系统、网络、PID、UID、IPC等等的相互隔离。 前者相对容易实现一些，后者则需要宿主机系统的深入支持。

随着 Linux 系统对于命名空间功能的完善实现，程序员已经可以实现上面的所有需求，让某些进程在彼此隔离的命名空间中运行。大家虽然都共用一个内核和某些运行时环境（例如一些系统命令和系统库），但是彼此却看不到，都以为系统中只有自己的存在。这种机制就是容器（Container），利用命名空间来做权限的隔离控制，利用 cgroups 来做资源分配。

### 命名空间

Linux 内核中提供了 6 中隔离支持，分别是：IPC 隔离、网络隔离、挂载点隔离、进程编号隔离、用户和用户组隔离、主机名和域名隔离。

| Namespace | flag          | 隔离内容                                |
| :-------- | :------------ | :-------------------------------------- |
| IPC       | CLONE_NEWIPC  | IPC（信号量、消息队列和共享内存等）隔离 |
| Network   | CLONE_NEWNET  | 网络隔离（网络栈、端口等）              |
| Mount     | CLONE_NEWNS   | 挂载点（文件系统）                      |
| PID       | CLONE_NEWPID  | 进程编号                                |
| User      | CLONE_NEWUSER | 用户和用户组                            |
| UTS       | CLONE_NEWUTS  | 主机名和域名                            |

### 控制组

控制组是 Linux 内核的一个特性，主要用来对共享资源进行隔离、限制、审计等。只有能控制分配到容器的资源，才能避免当多个容器同时运行时的对系统资源的竞争。

CGroups 中有几个重要概念：

- **cgroup**：通过 CGroups 系统进行限制的一组进程。CGroups 中的资源限制都是以进程组为单位实现的，一个进程可以加入到某个进程组，从而受到相同的资源限制。
- **task**：在 CGroups 中，task 可以理解为一个进程。
- **hierarchy**：可以理解成层级关系，CGroups 的组织关系就是层级的形式，每个节点都是一个 cgroup。cgroup 可以有多个子节点，子节点默认继承父节点的属性。
- **subsystem**：更准确的表述应该是 ***resource controllers\***，也就是资源控制器，比如 cpu 子系统负责控制 cpu 时间的分配。子系统必须应用（attach）到一个 hierarchy 上才能起作用。

其中最核心的是 ***subsystem\***，CGroups 目前支持的 ***subsystem\*** 包括：

- **cpu**：限制进程的 cpu 使用率；
- **cpuacct**：统计 CGroups 中的进程的 cpu 使用情况；
- **cpuset**：为 CGroups 中的进程分配单独的 cpu 节点或者内存节点；
- **memory**：限制进程的内存使用；
- **devices**：可以控制进程能够访问哪些设备；
- **blkio**：限制进程的块设备 IO；
- **freezer**：挂起或者恢复 CGroups 中的进程；
- **net_cls**：标记进程的网络数据包，然后可以使用防火墙或者 tc 模块（traffic controller）控制该数据包。这个控制器只适用从该 cgroup 离开的网络包，不适用到达该 cgroup 的网络包；
- **ns**：将不同 CGroups 下面的进程应用不同的 namespace；
- **perf_event**：监控 CGroups 中的进程的 perf 事件（注：perf 是 Linux 系统中的性能调优工具）；
- **pids**：限制一个 cgroup 以及它的子节点中可以创建的进程数目；
- **rdma**：限制 cgroup 中可以使用的 RDMA 资源。

### 联合文件系统

- 联合文件系统是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。

- 联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

- Docker 目前支持的联合文件系统包括 `OverlayFS`, `AUFS`, `Btrfs`, `VFS`, `ZFS` 和 `Device Mapper`。

- `overlay2` 是目前 Docker 默认的存储驱动，以前则是 `aufs`

### 数据存储

想要在 Docker 容器停止之后创建的文件依旧存在，也就是将文件在宿主机上保存。那么有两种方式：**volumes**、**bind mounts**。

#### 数据卷volumes

```
数据卷 可以在容器之间共享和重用

对 数据卷 的修改会立马生效

对 数据卷 的更新，不会影响镜像

数据卷 默认会一直存在，即使容器被删除

Volumes 会把文件存储到宿主机的指定位置，在 Linux 系统上这个位置为 /var/lib/docker/volumes/。这些文件只能由 Docker 进程进行修改，是 docker 文件持久化的最好的方式。
```

- 启动挂载数据卷的容器

在用 `docker run` 命令的时候，使用 `--mount` 标记来将 `数据卷` 挂载到容器里。在一次 `docker run` 中可以挂载多个 `数据卷`。

例如创建一个名为 `web` 的容器，并加载一个名为`my-vol`的 `数据卷` 到容器的 `/usr/share/nginx/html` 目录。

```bash
$ docker run -d -P \
    --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine
```

在主机里使用以下命令可以查看 `web` 容器的信息

```bash
$ docker inspect web
```

`数据卷` 信息在 "Mounts" Key 下面

```json
"Mounts": [
    {
        "Type": "volume",
        "Name": "my-vol",
        "Source": "/var/lib/docker/volumes/my-vol/_data",
        "Destination": "/usr/share/nginx/html",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```

#### 挂载主机目录 bind mounts

使用 `--mount` 标记可以指定挂载一个本地主机的目录到容器中去。

Docker 挂载主机目录的默认权限是 `读写`，用户也可以通过增加 `readonly` 指定为 `只读`。

**bind mounts** 可以将文件存储到宿主机上面任意位置，而且别的应用程序也可以修改。

```bash
$ docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html \
    nginx:alpine
```

```json
"Mounts": [
    {
        "Type": "bind",
        "Source": "/src/webapp",
        "Destination": "/usr/share/nginx/html",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

## 命令大全

[docker命令大全](https://www.runoob.com/docker/docker-command-manual.html)

# Docker的Linux安装

1. **查看当前的内核版本**

Linux 内核：官方建议 3.10 以上，3.8以上貌似也可。

```javascript
uname -r
```

2. **使用 root 权限更新 yum 包**

```javascript
yum -y update
```

这个命令不是必须执行的，看个人情况，后面出现不兼容的情况的话就必须update了

```javascript
注意 
yum -y update：升级所有包同时也升级软件和系统内核； 
yum -y upgrade：只升级所有包，不升级软件和系统内核
```

3. **卸载旧版本（如果之前安装过的话）**

```javascript
yum remove docker  docker-common docker-selinux docker-engine
```

4. **安装需要的软件包， yum-util 提供yum-config-manager功能，另两个是devicemapper驱动依赖**

   ```javascript
   yum install -y yum-utils device-mapper-persistent-data lvm2
   ```

5. 查看可用版本有哪些

```javascript
yum list docker-ce --showduplicates | sort -r
```

6. 选择一个版本并安装：`yum install docker-ce-版本号`

   ```javascript
   yum -y install docker-ce-18.03.1.ce
   ```

7. 启动 Docker 并设置开机自启

   ```shell
   systemctl start docker
   systemctl enable docker
   ```

# 部署Nginx实践

1. 拉取nginx镜像

默认最新版本latest

```
docker pull nginx
```

2. 在宿主机中设置挂在目录

```
mkdir -p /data/nginx/{conf,conf.d,html,log}
```

![image-20211130162322421](/../assets/Docker/image-20211130162322421.png)

3. 配置文件放置挂在目录

```
#user www-data;
worker_processes auto;
#pid /run/nginx.pid;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;
        gzip_disable "msie6";

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;

        server{
                listen 80;
                server_name your_server_name;   #你的serverName
                root /usr/share/nginx/html;
                index index.html;
        }

}
```

4. 启动nginx容器

```shell
docker run 
--name my_nginx
-d -p 80:80  
-v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf 
-v /data/nginx/log:/var/log/nginx 
-v /data/nginx/html:/usr/share/nginx/html
nginx
```

第一个-v：挂载nginx的主配置文件，以方便在宿主机上直接修改容器的配置文件

第二个-v：挂载容器内nginx的日志，容器运行起来之后，可以直接在宿主机的这个目录中查看nginx日志

第三个-v：挂载静态页面目录

5. 效果如下：

![image-20211130163158525](/../assets/Docker/image-20211130163158525.png)

