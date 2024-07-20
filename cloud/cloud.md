# Cloud

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
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        <version>2021.1</version>
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

![image-20240310193127679](.\images\image-20240310193127679.png)

2. 填写配置信息 （注意其文件名需独立）

![image-20240310193206435](.\images\image-20240310193206435.png)

3. 注意配置信息中应只填写需**热更新**的，勿必将所有的配置全部写上去，否则将大大浪费Nacos的配置资源



### 配置管理使用

添加了配置信息之后，我们需要在实际的项目中结合使用。这里以项目启动为流程先介绍冷更新。

![image-20240310192441239](.\images\image-20240310192441239.png)

在整个模块的运行中，一般情况下，程序会先读取我们所指定的本地`application.yml`（或properties）文件，并结合其中配置创建容器加载bean等等。

![image-20240310192611984](.\images\image-20240310192611984.png)

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

1. 通过`@Value`注解在对应类中**注入**，并结合`@RefreshScope`注解进行刷新。

![image-20240310192253354](.\images\image-20240310192253354.png)

![image-20240310192124916](.\images\image-20240310192124916.png)

2. 通过`@ConfigurationProperties(prefix = "pattern")`注解注入

![image-20240310192209913](.\images\image-20240310192209913.png)

### 多环境配置共享

微服务启动时会从nacos中读取多个配置文件：

1. [spring.application.name]-[spring.profiles.active].yaml
2. [spring.application.name].yaml

第一个配置文件中，我们可以根据**运行环境**的不同，来填写不同的配置。但是在程序中，有很大一部分配置都是已经一般完善好的，在一般情况下不许改变。这一类的配置在多种**运行环境**中都通用，一般存放在第二种配置文件当中。

对于相同的配置项，可能在不同的配置文件中被重复定义了，那么此时以哪一个为准呢？

![image-20240311224232528](.\images\image-20240311224232528.png)

将所有情况列举出来可以得到：`特定环境  >  云端共享  >  本地`





## Feign

Feign为一**声明式**http客户端，替代发起远程调用的`RestTemplate`



### 快速使用

引入依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
<dependency>
```

在主类**SpringBootApplication**中添加 `@EnableFeignClients`注解

```java
@EnableFeignClients
@SpringBootApplication
public class EurukademoApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurukademoApplication.class, args);
    }
}
```

上述两部完善使用Feign的前提，下面为对`RestTemplate`和`Feign客户端`的同一业务使用对比

1. **RestTemplate**

```java
String url = "http://userservice/user/" + order.getUserId();
User user = restTemplate.getForObject(url,User.class);
```

2. **Feign**

```java
@FeignClient("userservice")
public interface UserClient{
	@GetMapping("/user/{id}")
	User findByIf(@PathVariable("id") Long id);
}
```

当使用远程调用的时候，只需要将对应声明所需的信息告诉Feign，就可以自动装配，有点类似**Mybatis的注解使用**

> 声明一个服务需要什么信息？ 如上：
>
> 服务名称：userservice
>
> 请求方式：GET
>
> 请求路径：/user/{id}
>
> 请求参数：Long id
>
> 返回值类型：User

只要拿到这五个信息 并在对应的模块中进行注册并自动装配，就可以直接调用接口进行远程调用



### 自定义Feign配置项

配置Feign可在**配置文件中**或者**代码中**直接定义

1. 配置文件

**全局生效**

```yaml
feign:
	client:
		config:
			default:   # 此处default代表配置项的配置 适用于全局
				loggerLevel: FULL
```

**局部生效**

```
feign:
	client:
		config:
			userservice: 
    # 此处非default 而是以某一个服务名称 代表只适用于对应微服务的配置
				loggerLevel: FULL
```



2. 代码处修改

需要声明bean作为配置类

```java
public class FeignclientConfiguration{
	@Bean
	public Logger.Level feignLogLevel(){
		return Logger.Level.FULL;
	}
}
```

相似的，全局配置和局部配置也有不同的配置形式，表现在注解上

**全局配置**

```java
@EnableFeignCLients(defaultConfiguration = FeignclientConfiguration.class)
```

相似的，将配置类放到SpringBootApplication启动类中，开启上方注解，即可实现全局配置

**局部生效**

```java
@Feignclient(value "userservice",configuration = FeignclientConfiguration.class)
```

使用`value`指定微服务名称,`configuration`指定配置类，放到上方所述的Feign的client类中即可完成面向单服务的配置。



### 性能优化

关于底层的客户端实现：

- URLConnection 默认实现 不支持连接池
- Apache HttpClient 支持连接池
- OKHttp 支持连接池

而**Feign**采用的是默认的URLConnection，由于不支持连接池，其在性能方面没有后两者优，但是我们可以通过配置来改变。同时可以减小日志级别，最好是为`basic or none`因为打印日志对性能的消耗很大



**连接池配置** 以HttpClient为例

引入依赖：

```xml
<!--httpClient的依赖-->
<dependency>
	<groupId>io.github.openfeign</groupId>
	<artifactId>feign-httpclient</artifactId>
