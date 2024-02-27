# docker相关

## 卸载docker

```bash
yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine
```

> No match for argument: docker
> No match for argument: docker-client
> No match for argument: docker-client-latest
> No match for argument: docker-common
> No match for argument: docker-latest
> No match for argument: docker-latest-logrotate
> No match for argument: docker-logrotate
> No match for argument: docker-engine
> No packages marked for removal.
> Dependencies resolved.
> Nothing to do.
> Complete!



## 配置Docker的yum库

首先要安装一个yum工具

```Bash
yum install -y yum-utils
```

> Package yum-utils-4.0.21-19.al8.noarch is already installed.
> Dependencies resolved.
> Nothing to do.
> Complete!



安装成功后，执行命令，配置Docker的yum源：

```Bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

> Adding repo from: https://download.docker.com/linux/centos/docker-ce.repo



## 安装docker

```Bash
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

> Installed:
>   containerd.io-1.6.28-3.1.el8.x86_64
>   docker-buildx-plugin-0.12.1-1.el8.x86_64
>   docker-ce-3:25.0.3-1.el8.x86_64
>   docker-ce-cli-1:25.0.3-1.el8.x86_64
>   docker-ce-rootless-extras-25.0.3-1.el8.x86_64
>   docker-compose-plugin-2.24.5-1.el8.x86_64
>   fuse-overlayfs-1.11-1.0.1.al8.x86_64
>   fuse3-3.3.0-16.al8.x86_64
>   fuse3-libs-3.3.0-16.al8.x86_64
>   libcgroup-0.41-19.2.al8.x86_64
>   libslirp-4.4.0-1.al8.x86_64
>   slirp4netns-1.2.0-2.al8.x86_64
>
> Complete!



## 查看版本

```bash
docker -v
```

> Docker version 24.0.7, build afdd53b



## 查看镜像

```bash
docker images
```

> error during connect: this error may indicate that the docker daemon is not running: Get "http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.24/images/json": open //./pipe/docker_engine: The system cannot find the file specified.

守护进程未启动

> REPOSITORY   TAG       IMAGE ID   CREATED   SIZE

正常启动



## 启动和校验

```Bash
# 启动Docker
systemctl start docker

# 停止Docker
systemctl stop docker

# 重启
systemctl restart docker

# 设置开机自启
systemctl enable docker

# 执行docker ps命令，如果不报错，说明安装启动成功
docker ps
```



## 设置开机自启

```bash
systemctl enable docker
```



## 配置阿里云镜像加速

https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://l95dqhhg.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

配置完后进行加载并重启



## docker run中的常见参数

以mysql安装为例子

```bash
docker run -d \
 --name mysql \
 -p 3306:3306 \
 -e TZ=Asia/Shanghai \
 -e MYSQL_ROOT_PASSWORD=123456 \
 mysql
```

**-d** 表示后台执行

**--name** 容器的名字

**-p** 宿主机和容器映射的端口

**-e** 环境变量的设置 此处设置了时区和密码

最下方的`mysql`指此镜像名字（官方名字）

一般此处的镜像名称结构是 `Repository:TAG` 镜像名+版本号

若像此处没有写上版本号 就默认是安装最新版本镜像的容器







## docker操作镜像常见命令

![image-20240222133348658](.\images\image-20240222133348658.png)

- docker pull 将镜像从远端拉取到本地
- docker push 与拉取对应的 将镜像从本地推送到远端

> 远端指的是dockerHub

- docker images 查看所有的镜像
- docker ps 查看所有的容器及其状态
- docker build 构建自己的镜像 通过Dockerfile
- docker save 保存镜像为本地文件 
- docker load 加载本地镜像

- docker rmi 移除镜像

- docker run 运行镜像 创建容器 如果本地找不到对应的镜像文件 会到远端去找
- docker logs 查看docker日志
- docker exec 运行docker容器
- docker stop 停止(暂停)运行中的镜像
- docker start 重新运行构建容器

> docker支持类似断点续传的功能 当构建容器的过程中意外暂停了传输 可以通过`docker start`重新运行起这个过程 并可以通过`docker ps`来查看容器的安装状态

补充：

```bash
docker ps --format "table {{ID}\t{{.Image}r\t{{.Ports}}\t{{.Status}]\t{{.Names}}"
```

在查看容器状态的时候 自定义容器的格式







## 数据卷相关

docker容器具有多个特点，是一个在最大程度上最小化依赖和相关组件的单位。这一特点使docker更加的轻量化，提高操作的效率。但同时也带来了某些不便，因为在开发中，我们通常需要修改对应容器的配置。例如修改mysql的端口，在通常情况下，我们需要通过vim等编辑器进入配置文件的内部进行手动修改。但是在docker包装的linux环境中，由于最小化依赖的要求，我们通常不能通过vi进入文件内部编辑。

试使用`docker exec` + `vi`

```bash
[root@localhost ~]# docker exec -it nginx bash

root@6f6dd1f6932d:/# cd /usr/share/nginx/html

root@6f6dd1f6932d:/usr/share/nginx/html# ls
50x.html  index.html

root@6f6dd1f6932d:/usr/share/nginx/html# vi index.html
bash: vi: command not found
```

