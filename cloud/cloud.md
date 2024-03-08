# cloud

## 使用的版本

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.9.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

```xml
<properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Hoxton.SR10</spring-cloud.version>
</properties>
```

​	



目前架构一般笼统可分为：**单体架构**或**微服务架构**

单体架构特点？

- 简单方便，高度耦合，扩展性差，适合小型项目。例如：学
  生管理系统

分布式架构特点？

- 松耦合，扩展性好，但架构复杂，难度大。适合大型互联网
  项目，例如：京东、淘宝

微服务：一种良好的分布式架构方案

- 单一职责：微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责，避免重复业务开发
- 面向服务：微服务对外暴露业务接口
- 自治：团队独立、技术独立、数据独立、部署独立            对应团队独立开发对应模块 

- 隔离性强：服务调用做好隔离、容错、降级，避免出现级联问题

**做到高内聚低耦合**



## 远程调用

假如现有商城项目中已有的两个模块 一个负责订单 一个负责用户

在调用单模块的前提下 我们只能获取到单订单以及用户id的信息 或是 单用户的信息

什么方法可以通过一次前端的请求获取出两者的所有信息呢？
在获取订单信息时 同时向负责用户的模块发起http请求 从而对信息进行整合响应

springboot给我们提供了 **RestTemplate** 来进行远程调用



### 提供者 & 消费者

- 服务提供者：提供接口给其他微服务的对象 —— 被调用接口的服务

- 服务消费者：调用其他微服务接口的对象 —— 调用其他服务接口的服务

例如上例中订单板块是消费者 用户板块是提供者

注意：提供与消费的关系是**相对的**，一般只适用于具体情况中，若某一模块提供了服务，则仅代表着他在此功能中具有提供者的身份。



## Eureka

按照远程调用的基本思路，会引出两个问题

- 消费者如何得知提供者信息，包括url，健康状态等？
- 当程序并发量大，提供者以集群部署时，应该如何选择？

为了应对这样的问题，我们需要一个统一的中介角色，来管理提供者和消费者，并且登记提供者的地址等信息。同时，在出现集群时，有对应的算法均衡负载。Eureka便是这样的一个角色。在微服务启动时，提供者将自己的集群信息发送到Eureka中，并每隔30s向Eureka发送心跳响应，保证提供者的健康状态。（若提供者宕机则剔除出注册中心）。



### 服务注册

搭建eureka-server 引入其依赖

```XML
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

在主类**SpringBootApplication**中添加 `@EnableEurekaServer`注解

```java
@EnableEurekaServer
@SpringBootApplication
public class EurukademoApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurukademoApplication.class, args);
    }
}
```



```properties
server.port=10086
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
spring.application.name=eurekaserver
eureka.client.service-url.defaultZone=http://127.0.0.1:10086/eureka
```

如果使用yml启动不行的话 可以尝试上方properties的配置

启动服务之后 可以在本机对应端口页面看到eureka的GUI



注册模块至微服务当中

配置文件与上方一样但是需要引入的是eureka-clent的包

```XML
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

ps： 可多次注册同一服务 需要修改对应端口



### 调用服务（服务拉取）

服务拉取：经过负载均衡的服务调用

如上所述，服务拉取目的是为了让服务的消费者能够获取到提供者的url信息等。但是这个拉取并不是每一次需要远程调用时都需触发的。Eureka中规定了**定时拉取&缓存**的机制，用以提高服务的效率。每一次拉取之后，所有的服务都将缓存在一个服务列表中。同时若长时间多次使用此服务时。也需要对提供者的健康状态等信息做出更新。于是Eureka中规定，**对统一服务的列表缓存每隔30s做出更新。**（前提是过了30s中此提供者仍需再次提供服务）



**如何修改？**

修改访问的路径 修改其中的**端口&ip**为服务名称

```java
String url = "http://127.0.0.1:8080/user/" + userId
// 修改为
String url = "http://userservice/user/" + userId
```

同时需要在对应创建`RestTemplate`的代码块中加入 `@LoadBalanced` 负载均衡的注解

```java
@Bean
@LoadBalanced
publice RestTemplate restTemplate(){
}
```





## Ribbon

### 负载均衡

从上方我们可知，进行服务拉取的url并不是真实的url，需要经过处理才进行远程调用。而**Ribbon**便是起了这个作用。

在发起请求的时候，Ribbon通过拦截器获取到url，并形成一个loadBanlancer的对象。根据url中的服务名称，去euruka中获取服务列表的真实地址。之后，将其由一个名为IRule的对象来实现负载均衡的策略，用户可以按照业务需求选择不同的方案。包含轮询调度和随机调度。最后以真实地址修改url，通过拦截器发出请求。

下方是具体规则及其策略：