</dependency>
```

配置连接池：

```
feign:
	client:
		config:
			default:#default全局的配置
				LoggerLevel:BASIC#日志级别，BASIC就是基本的请求和响应信息
httpclient:
	enabled:true#开启Feign对HttpClient的支特
	maX-connections:200#最大的连接数
	maX-connections-per-route:50#每个路径的最大连接数
```



### 应用方法

观察远程调用，我们可以发现在消费者和提供者所提供的**接口信息是一致的且必须一致**（两者的请求类型 请求参数 路径信息等等）

```java
@GetMapping("/user/id]")
User findById(@PathVariable("id")Long id);
```

```java
@GetMapping("/id]")
public User queryById(@PathVariable("id")Long id){
	return userService.queryById(id);
}
```

我们可以根据这个特点对代码进行再次优化，可以大致分为两种：

1. **继承** 给消费者的FeignClient和提供者的controller定义统一的父接口作为标准。

```java
public interface UserAPI{
@GetMapping("/user/id}")
	User findById(@PathVariable("id")Long id);
}
```

上为两者的父接口吗，若此时我作为**消费者(orderService)**，我只需要继承这一个父接口便可

```java
@Feignclient(value "userservice")
public interface Userclient extends UserAPI{}
```

若我为**提供者(userService)**，我也可以实现这一个接口，并完善具体逻辑即可。

但是这一种方法会使不同的微服务模块的api层上存在**紧耦合**，于是就有了方法2



2. **抽取**  将FeignClient抽取为独立模块，并且把接口有关的POO、默认的Feigni配置都放到这个模块中，提供给所有消费者使用

​	![image-20240314153953782](.\images\image-20240314153953782.png)

​	大白话说，就是把使用频率高的clients和他的配置等都封装到一个模块中，消费者通过调用这个模块来实现远程调用。





### 关于抽取

如何实现抽取？以下以`userService & orderService`为例进行说明抽取过程

1. 创建新模块，另存userService模块有关的api

2. 导入`spring-cloud-open-feign`的依赖
3. 将orderService中与userService有关的代码挪移到新模块中，删掉user相关的配置类
4. orderService导入userService模块作为依赖，重新导包

此时已经将代码成功抽取成功了，但是如果运行的话会失败，如下文报错

```tex
Field userclient in cn.exp.order.service.OrderService required a bean of types'cn.exp.feign.clients.Userclient'that could not be found.

The injection point has the following annotations:  
@org.springframework.beans.factory.annotation.Autowired(required=true)
```

报错的原因是，虽然成功导入了UserClient类，但是ispringboot自动注入对象(@Autowired)是以包为单位。而UserClent在另外一个模块中，根本就不在一个保重，所以即使识别到了有这个类，也无法为其分配对象。



**解决办法**

依然是在SpringBoot启动类的注解上加参数

- 在@EnableFeignClients注解中添加pasePackages,指定
  FeignClient所在的包

```java
@EnableFeignclients(basePackages "cn.itcast.feign.clients")
```

但是这一种方法也是直接以包为单位，是直接把对应的路径中的所有包加入扫描范围。即使不需要用到的包也是

- 在@EnableFeignClients注解中添加clients,指定具体
  FeignClientl的字节码

```java
@EnableFeignclients(clients = {UserClient.class})
```

这一种方法只需要将使用到的类挨个注册就好了，在需要调用的包数量较少的时候，是更优的解法。





## GateWay

![image-20240317160850172](.\images\image-20240317160850172.png)

当我们的项目中有不同的角色，且不同的角色具有不同的权限组时，**身份的认证和权限的校验**是必需的，此时网关就可以起到这一作用。

同时，网关将服务路由到对应的微服务中，同时也起到负载均衡的作用。

当服务器的流量过大时，服务器可以起到一个拦截器的作用，将请求进行限流。



### 快速运行

创建新模块，并引入对应依赖（包括nacos的服务发现依赖）

```java
<！--网关依赖 -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<! --nacos服务发现依赖 -->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

GateWay本身也算是一个服务，也需要在nacos上进行注册

下方为基本配置及 **编写路由的配置**  

路由配置一般包括：

1. **路由id**:路由的唯一标示
2. **路由目标**(uri):路由的目标地址，http代表固定地址，lb代表根
   据服务名负载均衡
3. **路由断言**（predicates): 判断路由的规则是否符合规则组符合则转发到路由目的地
4. **路由过滤器**（filters):对请求或响应做处理