所以为了解决日常开发中的这一类需求 docker引出了数据卷挂载的概念。顾名思义，就是在宿主机的建立对应的目录及文件，映射到容器当中去

![image-20240225164657328](.\images\image-20240225164657328.png)

宿主机和nginx中的对应目录是双向绑定的，当在宿主机的映射文件中做修改时，会同步到容器当中去。相反，在容器内部进行修改的时候，也会同步到宿主机的文件系统中去。

了解概念之后，学习如何操作数据卷

| 指令                  | 作用           |
| --------------------- | -------------- |
| docker volume create  | 创建数据卷     |
| docker volume ls      | 查看所有数据卷 |
| docker volume rm      | 删除指定数据卷 |
| docker volume inspect | 查看某个数据卷 |
| docker volume prune   | 清除数据卷     |

以下以nginx创建容器时手动挂载为例

nginx中存放静态资源的目录是`/usr/share/nginx/html`

确认挂载目录的写法是否正确

在执行docker run命令时，使用`-v 本地目录`：容器内目录可以完成本地目录挂载

本地目录必须以“”或"/"开头，如果直接以名称开头，会被识别为数据卷而非本地目录

> - `-v mysql:var/ib/mysql` 会被识别为一个数据卷叫mysql docker会默认存放到宿主机中数据卷的目录 如上nginx中的`/var/lib/docker/volumes/html/_data`
> - `-v./mysql:Nar/Iib/mysql`会被识别为当前目录下的mysql目录



1. 创建容器并确定挂载的目录及其名字

```bash
[root@localhost ~]# docker run -d --name nginx -p 80:80 -v html:/usr/share/nginx/html
c25c429f3e8ed1470bc27df6ffcdb41d224bfb19087ec66624e24d1c6f8c323d
```

创建容器并返回容器的id  可使用docker ps观察容器状态

2. 查看数据卷

```bash
[root@localhost ~]# docker volume ls
DRIVER    VOLUME NAME
local     html
```

3. 查看挂载的数据卷的详情

```bash
[root@localhost ~]# docker volume inspect html
[
    {
        "CreatedAt": "2024-02-25T17:00:17+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/html/_data",
        "Name": "html",
        "Options": null,
        "Scope": "local"
    }
]
```

`MountPoint`则是对应宿主机中的目录路径

可以在宿主机cd到对应的目录中

```bash
[root@localhost ~]# cd /var/lib/docker/volumes/html/_data

[root@localhost _data]# ls
50x.html  index.html
```

发现经典两文件 这时候可以通过宿主机上的编辑器对映射文件进行修改，或者是新增静态资源。操作完毕之后可以再次通过`docker exec`进入容器内部，会发现挂载目录下资源或文件已经修改完成





## 镜像结构

镜像本质上是一个包含应用程序及其运行所需的系统函数库，运行配置，运行环境等文件的单元。

docker会将不同层次的文件分为多个部分 他们就叫做**层(Layers)**

> 例如一个java应用的部署可以分为
>
> - 启动程序的脚本
> - 程序压缩而成的jar包
> - JRE相关以及环境变量
> - 启动需要依赖的系统函数库 如Linux中CentOS和Ubuntu 

最底层的层次叫做**基础镜像** 一般存放应用依赖的系统函数库、环境、配置、文件等。

最顶层的层次叫做**入口** 指的是镜像的运行入口 一般来说是启动程序的脚本和参数

分层带来的优势是：

- 便于后期对函数库等文件进行优化

- 若两个镜像具有相同的基础运行环境，docker不会再次重复安装 大幅度减少容器安装所需的体积

  

## 自定义镜像

### **Dockerfile**

可以理解成是一个媒介，通过写在Dockerfile中的每一条指令，来描述并告诉docker，说明构建镜像时所要进行的操作。

常见指令如下：

| 指令名字   | 指令作用                                    |
| ---------- | ------------------------------------------- |
| FROM       | 基础镜像                                    |
| ENV        | 设定环境变量                                |
| RUN        | 运行cmd命令                                 |
| COPY       | 拷贝文件 可以是拷贝jar包 可以是拷贝其他组件 |
| ENTRYPOINT | 设定程序入口                                |
| WORKDIR    | 指定工作目录                                |

例子

```dockerfile
#基础镜像
FROM openjdk:11.0-jre-buster I
#设定环境变量（时区）
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/STZ /etc/localtime &echo $TZ /etc/timezone
#指定工作目录为app包
WORKDIR /app
#拷贝jar包
COPY docker-demo.jar /app.jar
#入口
ENTRYPOINT ["java","-jar","/app.jar"]
```

ps：还有很多docker相关的命令在官网当中，见https://docs.docker.com/build/building/packaging/



在确定好DockerFile后 就可以在控制台根据此文件创建镜像了

```bash
docker build -t demo-jar:1.0 .
# 即
docker build -t repositories:tags path
```