| 内置负载均衡规则类        | 规则描述                                                     |
| ------------------------- | ------------------------------------------------------------ |
| RoundRobinRule            | 简单轮询服务列表来选择服务器。它是Ribbon默认的负载均衡规则。 |
| AvailabilityFilteringRule | 对以下两种服务器进行忽略：<br/>(1)在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加。<br/>(2)并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule规则的客户端也会将其忽略。并发连接数的上限，可以由客户端的<clientName>.<clientConfigNameSpace:>.ActiveConnectionsLimit属性进行配置。 |
| WeightedResponseTimeRule  | 服务器，这个权重值会影响服务器的选择。为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。 |
| ZoneAvoidanceRule         | 以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。而后再对Zone内的多个服务做轮询。 |
| BestAvailableRule         | 忽略哪些短路的服务器，并选择并发数较低的服务器。             |
| RandomRule                | 随机选择一个可用的服务器。                                   |
| RetryRule                 | 重试机制的选择逻辑                                           |



### 如何修改此负载均衡规则？

1. **代码修改** 在调用类（消费者）的启动类中，定义一个新的**IRule**的bean

```java
@Bean
public IRule randomRule(){
	return new RandomRule();
}
```

2. **配置文件中修改**

在**application.yml**文件中，添加新的配置规则

```yml
userservice:
	ribbon:
		NfLoadBalanceRuleClassName: com.netflix.Loadbalancer.RandomRule #负载均衡规则
```



### 饥饿加载

**Ribbon**默认的是懒加载，即首次访问对应资源才会去创建对应的LoadBalanceClient，这一种特点也导致了请求的时间会很长。

而饥饿加载则会在项目启动时创建，降低访问时创建对象的时间，可通过修改如下配置开启。

```
ribbon:
	eager-load:
		enabled: true # 开启
		clients: userservice # 指定需要开启的服务的名字
		
		# 若需要同时指定多个开启 则如下
		clients:
		 - userservice
		 - adminservice
```





## Nacos 关于注册中心

### 下载运行

Nacos与euruka类似，也是起到一个注册中心的角色。可从其github主页上进行下载。成功下载运行后可以在默认的 **8848** 端口查看GUI，即为下方页面。

![image-20240307191110938](.\images\image-20240307191110938.png)

​	(命令行启动成功)

![image-20240307191349495](.\images\image-20240307191349495.png)

(浏览器打开时的官方GUI)



### 服务注册

此处仍以上方**订单orderService**和**用户userService**为例

与Eureka同理，使用nacos的时候需要在两者的父工程中导入父依赖
不完全恰当的说，可以把服务理解成client，父工程理解成对应client的server

在项目使用时首先需要添加对应的依赖：
父工程：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.5.RELEASE</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>
```

配置文件设置：（这里以properties为例）

```properties
spring.cloud.nacos.server-addr: localhost:8848
```


作为client的orderService和userService同样需要导入客户端的依赖包

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-
discovery</artifactId>
</dependency>
```



### 服务分级存储模型 （集群概念）TBD

在Nacos中，服务之间的关系可分为三个层次：

1. 服务
2. 集群
3. 实例

可想而知，在远程调用的时候，优先选择**同一集群中**的实例，因为实例之间的物理距离更短，调用时间也更短。而集群一般是以地址来划分的。



### 服务集群注册

配置文件需如下修改如

```properties
spring.cloud.nacos.discory.cluster-name: CN # 此处配置集群名称，一般为机房位置
```

将多个服务注册并注册到不同的集群当中之后，可以在Nacos的GUI中查看集群的信息。



### 集群权重配置 NacosRule

如上方服务注册的时候，我们虽然可以通过集群的配置和配合Ribbon进行负载均衡的方法来减轻各服务器的压力，但是在实际开发中，仍有很多不同的使用场景需要手动分配使用频率。例如说，两条服务器，一台性能比较好，一台性能比较一般，出于性能的考虑，我们肯定希望往好的装多一点数据诸如此类。于是Nacos就引入了权重的概念，并设计了根据权重负载均衡。

**如何修改？**

在Nacos的GUI界面中，可以看到不同实例的默认权重值都为1。可以根据实际情况将其调成任意值，但注意，若权重值为0，则代表此服务不会被调用。

结合上方修改权重的方式，我们就可以做到平滑更新（不停机更新）。可在软件使用的闲时短，流量较小时。令对应的服务权重为0，进行更新后，再度切换，以此类推。当然这只是一个大概的思路，实际的业务操作要更为复杂。



### 环境隔离 & 命名空间 (namespace)

我们根据不同运行环境的变化（生产环境 & 测试环境 & 开发环境），有时候需要对服务器做隔离，称之为环境隔离。

在Nacos中，**namespace**是环境隔离的体现。每个namespace都有自己的唯一id，且不同namespace下的服务**相互不可见**。

整个环境隔离的形式如下：

![image-20240308161846609](.\images\image-20240308161846609.png)

一般情况下，同一个Group中是业务相关度较高的服务集合。

> ps:这个Group也就是Nacos的GUI在服务列表处的分组名称。

**如何创建命名空间并应用服务？**

0. 在默认情况下，Nacos创建一个默认的命名空间为public，所有的服务都在此空间中。

