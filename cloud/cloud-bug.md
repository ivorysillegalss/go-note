```java
2024-03-07 10:32:36.419  WARN 3228 --- [           main] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanDefinitionStoreException: Failed to read candidate component class: file [D:\Idea\IdeaProject\eurukademo\target\classes\com\example\eurukademo\EurukademoApplication.class]; nested exception is org.springframework.core.NestedIOException: ASM ClassReader failed to parse class file - probably due to a new Java class file version that isn't supported yet: file [D:\Idea\IdeaProject\eurukademo\target\classes\com\example\eurukademo\EurukademoApplication.class]; nested exception is java.lang.IllegalArgumentException: Unsupported class file major version 61
```

jdk版本过高不支持





```
2024-03-07 11:14:02.513  INFO 6112 --- [           main] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_UNKNOWN/192.168.2.7 - was unable to refresh its cache! This periodic background refresh will be retried in 30 seconds. status = Cannot execute request on any known server stacktrace = com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server



2024-03-07 11:14:02.512  INFO 6112 --- [           main] c.n.d.s.t.d.RedirectingEurekaHttpClient  : Request execution error. endpoint=DefaultEndpoint{ serviceUrl='http://localhost:8761/eureka/}, exception=java.net.ConnectException: Connection refused: no further information stacktrace=com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: Connection refused: no further information



2024-03-07 11:14:02.645  WARN 6112 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_UNKNOWN/192.168.2.7 - registration failed Cannot execute request on any known server

com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
	at com.netflix.discovery.shared.transport.decorator.RetryableEurekaHttpClient.execute(RetryableEurekaHttpClient.java:112) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$1.execute(EurekaHttpClientDecorator.java:59) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.shared.transport.decorator.SessionedEurekaHttpClient.execute(SessionedEurekaHttpClient.java:77) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.DiscoveryClient.register(DiscoveryClient.java:876) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.InstanceInfoReplicator.run(InstanceInfoReplicator.java:121) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.InstanceInfoReplicator$1.run(InstanceInfoReplicator.java:101) ~[eureka-client-1.10.11.jar:1.10.11]
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:577) ~[na:na]
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:317) ~[na:na]
	at java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304) ~[na:na]
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1144) ~[na:na]
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642) ~[na:na]
	at java.base/java.lang.Thread.run(Thread.java:1589) ~[na:na]

2024-03-07 11:14:02.645  WARN 6112 --- [nfoReplicator-0] c.n.discovery.InstanceInfoReplicator     : There was a problem with the instance info replicator

com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
	at com.netflix.discovery.shared.transport.decorator.RetryableEurekaHttpClient.execute(RetryableEurekaHttpClient.java:112) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$1.execute(EurekaHttpClientDecorator.java:59) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.shared.transport.decorator.SessionedEurekaHttpClient.execute(SessionedEurekaHttpClient.java:77) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.DiscoveryClient.register(DiscoveryClient.java:876) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.InstanceInfoReplicator.run(InstanceInfoReplicator.java:121) ~[eureka-client-1.10.11.jar:1.10.11]
	at com.netflix.discovery.InstanceInfoReplicator$1.run(InstanceInfoReplicator.java:101) ~[eureka-client-1.10.11.jar:1.10.11]
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:577) ~[na:na]
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:317) ~[na:na]
	at java.base/java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:304) ~[na:na]
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1144) ~[na:na]
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642) ~[na:na]
	at java.base/java.lang.Thread.run(Thread.java:1589) ~[na:na]

```

以上等等报错 把配置文件从yml换成properties就可以了

```yaml
server:
  port: 10086
spring:
  application:
    name: eurekaserver
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```

上变下

```properties
server.port=10086
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
spring.application.name=eurekaserver
eureka.client.service-url.defaultZone=http://127.0.0.1:10086/eureka
```

感觉是因为没有把yml识别成配置文件所导致的