`-t `指定镜像的名字和版本号，如果不写版本号则默认最新latest

`.` 指定Dockerfile和jar包的相对路径   `.`则代表两者同级



构建完成之后，就可以使用`docker images`查看构建好的镜像





## 容器网络互连

默认情况下，docker启动的时候会申请分配一个网段，之后在docker中启动的容器中，若有需要用到网络相关。会在docker的网段中分配一个ip地址，但是若对应容器关闭或启动顺序改变，其所被分配的ip地址可能会改变。为了解决这个问题，可以通过docker自带的命令进行手动配置。

| 命令                      | 说明             |
| ------------------------- | ---------------- |
| docker network create     | 创建一个网络     |
| docker network ls         | 查看所有网络     |
| docker network rm         | 删除指定网络     |
| docker network prune      | 清除未使用的网络 |
| docker network connect    | 使容器连接某网络 |
| docker network disconnect | 使容器断连某网络 |
| docker network inspect    | 查看详细信息     |

1. 创建网桥并查看其状态

```bash
[root@localhost ~]# docker network create demo
ea9ea5bb9488c62a01059e862791817db1108e1fecf43b911f4801f8c6942a20

[root@localhost ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
73a42a826849   bridge    bridge    local
ea9ea5bb9488   demo      bridge    local
ec3ae5e04b95   host      host      local
5dc302b468a6   none      null      local
```

创建完毕后可以通过`ip addr`查看修改

> [root@localhost ~]# ip addr
> 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>     link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>     inet 127.0.0.1/8 scope host lo
>        valid_lft forever preferred_lft forever
>     inet6 ::1/128 scope host
>        valid_lft forever preferred_lft forever
> 2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>     link/ether 00:0c:29:46:e0:92 brd ff:ff:ff:ff:ff:ff
>     inet 192.168.138.129/24 brd 192.168.138.255 scope global noprefixroute dynamic ens33
>        valid_lft 1566sec preferred_lft 1566sec
>     inet 192.168.138.128/24 brd 192.168.138.255 scope global secondary noprefixroute ens33
>        valid_lft forever preferred_lft forever
>     inet6 fe80::8b28:3594:bbe0:f565/64 scope link noprefixroute
>        valid_lft forever preferred_lft forever
> 3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
>     link/ether 02:42:f2:3c:ad:1e brd ff:ff:ff:ff:ff:ff
>     inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
>        valid_lft forever preferred_lft forever
> 4: **br-ea9ea5bb9488:** <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
>     link/ether 02:42:64:27:4d:c3 brd ff:ff:ff:ff:ff:ff
>     inet 172.18.0.1/16 brd 172.18.255.255 scope global br-ea9ea5bb9488
>        valid_lft forever preferred_lft forever

2. 可以发现第四个就是刚刚所创建的网桥，我们可以人为的让容器加入这个网段当中

```bash
[root@localhost ~]# docker network connect demo nginx
```

连接之后，可以通过`docker inspect + 容器`查看其状态

```bash
[root@localhost ~]# docker inspect nginx
```

在冒出来的参数最后一部分里面有容器的网络状态

> "Networks": {
>                 "bridge": {
>                     "IPAMConfig": null,
>                     "Links": null,
>                     "Aliases": null,
>                     "MacAddress": "02:42:ac:11:00:02",
>                     "NetworkID": "a2e419eeef6a4ffcaff19b2bbdf9abf1fb04057c73b6adff655a42c5a280e7c3",
>                     "EndpointID": "885515cd78013114e7a3010f6f6f406333ebb1bb852e11ff305441f4295cfa38",
>                     "Gateway": "172.17.0.1",
>                     "IPAddress": "172.17.0.2",
>                     "IPPrefixLen": 16,
>                     "IPv6Gateway": "",
>                     "GlobalIPv6Address": "",
>                     "GlobalIPv6PrefixLen": 0,
>                     "DriverOpts": null,
>                     "DNSNames": null
>                 },
>                 "**demo**": {
>                     "IPAMConfig": {},
>                     "Links": null,
>                     "Aliases": [
>                         "c25c429f3e8e"
>                     ],
>                     "MacAddress": "",
>                     "NetworkID": "",
>                     "EndpointID": "",
>                     "Gateway": "",
>                     "IPAddress": "",
>                     "IPPrefixLen": 0,
>                     "IPv6Gateway": "",
>                     "GlobalIPv6Address": "",
>                     "GlobalIPv6PrefixLen": 0,
>                     "DriverOpts": {},
>                     "DNSNames": [
>                         "nginx",
>                         "c25c429f3e8e"
>                     ]
>                 }

可以看到第一个`bridge`是默认分配的网卡，第二个是刚刚手动加的，我们也可以使用`docker disconnect`命令来取消与默认网卡的连接。

除了这一种手动分配网卡的方式，我们也可以在创建容器的时候就指定网卡，在这种情况下，容器只与指定的网卡连接，不与系统默认分配的连接。格式为`--network networkName`

```bash
docker run -d 
--name mySQL \
-p 8080:8080 \
--network demo \   #此处指定
repositoriyName
```