![image-20240308162501733](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240308162501733.png)

1. 如图 命名空间处新建

![image-20240308162231343](.\images\image-20240308162231343.png)

2. 填写信息

![image-20240308162753460](.\images\image-20240308162753460.png)

3. 成功创建后可以看到此命名空间的id (默认以uuid规则生成)，每个namespace都有自己的唯一id。

![image-20240308163016581](.\images\image-20240308163016581.png)

4. 在对应的配置文件下加入namespace的配置选项，例：

```properties
spring.cloud.nacos.discovery.namespace: 492a7d5d-237b-46a1-a99a-fa8e98e4b0f9
# 此选项与集群名字配置为同级
```

以上例修改之后，效果如下：

![image-20240308163506911](.\images\image-20240308163506911.png)



### 服务拉取机制 differenceWithEureka

我们知道Eureka中，服务的提供者定时与Eureka进行心跳响应，从而更新其状态。但是在Nacos中不完全一样。

Nacos会将实例区分为**临时实例**或**非临时实例**：

- 对于临时实例，其心跳响应机制跟Eureka大致相同，但是其响应频率更高。若其宕机，Nacos**直接将其从服务列表中剔除**。
- 对于非临时实例，其不会对Nacos进行心跳相应。而是由Nacos主动发送请求，并确认对应状态。（有点像是变相的心跳响应）若其宕机，Nacos也不会剔除，仅仅标记其为**不健康服务**。

在Eureka中，服务消费者会对Eureka定时拉取**(pull)**，拉取更新服务列表的信息。而由于Nacos与服务提供者的不同特性，相比之下服务更新的状态时效性较差。于是Nacos中增加了新的特性。在提供者和Nacos的状态更新中，若**有实例宕机或健康状态改变，Nacos会主动推送(push)实例的变更信息。**从而保证服务列表中状态的一致性。

但是主动推送这一特性，对服务器带来的压力会较大。所以需结合使用场景，斟酌好使用临时实例或非临时实例。



**如何将服务配置为临时或 非临时实例？**

修改配置文件如：

```properties
spring.cloud.nacos.discovery.ephemeral: flase
# 非临时实例
spring.cloud.nacos.discovery.ephemeral: true
# 临时实例
```



**集群相关TBD**

- Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；

- Eureka采用AP方式

CP侧重于保证数据的一致性。



## Nacos 关于配置管理

在之前的单体架构项目中，如果我们需要修改配置，只需要修改它的配置文件就好了。但是在微服务的应用场景下。同时修改多个配置文件并重新运行进行使用所需的时间成本很长，效率极低。为了解决此需求，Nacos中有对应的统一配置管理，实现了所有配置更改的热更新。

**TBD图**

### 如何添加配置信息？

1. 于Nacos配置的GUI配置列表处新建



2. 填写配置信息 （注意其文件名需独立）



3. 注意配置信息中应只填写需**热更新**的，勿必将所有的配置全部写上去，否则将大大浪费Nacos的配置资源



### 配置管理使用

添加了配置信息之后，我们需要在实际的项目中结合使用。这里以项目启动为流程先介绍冷更新。



**TBD图**

在整个模块的运行中，一般情况下，程序会先读取我们所指定的本地`application.yml`（或properties）文件，并结合其中配置创建容器加载bean等等。

而如今我们需要读取在线的**nacos热配置文件**，但是对应的nacos配置文件在哪，其中内容是什么都不知道，所以我们需要再引入一个`bootstrap.yml`文件，其中记录了在线nacos的配置文件地址，通过此文件找到naco热配置文件，再读取本机配置文件，与其相结合。

不难看出，**bootstrap文件**与**naco热配置文件**的优先级都比本地配置文件要高。



在需要使用热配置更新的客户端当中，我们首先需要引入对应的配置管理客户端依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
<dependency>
```



引入依赖后，在对应的模块中新建**bootstrap**并写入类似以下的信息：

```yaml
spring:
	application:
		name: userservice # 对应的服务名称
    profiles:
    	active: dev # 开发环境 dev为生产环境
    cloud:
    	nacos:
    		server-addr: localhost:8848	# nacos的地址
    		config:
    			file-extension: yaml # 对应热配置文件的后缀名
```



### 配置管理热更新

Nacos不仅支持上方的配置冷更新，同时也支持热更新。结合springboot的有两种方式实现：

**TBD**

1. 通过`@Value`注解在对应类中**注入**，并结合`@RefreshScope`注解进行刷新。



2. 通过`@ConfigurationProperties(prefix = "pattern")`注解注入



### 多环境配置共享

微服务启动时会从nacos中读取多个配置文件：

1. [spring.application.name]-[spring.profiles.active].yaml
2. [spring.application.name].yaml

第一个配置文件中，我们可以根据**运行环境**的不同，来填写不同的配置。但是在程序中，有很大一部分配置都是已经一般完善好的，在一般情况下不许改变。这一类的配置在多种**运行环境**中都通用，一般存放在第二种配置文件当中。

