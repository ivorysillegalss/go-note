





![image-20240123000420607](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123000420607.png)







![image-20240123000411884](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123000411884.png)



![image-20240123001502119](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123001502119.png)



![image-20240123002053763](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123002053763.png)





![image-20240123002138368](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123002138368.png)



![image-20240123002409055](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123002409055.png)





![image-20240123002717848](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123002717848.png)





![image-20240123002713335](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123002713335.png)



![image-20240123003408460](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123003408460.png)





![image-20240123003558553](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123003558553.png)





这里修改的Remote host如果写错了 可以左方edit session进行修改 

但是切记修改之后重启才能生效 不要一直按R retry重试

![image-20240123004330754](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123004330754.png)









![image-20240123004255764](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123004255764.png)





![image-20240123004519416](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123004519416.png)







![image-20240123011228491](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123011228491.png)

![image-20240123011342137](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123011342137.png)



![image-20240123011353888](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123011353888.png)





修改前：

![image-20240123012105524](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123012105524.png)





修改后：

![image-20240123012354450](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123012354450.png)



![image-20240123150534238](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123150534238.png)













**云服务器**







![image-20240123164239263](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123164239263.png)











目的：想要在云服务器上访问openAI的api服务器

方法：通过github上的**clash-for-linux**项目 在云服务器上部署clash 实现代理

进度：成功部署项目 并且ping部分墙外ip也有响应

(查询项目的配置文件和GUI页面 在linux中ping测试 只有规则内的网站才有响应)

这里很奇怪 因为linux下的yaml配置文件按理来说和windows的是一样的 因为原网址是一样的 

但是我localhost是可以正常使用**谷歌openAI**一类的网站的 这些网站在配置文件的规则中通通没有

那到底是在什么情况下 可以成功访问墙外网站呢？在ping不通的情况下 我又如何知道我linux的云服务器是否可以访问这一类网站？

下方是clash官方的GUI界面 clashbord

http://47.113.231.211:9090/ui/#/rules

![image-20240123164229225](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240123164229225.png)

尝试进入配置文件 修改增加规则

增加了 **api.openai.com** 和 **google.com** 

但是不出所料 仍然ping不通 



想到的测试方法 往服务器上部署一个简易测试墙外网站api的程序 并测试是否能运行成功

说到底 如果不是以规则为基准 到底是怎么访问到外网的？





项目目的是想要调取api服务器 如果不借助clash的话 是否还有其他方法达到目的？









如何知道自己是否配置成功，连接上外网？

使用curl 外网网站

```bash
[root@iZf8zbjkf8rufzntdamo8sZ clash-for-linux]# curl -I https://www.google.com
```

> **成功时的响应：**
>
> HTTP/1.1 200 Connection established
>
> HTTP/2 200
> content-type: text/html; charset=ISO-8859-1
> content-security-policy-report-only: object-src 'none';base-uri 'self';script-sr                                                                              c 'nonce-vx62rPytlrXnMnOtNoxtgA' 'strict-dynamic' 'report-sample' 'unsafe-eval'                                                                               'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other                                                                              -hp
> p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
> date: Mon, 11 Mar 2024 09:15:30 GMT
> server: gws
> x-xss-protection: 0
> x-frame-options: SAMEORIGIN
> expires: Mon, 11 Mar 2024 09:15:30 GMT
> cache-control: private
> set-cookie: 1P_JAR=2024-03-11-09; expires=Wed, 10-Apr-2024 09:15:30 GMT; path=/;                                                                               domain=.google.com; Secure
> set-cookie: AEC=Ae3NU9P6ZkeQ5a4Cwo_H1P0rFyJKiY_8jm-WZvzTX6MCX3rEwDnVSSYQZQ; expi                                                                              res=Sat, 07-Sep-2024 09:15:30 GMT; path=/; domain=.google.com; Secure; HttpOnly;                                                                               SameSite=lax
> set-cookie: NID=512=jl4BpAZbGqpae0AyMqhcXjvEPN1EqfndV-uRTRpONSiaIrf-ce-Bo1EwGPzu                                                                              f9sukjQxUkH-_sMSkHURqhaJwPHBJt8CCusGmY3WE3jIl3nnReY7ndWE2SCPuG9IzGWI8PpnX5IS6oAd                                                                              BsesyH24XIeUQj2E3weSwf-ZGZA4JyU; expires=Tue, 10-Sep-2024 09:15:30 GMT; path=/;                                                                               domain=.google.com; HttpOnly
> alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000





也可以直接包装成一次对openai的api服务器的请求

```bash
[root@iZf8zbjkf8rufzntdamo8sZ clash-for-linux]# curl -X POST      -H "Content-Type: application/json"      -H "Authorization: Bearer sk-APIKEY"      --data '{"prompt":"translate English to French: Hello, world!","model":"text-davinci-003"}'      https://api.openai.com/v1/engines/text-davinci-003/completions     
```

> {
>     "error": {
>         "message": "Cannot specify both model and engine",
>         "type": "invalid_request_error",
>         "param": null,
>         "code": null
>     }
> }

虽然它返回的是error，但是从json的响应格式和错误类型也可以看出来，是成功访问到api的服务器了。









2024.1.31 成功设置代理

但是已知

2024.3.8 无法使用 环境变量没了 代理也没了 显示openssl的错误 不只是GPT的api服务器 而是所有的外网网址都有这个问题 内网网址可以正常访问。就是如上的问题，重新关闭打开一下就又好了。

```bash
curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to api.openai.com:443
```

并且在这段时间内GUI一直无法正常打开，最后纠因是因为本地目录下存在两个不同版本的clash目录，他们都有一个配置文件，将其中一个删除之后，就可以正常打开了。



![3ae074a9aafd2fd249f99625cbbf6aa](.\images\3ae074a9aafd2fd249f99625cbbf6aa.png)![image-20240311174236700](.\images\image-20240311174236700.png)