```yaml
server:
	port: 10010 #网关端
spring:
	application:
		name: gateway #服务名称
cloud:
	nacos:
		server-addr: localhost:8848#nacos地址
		
		
gateway:
	routes: #网关路由配置
		- id: User-service #路由id,自定义，只要唯一即可
			#uri: http://127.0.0.1:8081#路由的目标地址http就是固定地址
			uri: lb://userservice #路由的目标地址Lb就是负载均衡，后面限服务名称
			predicates: #路由断言，也就是判断请求是否符合路由规则的条件
				- Path=/User/** #这个是按照路径匹配，只要以/User/开头就符合要求
		- id: order-service
			uri: lb://orderservice
			predicates:
				- Path=/order/**
            
            # 网关过滤器配置
            # filters: AddRequestHeader=Truth,Itcast is freaking aowsome!
    default-filters: AddRequestHeader=Truth,Itcast is freaking awesome!
```

上方routes为基本的路由配置

![image-20240317162653823](.\images\image-20240317162653823.png)

以此图+上方路由配置为例，由于端口是10010，用户访问到网关，由于访问的是`/user/1`，按照路由规则应路由到userservice中，于是网关通过nacos注册中心拉取userservice服务的列表。并经过负载均衡后访问到真实的地址。



### 路由断言工厂

**Route Predicate Factory**

在gateway中，路由断言的规则是由其中**断言工厂**读取并进行处理的。可知上方的路由断言规则是根据路径来判断的，但是断言的条件在 **SpringCloudGateway** 中还有很多个

```yaml
routes: #网关路由配置
		- id: User-service #路由id,自定义，只要唯一即可
			#uri: http://127.0.0.1:8081#路由的目标地址http就是固定地址
			uri: lb://userservice #路由的目标地址Lb就是负载均衡，后面限服务名称
			predicates: #路由断言，也就是判断请求是否符合路由规则的条件
				- Path=/User/** #这个是按照路径匹配，只要以/User/开头就符合要求
```

如下：

| 名称       | 说明                         | 示例                                                         |
| ---------- | ---------------------------- | ------------------------------------------------------------ |
| After      | 是某个时间点后的请求         | `-After:=2037-01-20T17:42:47.789-07:00[America/Denver]`      |
| Before     | 是某个时间点之前的请求       | `-Before:=2031-04-13T15:14:47.433+08:00[Asia/Shanghai]`      |
| Between    | 是某两个时间点之前的请求     | `-Between:=2037-01-20T17:42:47.789-07:00[America/Denver],2037-01-21T17:42:47.789-07:00[America/Denver']` |
| Cookie     | 请求必须包含某些cookie       | `Cookie=chocolate,ch.p`                                      |
| Header     | 请求必须包含某些neader       | `Header=X-Request-ld,\d+`                                    |
| Host       | 请求必须是访问某个host(域名) | `  Host=**.somehost.org,**.anotherhost.org`                  |
| Method     | 请求方式必须是指定方式       | `Method=GET,POST`                                            |
| Path       | 请求路径必须符合指定规则     | `-Path=/red/{segment),/blue/**`                              |
| Query      | 请求参数必须包含指定参数     | `  -Query=:name,Jack或者-Query=name`                         |
| RemoteAddr | 请求者的ip必须是指定范围     | `-RemoteAddr=:192.168.1.1/24`                                |
| Weight     | 权重处理                     |                                     



### 路由过滤器

#### **GateWayFilter **

![image-20240317170154055](.\images\image-20240317170154055.png)

此处的路由过滤器与一般的过滤器概念是相同的，执行时机为**路由到对应微服务组件**，以及处理完**微服务逻辑之后的**，即网关的执行时机。可以在这两环节中对其中数据进行处理。

```yaml
gateway:
	routes: #网关路由配置
		- id: User-service 
            # 网关过滤器配置
            # filters: AddRequestHeader=Truth,Itcast is freaking aowsome!
    default-filters: AddRequestHeader=Truth,Itcast is freaking awesome!
```

相关的过滤器可配置选项也有很多，详细可查阅光放网站，此处仅展现配置的基本形式

如例：写在特定的网关路由当中，则表明此过滤器仅使用于当前的路由规则

若想要过滤器对所有的网关规则生效，可如上配置`default-filters`



####  **GlobalFilter**

上方的可配置选项是spring提供的，换句话说，虽然可选项很多，但是难免也会由不满足需求的时候，特别是需要此过滤器执行的逻辑较复杂。此时需要定义一个**GlobalFilter**，此过滤器是通过实现同名接口实现的，充分满足逻辑处理的要求。

下为对应接口及其对应参数的信息

```java
public interface GlobalFilter{
    /**
    *	处理当前清求，有必要的话通过{@link GatewayFilterChain}将清求交给下一个过滤器处理
    * @param exchange请求上下文，里面可以获取Request、Response等信息
    * @param chain用来把请求委托给下一代过滤器
    * @return{@code Mono<Void>}返回标示当前过滤器业务结束
    */
	Mono<Void>filter(ServerWebExchange exchange,GatewayFilterChain chain);
}
```

具体以登录鉴权为例：

