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