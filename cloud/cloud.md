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



## 远程调用 - with Eureka

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



### Eureka

按照远程调用的基本思路，会引出两个问题

- 消费者如何得知提供者信息，包括url，健康状态等？
- 当程序并发量大，提供者以集群部署时，应该如何选择？

为了应对这样的问题，我们需要一个统一的中介角色，来管理提供者和消费者，并且登记提供者的地址等信息。同时，在出现集群时，有对应的算法均衡负载。Eureka便是这样的一个角色。在微服务启动时，提供者将自己的集群信息发送到Eureka中，并每隔30s向Eureka发送心跳响应，保证提供者的健康状态。（若提供者宕机则剔除出注册中心）。



#### 服务注册

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



#### 调用服务（服务拉取）

服务拉取：经过负载均衡的服务调用

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



### 负载均衡 - with Ribbon

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





## Nacos

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
	<scope>import/scope>
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

 

### 服务集群注册

配置文件需如下修改如

```properties
spring.cloud.nacos.discory.cluster-name: CN # 此处配置集群名称，一般为机房位置
```

将多个服务注册并注册到不同的集群当中之后，可以在Nacos的GUI中查看集群的信息。