```java
@Order(-1)
@Component

public class AuthorizeFilter implements GlobalFilter{
	@Override
    public Mono<Void>filter(ServerWebExchange exchange,GatewayFilterChain chain){
	    //1.获取请求参数
        ServerHttpRequest request = exchange.getRequest();
        MultiValueMap<String,String> params = request.getQueryParams();
        //2.获取参数中的authorization参数
        String auth = params.getFirst(key:"authorization");
        //3.判断参数值是否等于admin
        if ("admin".equals(auth)){
        //4.是，放行
        return chain.filter(exchange);
        }
        //5.否，拦截
        //5.1.设置状态码
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        //5.2.拦截请求
        return exchange.getResponse().setComplete();
    }
}
```

例子方法中简单封装了鉴权逻辑，初次之外，上方的`@Order`代表着在多个过滤器并存时，过滤器之间执行的顺序（优先级）
其中可以填一个int型的数字。**order中值越大，执行顺序越小**。即当`@Order(Integer.MAX_VALUE)`时，执行的优先级最小，反之亦然。

同时，也可以使对应的过滤实现一个名为`Ordered`的接口，实现其中的`getOrder()`方法，这一方法和注解指定的效果是一样的。

```java
@Override
public int getOrder(){
	return 0;
}
```



### 关于过滤器的执行顺序

- 每一个过滤器都必须指定一个int类型的order值，orderf值越小，优先级越高，执行顺序越靠前。
- GlobalFilteri通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定
- 路由过滤器和defaultFilter的order由Spring指定，默认是按照声明顺序从1递增。
- 当过滤器的order值一样时，会按照**defaultFilter>路由过滤器>GlobalFilter**的顺序执行。
  

### 跨域处理

在网关介入的微服务程序中，由于所有的请求都是先到网关再到微服务的。也就是说对于相同的配置和请求，我们不需要在每一个微服务组件都进行实现，只需要在网关中配置即可。跨域请求也是如此。

有的时候通过接口测试工具调试接口的时候可以正常访问，但是一旦使用前端页面+ajax发起请求就会有问题，这种情况下很有可能就是跨域的锅。

![image-20240317184009317](.\images\image-20240317184009317.png)



**什么是跨域？**

跨域：域名不一致就是跨域，主要包括：

- 域名不同：www.taobao.com和www.taobao.org和www.jd.com和 miaosha.jd.com
- 域名相同，端口不同：localhost:8080 和 localhost8081
- 跨域问题：**浏览器**禁止请求的发起者与服务端发生跨域**ajax**请求，请求被浏览器拦截的问题
- 解决方案：CORS   (浏览器询问服务器是否允许跨域)
  

但是网关是基于`webFlux`实现的，在实现上与之前不同



```yaml
spring:
	cloud:
		gateway:

			globalcors: #全局的跨域处理
				add-to-simple-url-handler-mapping: true #解决options请求被拦截问题
				corsConfigurations:
					'[/*]':
						allowed0rigins:#允许哪些网站的跨域请求
							- "http://localhost:8090"
							- "http://www.leyou.com"
						allowedMethods: #允许的跨域ajaX的请求方式
                            - "GET"
                            - "POST"
                            - "DELETE"
                            - "PUT"
                            - "OPTIONS"
                        allowedHeaders: "*" #允许在请求中携带的头信息
                        allowCredentials: true #是否允许携带cookie
                        maxAge: 360000 #这次跨域检测的有效期
```

上方所说的CORS方案是浏览器通过`options`请求来询问服务器的 通过`add-to-simple-url-handler-mapping`的配置，首先先允许浏览器的访问请求。并根据需要选择需要跨域的url

跨域的有效期结束后需要重新配置，而频繁配置对服务器的压力很大，所以我们这里设置`maxAge`为大数





## RabbitMQ

所有的服务可以按照特性粗略的分为**异步通讯**或**同步通讯**

##### 同步調用

就如发起远程调用的**Feign**是同步通讯。消費者需要等待生产者的响应。

当业务逻辑随着迭代变得更复杂，多个微服务组件共存时。每次调用时，会挨个模块进行处理，**时延**会变长，系统效率变低。并且同步通讯下，信息容易耦合。不利于服务的更新迭代。

若在调用中某个生产者突然宕机，也会由于服务间的强依赖，而导致**级联失败**的问题

##### 异步调用

事件驱动模式，在同样需要处理多个服务板块的时候。异步通讯的方式会使消费者**发布一个事件**，以此事件为媒介（**broker**），通知各个模块进行数据处理。与此同时，消费者无需等待各模块信息处理完毕，便可返回信息给用户。

这一特性首先意味着**性能的提升和吞吐量的提高**

同时broker的存在也对流量进行了**削峰**的作用。在高并发情况下，若按照同步通讯的模式，所有的请求就会瞬间打在各个组件当中，直接由各服务来承受高并发的压力。相比之下，异步通讯broker的存在，可以添加限流的缓冲机制。若超于服务的承受能力，可以先将请求阻塞在broker当中。起到缓存实现保护作用。

