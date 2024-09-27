Nginx & Docker & 反向代理

Q：有两个容器，分别为onlinejudgedeploy-backend-1  & main-nginx 简称oj 和 nginx 

现在需要两个容器。由于oj镜像已默认设置了流量代理在80端口。而nginx也是默认的监听80端口

所以这两个服务本来是不能同时运行的

现在要求挂载域名在oj网站上



所以：1、修改oj默认绑定的端口，在docker-compose.yml文件中进行修改

​			  2、创建network，绑定两个容器

（docker中每一个容器之间的网络都是独立的 也就是说如果我在作为容器的nginx中使用了127.0.0.1 访问的是此容器本身——而非宿主机本身）

首先使用

docker network create ojdy

创建网络

然后绑定两者

docker network connect oj

docker network connect nginx

按理来说 这个应该是可行的

如果成功的话 我们可以使用docker network inspect ojdy

此处注意！！！这一命令只能列举出成功运行启动的容器列表 ，如果oj或nginx两个镜像其中之一本身在启动的时候就已经出现错误

那么在containers这里是看不到他的 （docker ps -a 查看启动情况）

![image-20240918223255298](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240918223255298.png)

言归正传

在containers一栏关注是否有对应的两个容器

![image-20240918223135802](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240918223135802.png)

如果如上 那就是网络设置成功了

但是我们还没有设置nginx中的网络请求

vim nginx.conf

如下是一个最简单的设置

    server{
        listen  80;
        server_name  oj.jdygpt.com;
    
        location / {
                proxy_pass http://${ur_server_ip}:18000/;
                proxy_set_header Host $http_host;
        }
    }
上方的serverIP正如之前所说 写127.0.0.1或localhost都是不合法的 所以需要把服务器地址写上去

此处也可以把另外一个容器的名字写上去 连接两者 

（按理来说这才是标准做法 但是我无论如何都成功不了 nginx希纳是找不到上有服务器的地址）

所以我就写了IP 在此基础上须在阿里云等平台配置对应的域名。

![image-20240918225449840](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240918225449840.png)

但是按理来说我猜写IP是比写容器名更慢一点的  毕竟是内网请求和广域网请求的区别

注意，如果nginx无法启动，可以通过docker inspect nginx查看他所属的网络信息

但是无法通过docker network inspect ojdy进行查看 这个其实也很好理解 （只能查看进行中进程）



还有一个问题 我需要在阿里云中开对应端口的安全组才可以访问到对应的网站。

但是我的请求是先打到nginx 然后再打到后端服务器上的 而nginx到服务器这一段可以说是内网传播

所以为什么要开安全组？

![image-20240918225634826](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240918225634826.png)

A：阿里云中的安全组不仅限定外网能否访问 同时也对内网做出了限制。

![image-20240918225738431](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240918225738431.png)

但是再阿里云中将几个内网的子网全都全都圈上了 还是不行 开摆开摆