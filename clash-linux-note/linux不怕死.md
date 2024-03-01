





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