但同时这一模式也会存在缺点

不难发现，上述所说的所有优点都由于一个新引入的角色（事件 or broker），那么如若**broker宕机**，对整个微服务集群的影响是非常严重的。

并且，引入broker之后，异步通讯的模式也决定了他的架构更加复杂，代码的实现和运维的难度大大提高。





### MQ

经过上面的引入，我们可以了解到在事件驱动架构中，最重要的部分是 **broker** 。在实际开发中，最常用的broker实现也就是**MQ**（MessageQueue）。

以订单系统为例，若A订单下单成功，则会往MQ中发送一个事件（成功订购）的事件，也就是消息队列中的消息，并由MQ进行管理。相对应的，生产者（需要处理的业务逻辑）会订阅此事件。

市面上主流的MQ包括 RabbitMQ , ActiveMQ , RocketMQ , Kafka 并且各有各的优点 此处不再赘述 

![image-20240318165043335](.\images\image-20240318165043335.png)

由于不同MQ之间的特性，一般来说，我们在**日志数据传输的场合中**可能偏向于使用kafka （时延 & 吞吐量）

而在中小型企业中的**业务数据处理** 使用RabbitMQ可能会更适合（稳定性 & 无需深度定制）

大型企业则可能更多的使用Rocket并在此基础上对其做出改进



### 安装 - docker

##### 单机部署

以Centos7虚拟机为例，使用Docker来安装。

在线拉取镜像文件

``` sh
docker pull rabbitmq:3-management
```




##### 集群部署

接下来，我们看看如何安装RabbitMQ的集群。

###### 集群分类

在RabbitMQ的官方文档中，讲述了两种集群的配置方式：

- 普通模式：普通模式集群不进行数据同步，每个MQ都有自己的队列、数据信息（其它元数据信息如交换机等会同步）。例如我们有2个MQ：mq1，和mq2，如果你的消息在mq1，而你连接到了mq2，那么mq2会去mq1拉取消息，然后返回给你。如果mq1宕机，消息就会丢失。
- 镜像模式：与普通模式不同，队列会在各个mq的镜像节点之间同步，因此你连接到任何一个镜像节点，均可获取到消息。而且如果一个节点宕机，并不会导致数据丢失。不过，这种方式增加了数据同步的带宽消耗。

我们先来看普通模式集群。

###### 设置网络

首先，我们需要让3台MQ互相知道对方的存在。

分别在3台机器中，设置 /etc/hosts文件，添加如下内容：

```
192.168.150.101 mq1
192.168.150.102 mq2
192.168.150.103 mq3
```

并在每台机器上测试，是否可以ping通对方：



### 运行MQ

执行下面的命令来运行MQ容器：

```sh
docker run \
 -e RABBITMQ_DEFAULT_USER=root \
 -e RABBITMQ_DEFAULT_PASS=123456 \
 --name mq \
 --hostname mq1 \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq:3-management
```

> -- hostname 为主机名字 单机部署时作用不大 但是若需要升级为集群的部署 则需要使用
>
> -- p 端口映射  15672为RabbitMQ官方的GUI映射端口  5672为通讯端口

成功运行之后 也可以通过`docker ps`观察其是否在执行中的进程

若没有问题，可进入对应的端口地址，能看到他的GUI页面。

若为云服务器，则为真实IP。若为虚拟机，可以在

![image-20240318171440970](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240318171440970.png)

成功登录之后， 可以看到以下的页面：

- Connections中记录了生产者和消费者经MQ所进行的连接

- Channels则是一个让生产者和消费者相连接的通道

- Exchanges代表担当路由角色的交换机

- Queues则是对应的队列

- Admin是对应的管理员界面，管理信息

![image-20240318172139233](.\images\image-20240318172139233.png)



### 总体结构和概述

![image-20240318172638235](.\images\image-20240318172638235.png)

当信息被发送过来时，首先经由交换机进行路由，将信息传输到队列之中进行缓存（交换机只具备路由的功能）。消费者从队列中获取消息，进行处理。至于其中的虚拟主机是以角色为单位的，若新增一角色，则新增一虚拟主机。并且各个虚拟主机之间是**隔离**的，**不会互相干扰**。



### AMQP

消息发送和接受的一种标准，与语言无关。可用任意的语言发送。多被使用于MQ中的信息收发。



### 快速入门 && SpringAMQP

由于rabbitMQ自带的相关api操作及其繁琐，spring在其基础上进行了简化和封装，并提供了对应的模板`RabbitTemplate`供使用。以下直接略过原生api的使用，介绍如何通过`SpringAMQP`达成操作。

​	MQ的官方文档中有5个demp示例，分别对应不同的用法，以下一一道来：

1. **基本消息队列**

HelloWorld案例

​	`发布者 --> 消息队列 --> 订阅队列` 

![image-20240318173240610](.\images\image-20240318173240610.png)

 基本的实现流程如下：

