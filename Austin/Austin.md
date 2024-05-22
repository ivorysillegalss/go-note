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

core是上行消息 发送信息至mq
