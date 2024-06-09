已经有相当一段时间没有碰过java了 此md记录学习austin时的bug和笔记 im暂时搁置

### maven

1. **Could not find artifact org.pentaho:pentaho-aggdesigner-algorithm:jar:5.1.5-jhyde**

博客https://blog.csdn.net/Tangnin/article/details/125541627有解

修改maven配置文件settings.xml

报错时配置为

```xml
<mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>


     -->
    <!-- <mirror>
      <id>maven-default-http-blocker</id>
      <mirrorOf>external:http:*</mirrorOf>
      <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
      <url>http://0.0.0.0/</url>
      <blocked>true</blocked>
    </mirror>


  <! - 加入阿里云华为云作为镜像 -->
      <!-- huawei yun -->
      <mirror>
          <id>huaweicloud</id>
          <mirrorOf>*</mirrorOf>
          <url>https://repo.huaweicloud.com/repository/maven/</url>
      </mirror>

      <!-- 中央仓库1 -->
      <mirror>
          <id>repo1</id>
          <mirrorOf>central</mirrorOf>
          <name>Human Readable Name for this Mirror.</name>
          <url>http://repo1.maven.org/maven2/</url>
      </mirror>

      <!-- 中央仓库2 -->
      <mirror>
          <id>repo2</id>
          <mirrorOf>central</mirrorOf>
          <name>Human Readable Name for this Mirror.</name>
          <url>http://repo2.maven.org/maven2/</url>
      </mirror>


  </mirrors>
```

在原有配置前加入以下镜像依赖 成功解决

```xml
     <!-- aliyun yun -->
      <mirror>
          <id>alimaven</id>
          <mirrorOf>central</mirrorOf>
          <name>aliyun maven</name>
          <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      </mirror>

  <mirror>
	<id>aliyunmaven</id>
	<mirrorOf>*</mirrorOf>
	<name>spring-plugin</name>
	<url>https://maven.aliyun.com/repository/spring-plugin</url>
</mirror>
 
<mirror>
	<id>aliyunmaven</id>
	<mirrorOf>*</mirrorOf>
	<name>阿里云公共仓库</name>
	<url>https://maven.aliyun.com/repository/public</url>
</mirror>
 
<mirror>
	<id>nexus-aliyun</id>
	<mirrorOf>*,!cloudera</mirrorOf>
	<name>Nexus aliyun</name> 
	<url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```



2. maven3.8以上的版本不再支持https 但是可以通过修改配置文件解除屏蔽

同样是在mirror中的语句 注释掉其就可以

```xml
<mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>


     -->
    <!-- <mirror>
      <id>maven-default-http-blocker</id>
      <mirrorOf>external:http:*</mirrorOf>
      <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
      <url>http://0.0.0.0/</url>
      <blocked>true</blocked>
    </mirror>

  </mirrors>
```











重写一个项目取舍

取：

责任链模式

消息MQ

去重



舍：

hive



and：

flink 



改变：

ratelimiter改造

loadbalance算法接入第三方 可配置化 nacos

静态代码块注入筛选规则 改造为单独工厂



夜间消息屏蔽的运行逻辑

手写算法判断账号活跃度 决定是否刷新accessToken 节约资源



创建枚举类 包括消息正在责任链中的节点位置 在消息处理到链路中的不同环节的时候 使用枚举类标志对应的状态



对于消息是否送达重发送ack信息 异步处理写入数据库（回执）

目前的逻辑是仅仅同步判断是否写入成功 并将结果写入日志当中







handler是下发消息 执行去重等逻辑 消费MQ的信息

core是责任链相关类

service类代表上行消息 发送信息至mq的逻辑



也许可以抽离出再多一层 api层 与外界做交互？















目前写完了责任链初始化配置 其中具体action未补充

剩余业务

messsageTemplate消息模板的crud

根据消息模板发送信息至mq （4action with 责任链）

责任链相关entity的设计

消息链路追踪的设计





对于普通的一个下发消息 总共需要4个逻辑 

1. 前置检查 参数是否为空
2. 组装taskinfo信息
3. 判断参数格式是否合法
4. 发送信息至MQ

将逻辑分碎一点有利于业务的复用







@value注入分配错误会导致bean创建失败





发送 & 消费信息

根据程序运行时的逻辑来看的话

首先需要创建对应的消费者组

使用application.getBean()获取bean 

使用kafka提供的增强器将其映射到对应的类当中

在发信息的时候对其进行绑定

接受的时候 判断是否自己对应的消费者组







去重

去重整个可以分割成几个环节？

给谁去重？（满足去重的条件、去重规则）

如何去重？

去重之后呢？



同时这些处理方式都是多样化的

也就是说需要定义对应的模板 什么需要定义模板—— 在去重的过程中 什么是可变的？



去重的规则是可变的 且去重的规则会刷新

去重的对象是可变的

去重之后的处理方式是可变的？



判断是否执行满足去重的条件之后 

if 满足对应条件 执行去重 之后呢？ 根据去重的规则重新计算？

if 不满足对应条件 不执行去重 之后呢？ 在现有的基础上根据去重规则继续累加去重次数



同时 判断满足与否对应的条件 这一过程也可分成多个

- 获取对应去重配置 （去重规则）   一天只能发5条
- 从某一个地方读取对应条件    读取到现在已经发了4条
- 判断是否符合条件  去重or not      4 < 5   符合对应条件

- 在对应条件上累加 并再次持久化到对应存储介质当中      今天原发4条 +1条 现发5条

对于上方的读取操作 可以是在内存中 redis中等