1. 引入依赖
2. 利用`RabbitTemplate`，在提供者服务中发送消息到`simple.queue`队列 （即基础消息队列）

3. 在消费者服务中编写消费逻辑，并且绑定`simple.quque`队列
   

对应的AMQP依赖
由于提供者和消费者都需要使用到这一依赖 因此这里直接把依赖写入父工程的文件中

```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```



**提供者操作**

在`application,yml`中编写配置，添加对应的连接信息

```yaml
spring:
	rabbitmq:
        host: 192.168.150.138 # 主机名
        port: 5672 # 端口
        virtual-host: / # 虚拟主机
        username: root # 用户名
        password: 123456 # 密码
```

构建一个单元测试类，编写测试方法：

```java
@SpringBootTest
public class SpringAmqpTest{
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Test
    public void testSimpleQueue(){
        String queueName = "simple.queue";
        String message "hello,spring amqp!";
		rabbitTemplate.convertAndSend(queueName,message);
    }
}
```



**消费者操作**

与提供者不同，消费者只需要创建一个监听器，来编写对应的消费逻辑：

```java
@Component
public class SpringRabbitListener{
    @RabbitListener(queues = "simple.queue")
	// 注解声明队列名称
    public void listensimpleQueueMessage(String msg) throws InterruptedException{
		System.oUt.println("spring消费者接收到消息：【"+msg+"】");
    }
}
```



2. **工作队列**

可以提高消息处理速度，避免队列消息堆积

多个消费者绑定同一个队列，一个信息只能由一个消费者进行处理

![image-20240319145919809](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240319145919809.png)

在默认配置下，工作队列会采用消息预取机制：当大量的消息到达队列时，不同的消费者会提前通过**channel**把消息先拿过来（不处理）。但是这种情况下是与消费者的性能无关，是按照消费者的数量平均分配的。

而可以通过修改配置文件，控制消息预取的上限：

 

```yaml
spring:
	rabbitmq:
    h0st: 192.168.150.101 #主机名
	p0nt: 5672 #端口
    virtval-host:/ #虚拟主机
    username: itcast #用户名
    password: 123321 #密码
    listener:
	    simple:
    		prefetch: 1 #每次只能获取一条消息，处理完成才能获取下一个消息
```



3. **发布 订阅模式**

其队列分发消息原理与基本工作队列是类似的。但是在此基础上加入了交换机，在SpringAMQP中，常见的交换机包括以下三种：

![image-20240319152620723](.\images\image-20240319152620723.png)

当提供者发送消息的时候，通过交换机路由到对应的队列中，进行消息的分发。但是exchange交换机这一角色只负责消息路由，不负责消息存储，若**在路由的过程中失败，会导致消息丢失**。



3.1 **fanout** —— 广播交换机

这种类型的交换机把所收到的信息路由到每一个队列当中

**提供者发布信息**

创建配置类并把交换机和队列的信息进行注册

```java
@Confiquration
public class FanoutConfig {
	// 声明FanoutExchange交换机
    @Bean
    public FanoutExchange fanoutExchange(){
	    return new FanoutExchange("itcast.fanout");
    }
    //声明第1个队列
    @Bean
    public Queue fanoutQueve1(){
 	   return new Queue("fanout.queue1");
    }
    //绑定队列1和交换机
    @Bean
    public Binding bindingQueue1(Queue fanoutQueue1,FanoutExchange fanoutExchange){
    	return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }
    // ···
    // 以相同方式声明第2个队列，并完成绑定
}
```

**消费者消费信息**

```java
@RabbitListener(queues = "fanout.queue1")
public void listenFanoutQueue1(String msg) {
	System.out.println("消费者1接收到Fanout消息：【"+msg+"】");
}

@RabbitListener(queues = "fanout.queue2")
public void listenFanoutQueue2(String msg){
	System.out.println("消费者2接收到Fanout消息：【"+msg+"】");
}
```

 在**提供者服务**中发送信息到**交换机**

```java
@Test
public void testFanoutExchange(){
	// 队列名称
	String exchangeName = "itcast.fanout";
	//消息
	String message = "hello,everyone!";
	//发送消息，参数分别是.交互机名称、RoutingKey(暂时为空)、消息
	rabbitTemplate.convertAndSend(exchangeName,"",message);
}
```

按照上方的流程编写好，就写好了一个由fanout交换机实现的发布订阅demo



3.2 **directexchange** —— 路由模式交换机

与fanout广播不同，dierect模式下会将接收到到的信息，按照规则路由到指定的队列当中。被称为路由模式。

- 每一个Queue都与Exchange会设置一个**BindingKey**
- 而发布者发送消息时，也需要指定消息的**RoutingKey**
- 过程中，交换机通过比较两个key。将其路由到对应的规则当中

同时，由于是根据两个key进行配对的，当多个队列都具有相同的key时，此`directExchange`也等价于`fanoutExchange`

![image-20240319160200311](.\images\image-20240319160200311.png)

在消费者监听信息板块 我们可以改变原有使用`@Bean`方式注入，而是改为采用`@RabbitListener`注入

```java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue1"),
    exchange = @Exchange(name = "exp.direct",type = ExchangeTypes.DIRECT),
    key = {"red","blue"}
))
public void listenDirectQueue1(String msg){
	System.out.println("消费者1接收到Direct消息：【"+msg+"】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue2"),
    exchange = @Exchange(name = "exp.direct",type = ExchangeTypes.DIRECT),
    key = {"red","yellow"}
))
public void listenDirectQueue2(String msg){
	System.out.println("消费者2接收到Direct消息：【"+msg+"】");
}
```

在`@RabbitListener`注解中，我们通过`binding`选项定义需要绑定的队列。绑定队列可以使用另一注解`@QueueBinding`。

如上，需要绑定队列`@Queue`，交换机`@Exchange`，`key`则是上方所说的`BindingKey`和`RoutingKey`



3.3 topicExchange

TopicExchange.与DirectExchange类似，区别在于routingKey必须是多个单词的列表，并且以`.`分割。

Queue与Exchange指定BindingKey时可以使用通配符 `#` 和 `*`

![image-20240319165437956](.\images\image-20240319165437956.png)

这部分的代码与上方directExchange没有大差别，就只是把`RoutingKey & BindingKey`修改为通配符的格式罢了。

```java
@RabbitListener(bindings = @QueveBinding(
    value = @Queue(name = "topic.queue1"),
    exchange = @Exchange(name = "itcast.topic",type = ExchangeTypes.TOPIC),
    key = "china.#"
))
public void listenTopicQueve1(String msg){
	System.out.println("消费者1接收到Topic消息：【"+msg+"】");
}
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue2"),
    exchange = @Exchange(name = "itcast.topic",type = ExchangeTypes.TOPIC),
    key = "#.news"
))
public void listenTopicQueue2(String msg){
	System.out.println("消费者2接收到Topici消息：【"+msg+"】");
}
```



### 消息转换器

在RabbitMQ中，我们可以不仅可以发送字符串，也可以发送任意类型的对象。但是这里发送的原理默认是将**字符串序列化**后发送。序列化的方式耗时长，并且序列化后生成的字符串长度较长，放入消息队列中传输效率也低。所以我们可以通过自定义他的转换规则，提高整体传输的效率。修改为JSON序列化就是一种解法。

我们可以找到Spring中对消息处理的位置 `org.springframework.amqp.support.converter.MessageConverter`

如果要修改只需要定义一个`MessageConverter`的Bean即可

（**一旦声明就会将原有配置覆盖** —— 自动装配的原理）

由于修改响应类型之后，无论是提供者和消费者都需要用到，所以可以直接把依赖放入父工程当中。

首先需要引入`fastjson`依赖

```java
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.10</version>
</dependency>
```

在提供者服务处声明对应的Bean

```java
@Bean
public MessageConverter jsonMessageConverter(){
	return new Jackson2JsonMessageConverter();
}
```



ps: 修改完之后能在RabbitMQ的GUI中看到对应的响应信息变化

修改前：

![image-20240319173307679](.\images\image-20240319173307679.png)

修改后：

![image-20240319172316915](.\images\image-20240319172316915.png)





**消费者处修改**

提供者只需要创建对应bean覆盖规则就好了，因为默认`rabbitTemplate.convertAndSend()`方法中就允许发送任意类型对象的值。

但是作为消费者，在引入依赖创建Bean之后，我们需要修改监听的规则，从而能对json反序列化后的结果进行映射。

```java
@RabbitListener(queues = "object.queue")
public void listenObjectQueue(Map<String,Object> msg){
	System.out.println("接收到object.queue的消息："+msg);
}
```

!!! **一定需要确定提供者和消费者双方的MessageConverter是相同的。**





## EleasticSearch

开源分布式搜索引擎，可实现 **搜索** 、 **日志统计** 、 **分析** 、 **系统监控等功能** 

### ELK

以elasticsearch为核心的技术栈，包括beats,Logstash,kibana,es

**Lucene**

 es基本的搜索api



### 倒排索引

此处先引入`文档`与`词条`的概念

- **文档**：每一条数据就是一个文档
- **词条**：将文档中的词语按照语义分词，得到的就是词条

一般的搜索方式根据id检索，称为”**正向索引**“，当需要搜索的字符串比较复杂的时候，正向索引只能根据id一条一条的进行筛选。效率较低。

针对此查询效率较低的问题，开发了`倒排索引`。按文档中原有语义**分词**，以分词后的词条创建索引。并在同时记录对应文档的信息（id信息）。这一方法在查询的时候，首先会通过原句各词条挑选出对应的文档id，后返回**更符合要求的**文档。



![image-20240322193029118](.\images\image-20240322193029118.png)



### 文档

elasticsearch是面向文档存储的。严格意义上属于`NoSql`，类似于mongoDB等的存储方式。对应的文档数据会被序列化为json后存储在es中。

![image-20240322194335602](.\images\image-20240322194335602.png)

在某种意义上，es和mysql等传统关系型数据库具有一定的共通点

| MySQL  | Elasticsearch | 说明                                                         |
| ------ | ------------- | ------------------------------------------------------------ |
| Table  | Index         | 索引(index),就是文档的集合，类似数据库的表(table)            |
| Row    | Document      | 文档(Document),就是一条条的数据，类似数据库中的行(Row),文档都是JS0N格式 |
| CoLumn | Field         | 字段(Field),就是JS0N文档中的字段，类似数据库中的列(Column)   |
| Schema | Mapping       | Mapping(映射)是索引中文档的约束，例如字段类型约束。类似数据库的表结构（Schema) |
| SQL    | DSL           | DSL是elasticsearch提供的JS0N风格的请求语句，用来操作elasticsearch,实现CRUD |

从上面表格可以知道es确实具有一定的存储能力，但是其nosql的性质和其他特点决定了他在保护数据的**安全与一致性**上不如Mysql。在实际开发中，我们一般还是利用es进行数据的搜索分析和计算。存储相关的任务还是给到Mysql等传统数据库。

但是如何保证es中待搜索的数据与mysql中已包含的数据之间的一致性呢？两者之间存在对应的数据同步内容，后文将会介绍。

![image-20240322195414659](.\images\image-20240322195414659.png)



### 部署 安装 运行

详细安装教程可以参照另一md 此处仅介绍如何部署单点es

同样可以直接使用docker进行拉取

```bash
docker pull elasticsearch
```

我此处是直接docker来load本地文件中的镜像

![image-20240322200908001](.\images\image-20240322200908001.png)



创建网络组

```bash
docker network create es-net
```

运行docker命令，启动docker容器

```bash
docker run -d \
	--name es1 \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -v es-data:/usr/share/elasticsearch/data \
    -v es-plugins:/usr/share/elasticsearch/plugins \
    --privileged \
    --network es-net \
    -p 9200:9200 \
    -p 9300:9300 \
elasticsearch:7.12.1
```

- `-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" `设置jvm内存堆大小 因为es是基于java实现的，在运行前需要提前设定他能使用的内存大小，这里是修改成512m
- `-e "discovery.type=single-node" `配置运行模式，此处是将es配置为单点模式下运行。
- `-v es-data:/usr/share/elasticsearch/data \
      -v es-plugins:/usr/share/elasticsearch/plugins`设置es存放数据的目录以及存放插件的目录。

- `--privileged`：授予逻辑卷访问权
- `--network es-net`使用的网络组 可修改为自定义网络组
- `-p 9200:9200`暴露给用户访问的http使用的端口
- `-p 9300:9300 `es中各个容器节点互联时使用的端口



成功启动容器之后可以通过`docker ps`查看运行中的容器是否含有对应的es

```

[root@localhost home]# docker ps
CONTAINER ID   IMAGE                  COMMAND                   CREATED         STATUS         PORTS                                                                                  NAMES
00909ae6cfdb   elasticsearch:7.12.1   "/bin/tini -- /usr/l…"   7 seconds ago   Up 6 seconds   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300->9300/tcp, :::9300->9300/tcp   es1
```



在成功运行容器后，可以在主机输入对应的`ip+端口`，若浏览器返回下方json信息，则说明es启动成功。

![image-20240322203039290](.\images\image-20240322203039290.png)



接下来，我们需要安装操控es的GUI界面，即kibana

同样使用docker在网络上拉取包或是加载本地的镜像

```bash
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://es1:9200 \
--network=es-net \
-p 5601:5601  \
kibana:7.12.1
```

- `-e ELASTICSEARCH_HOSTS=http://es:9200`设置环境变量，设置es的主机名，若和对应的es在同一个网络组当中，便可以类似上方直接以es容器的名称作为ip进行设置。
- `--network`设置网络组，一定要与es在同一个网络组当中，不然无法连接，上方环境变量同理。

同理，启动之后也可以通过`docker ps`来观察进程情况

```bash
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE                  COMMAND                   CREATED         STATUS          PORTS                                                                                  NAMES
a5dd1cfc3b89   kibana:7.12.1          "/bin/tini -- /usr/l…"   6 seconds ago   Up 5 seconds    0.0.0.0:5601->5601/tcp, :::5601->5601/tcp                                              kibana
26d99c8094b5   elasticsearch:7.12.1   "/bin/tini -- /usr/l…"   2 hours ago     Up 53 seconds   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300->9300/tcp, :::9300->9300/tcp   es
```



成功启动容器后，也可以通过映射的端口来进入此kibana的界面

![image-20240322221359596](.\images\image-20240322221359596.png)
