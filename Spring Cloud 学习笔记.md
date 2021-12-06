# Spring Cloud 学习笔记

## 一、环境准备

### 软件版本

​	springcloud Hoxtom.SR1

​	springboot 2.2.2

​	cloud alibaba 2.1.0

​	java8

​	maven 3.5+

​	mysql 5,7+

​	参考官网查看版本匹配

### maven

```maven
<dependencyManagement>和<dependencies>的区别
```

dependencyManagement元素能让所有的子项目中引用一个依赖二不用显式的列出版本号。maven会从子类往父类找dependencyManagement中对应的版本号，子模块中可以不指定版本号。

**dependencyManagement只声明依赖并不引入依赖，在子模块使用时引入。**

maven中跳过单元测试

![image-20211026214856144](C:\Users\wy\Desktop\笔记\images\image-20211026214856144.png)

~~error：上次看到P8 10:34 application.yml配置文件没有被项目识别 **（原因：maven在创建子模块时还没有创建完成）**~~



**学习新框架内容，官网api文档和github相关项目的wiki**



## 二、服务注册中心

### Eureka

在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。

![image-20211027191907309](C:\Users\wy\Desktop\笔记\images\image-20211027191907309.png)

#### Eureka单机配置

**Eureka包含两个组件:Eureka Server和Eureka Client**
**Eureka Server提供服务注册服务**
各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

```yml
eureka:
  client:
    register-with-eureka: false   #是否将自己注册到注册中心
    fetch-registry: false    #是否从服务端抓取已有的注册信息
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #,http://eureka7002.com:7002/eureka
  instance:
    hostname: localhost
```

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

**EurekaClient通过注册中心进行访问**
是一个lava客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒)



客户端需要添加yml配置和pom.xml引入客户端

```yml
spring:
  application:
    name: cloud-order-service   #
eureka:
  client:
    register-with-eureka: true   #是否将自己注册到注册中心,集群必须设置为true配合ribbon
    fetch-registry: true    #是否从服务端抓取已有的注册信息
    service-url:
      defaultZone: http://localhost:7001/eureka #,http://eureka7002.com:7002/eureka
  instance:
    instance-id: payment8001
    prefer-ip-address: true  #访问路径可以显示IP地址
    lease-renewal-interval-in-seconds: 1  #向服务端发送心跳的时间间隔，单位为秒（默认是30秒）
    lease-expiration-duration-in-seconds: 2 #收到最后一次心跳后等待时间上限，单位为秒（默认是90秒），超时将剔除
```

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### Eureka集群配置

**EurekaServer集群配置**

为子模块cloud-eureka-server7001和cloud-eureka-server7002修改yml配置

```yml
server:
  port: 7001   

eureka:
  client:
    register-with-eureka: false   #是否将自己注册到注册中心
    fetch-registry: false    #是否从服务端抓取已有的注册信息
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/ #,http://eureka7001.com:7001/eureka
  instance:
    hostname: eureka7001.com #eureka7001.com    #本机修改hosts文件
```

**EurekaClient集群配置**

需要把多个EurekaServer都写进配置中

```yml
eureka:
  client:
    register-with-eureka: true   #是否将自己注册到注册中心,集群必须设置为true配合ribbon
    fetch-registry: true    #是否从服务端抓取已有的注册信息
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

​	服务生产模块注册多个服务进入eureka（但是name相同）

```yml
spring:
  application:
    name: cloud-payment-service
eureka:
  client:
    register-with-eureka: true   #是否将自己注册到注册中心,集群必须设置为true配合ribbon
    fetch-registry: true    #是否从服务端抓取已有的注册信息
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

​	服务消费模块的Controller不指定特定的服务端口（把单机的固定端口改成eureka注册的服务名称）

```java
//    public static final String PAYMENT_URL="http://localhost:8001";
    public static final String PAYMENT_URL="http://CLOUD-PAYMENT-SERVICE";
```

​	并且开启RestTemplate的负载均衡（@LoadBalanced）默认采用轮询方式

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced   //赋予RestTemplate负载均衡的能力
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
    //等同越在application.xml 中添加<bean id=”“ class="">  把类注入到spring boot中

}
```

~~今天完成到了P22，使用Eureka实现了集群的微服务。~~



### Discovery服务发现

```
@EnableDiscoveryClient   //application中开启服务发现
```

Controler中注入discoveryClinet实体类

```java
import org.springframework.cloud.client.discovery.DiscoveryClient;


	@Resource
	private DiscoveryClient discoveryClient;

    @GetMapping("/payment/discovery")
    public Object discovery(){
        //获得服务列表
        List<String> services = discoveryClient.getServices();
        for(String element : services){
            log.info("*********element: "+ element);
        }

        //获得指定服务的集群信息  
        List<ServiceInstance>instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for(ServiceInstance instance : instances){
            log.info("********"+instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());

        }
        return this.discoveryClient;
    }
```

### Zookeeper注册中心

Zookeeper是一个分布式协调工具，可以实现注册中心功能

Zookeeper集群只需在yml配置文件中，添加多个注册中心地址

pom.xml配置

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
</dependency>

解决包冲突
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.4.9</version>
    </dependency>
```

application.yml配置

```yml
spring:
  application:
    name: cloud-provider-order
  cloud:
    zookeeper:
      connect-string: 172.27.98.87:2181
```

使用时再main函数添加@EnableDiscoveryClient 注解

~~今天完成P30，了解了zookeeper的使用~~

### Consul

pom.xml配置

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

application.yml配置

```yml
spring:
  #  application:
  #    name: zookeeper-order-payment
  #  cloud:
  #    zookeeper:
  #      connect-string: 172.27.98.87:2181
  application:
    name: consul-order-payment
  cloud:
    consul:
      host: 172.27.98.87
      port: 8500
      discovery:
        service-name: ${spring.application.name}
        prefer-ip-address: true    #设置host为首选地址
        heartbeat:
          enabled: true #发送心跳包
```

使用时再main函数添加@EnableDiscoveryClient 注解

### 三个注册服务中心的不同

![image-20211103102621789](C:\Users\wy\Desktop\笔记\images\image-20211103102621789.png)

#### CAP

​	一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance）（分布式架构一定要保证）

**AP**

​	当网络分区出错后，为了保证可用性，系统B可以先放回旧值，保证系统的可用性。

​	![image-20211103103421019](C:\Users\wy\Desktop\笔记\images\image-20211103103421019.png)

Euraka来说，当一个服务节点网络中断，Euraka不会立即删除节点

**CP**

​	当网络分区出错后，为了保证一致性，必须拒绝请求，否则无法保证一致性。

![image-20211103103703633](C:\Users\wy\Desktop\笔记\images\image-20211103103703633.png)

Consul和Zookeeper当节点心挑检测失败会直接删除节点

Consul和Zookeeper保证高一致性和分区容错性，Eureka保证高可用性和分区容错性

Euraka和Consul有web界面





## 三、服务调用

### 负载均衡

​	本地负载均衡：（RIbbon）

​			在调用微服务接口的时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程调用。

​	服务器负载均衡：（Nginx）

​			客户端的所有请求都会交给nginx，然后由nginx实现转发请求。由服务器端实现负载均衡

### Ribbon

提高客户端软件的负载均衡算法和服务调用（在配置中添加@LoadBalancer）

pom.xml(引入eureka的时候就自动引入了)

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

![image-20211103110758829](C:\Users\wy\Desktop\笔记\images\image-20211103110758829.png)

### RestTemplate

​	getForObject:返回json数据

```java
public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
    return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id, CommonResult.class);
}
```

​	getForEntity：返回对象为ResponseEntity对象，包含响应中的一些重要信息，响应头、响应状态码、响应体等

```java
public CommonResult<Payment> getPayment2(@PathVariable("id") Long id){
    ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_URL+"/payment/get/"+id, CommonResult.class);
    if(entity.getStatusCode().is2xxSuccessful()){
        return entity.getBody();
    }else{
        return new CommonResult<>(444,"查询错误");
    }
```

​	postForObject

​	postForEntity

### IRule根据特定算法从服务列表中选取服务

#### com.netflix.loadbalacer.RoundRobinRule 轮询

#### com.netflix.loadbalacer.RandomRule 随机

#### com.netflix.loadbalacer.RetryRule 

​	先按照RoundRobinRule的策略获取服务，如果获取的服务失败则在指定时间内会进行重试，获取可用的服务

#### WeightedResponseTimeRule

​	对RoundRObinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择

#### BestAvailableRule

​	会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务

#### AvialiabilityFilteringRule

​	选过滤掉故障实例，在选择并发较小的实例

#### ZoneAvoidanceRule

​	复合判断server所在区域的性能和server的可用性选择服务器

### 替换默认负载规则

默认使用轮询算法

官方建议配置类不与@ComponentScan注解放一起，以达到特殊定制的需求。所以配置类不能放在Main函数目录下面因为@SpringBootApplication注解中包含@ComponentScan

![image-20211103151221686](C:\Users\wy\Desktop\笔记\images\image-20211103151221686.png)

**创建配置类**

```java
@Configuration
public class MySelfRule {

    @Bean
    public IRule myRule(){
        return new RandomRule();  //改成随机访问
    }
}
```

并且在Main函数中**添加注解**@RibbonClient(name = "CLOUD-PAYMENT-SERVICE", configuration = MySelfRule.class)

```java
@SpringBootApplication
@EnableDiscoveryClient

@RibbonClient(name = "CLOUD-PAYMENT-SERVICE", configuration = MySelfRule.class)
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class, args);
    }
}
```

### 自己实现Ribbon负载均衡算法

#### **轮询算法原理**

​	获取注册中心中的服务列表

​	实际调用的服务器下标 = Rest接口第几次请求%服务器集群总数量 （每次重启服务后rest接口计数从1开始）

#### **RoundRobinRule源码**

```java
package com.netflix.loadbalancer;

import com.netflix.client.config.IClientConfig;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
public class RoundRobinRule extends AbstractLoadBalancerRule {
    private AtomicInteger nextServerCyclicCounter;		//服务计数器
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;
    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public RoundRobinRule() {
        this.nextServerCyclicCounter = new AtomicInteger(0);  //服务计数器初始化0
    }

    public RoundRobinRule(ILoadBalancer lb) {
        this();
        this.setLoadBalancer(lb);
    }

    //每次请求服务时候调用choose方法选择调用的服务器
    //ILoadBalancer为所有
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        } else {
            Server server = null;  //记录找到的服务
            
            int count = 0;  //查找可用服务次数

            while(true) {
                if (server == null && count++ < 10) {
                    //如果当前服务为空，并且查找次数小于10,继续寻找
                    List<Server> reachableServers = lb.getReachableServers(); //当前可以用服务器总数
                    List<Server> allServers = lb.getAllServers();  //注册中心列表中的当前服务器的所有服务器总数（保证注册中心当前服务是在注册中心中注册了的）
                    int upCount = reachableServers.size();
                    int serverCount = allServers.size();
                    if (upCount != 0 && serverCount != 0) {
                        //有可用的服务器
                        int nextServerIndex = this.incrementAndGetModulo(serverCount);  //得到选择服务器的下标
                        server = (Server)allServers.get(nextServerIndex); //获取选择的服务服务
                        if (server == null) {
                            Thread.yield();
                        } else {
                            if (server.isAlive() && server.isReadyToServe()) {
                                return server;
                            }

                            server = null;
                        }
                        continue;
                    }

                    log.warn("No up servers available from load balancer: " + lb);
                    return null;
                }

                if (count >= 10) {
                    //默认认为查找次数超过10次为查找失败
                    log.warn("No available alive servers after 10 tries from load balancer: " + lb);
                }

                return server;
            }
        }
    }

    private int incrementAndGetModulo(int modulo) {
        int current;
        int next;
        do {
            current = this.nextServerCyclicCounter.get();//从共享内存中读取最新值。
            next = (current + 1) % modulo;
        } while(!this.nextServerCyclicCounter.compareAndSet(current, next));

        return next;
    }

    public Server choose(Object key) {
        return this.choose(this.getLoadBalancer(), key);
    }

    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}
```

#### 手写轮询算法

```java
package com.springcloud.lb;

import com.netflix.appinfo.InstanceInfo;
import org.springframework.cloud.client.ServiceInstance;

import java.util.List;

/**
 * @auther wy
 * @create 2021/11/3 19:34
 */
public interface LoadBalance {
    ServiceInstance instance(List<ServiceInstance> serviceInstances);
}
```

```java
package com.springcloud.lb;

import com.netflix.appinfo.InstanceInfo;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.concurrent.atomic.AtomicReference;

/**
 * @auther wy
 * @create 2021/11/3 19:36
 */

@Component
public class MyLb implements LoadBalance{

    private AtomicReference<Integer> atomicReference = new AtomicReference<>(0);

    @Override
    public ServiceInstance instance(List<ServiceInstance> serviceInstances) {
        int serviceInstanceSize = serviceInstances.size();
        int nextIndex = getAndIncrement()%serviceInstanceSize;
        return serviceInstances.get(nextIndex);
    }
    public final int getAndIncrement(){
        int value;
        int next;
        do{
            value = atomicReference.get();
            next = value >= Integer.MAX_VALUE? 0 : value + 1;
        }while (!atomicReference.compareAndSet(value, next));
        return next;
    }
}
```

```java
@GetMapping("/consumer/payment/lb")
public String testLb(){
    List<ServiceInstance> serviceInstanceList = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
    if(serviceInstanceList == null || serviceInstanceList.size()<=0){
        return "没有可用服务";
    }
    ServiceInstance serviceInstance = myLoadBalance.instance(serviceInstanceList);
    URI uri = serviceInstance.getUri();
    return restTemplate.getForObject(uri +"/payment/lb", String.class);
}
```



### OpenFeign

  一个声明式的Web服务客户端，让编写Web服务器客户端变得非常容易，只需创建一个接口并在接口上添加注解即可。
	Feign旨在使编写Java Http客户端变得更容易。
前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，**往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用**。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，**我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)**，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

Feign集成了Ribbon
	利用Ribbon维护了Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，通过feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用。



使用在Main中添加@EnableFeignClients

编写service.xxxServcie.java接口，定义接口为需要远程调用的生产者服务，在controller中调用接口方法。

![image-20211103205606361](C:\Users\wy\Desktop\笔记\images\image-20211103205606361.png)



#### 超时控制

默认Feign客户端只等待一秒钟，但是服务端处理需要超过1秒钟，导致Feign客户端不想等待了，直接返回报错。为了避免这样的情况，有时候我们需要设置Feign客户端的超时控制。yml中添加ribbon配置。

```yml
ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000
```

#### 开启日志打印

```java
package com.springcloud.cfgBeans;

import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @auther wy
 * @create 2021/11/3 21:07
 */
@Configuration
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}
```

添加yml配置

```yml
logging:
  level:
    com.sprongcloud.service.PaymentFeignService: debug  //类名不要手打，直接复制
```



## 四、服务降级

服务雪崩
多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”.对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。所以，通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。|



### Hystrix

Hystrix是一个用于处理分布式系统**的延迟和容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，**不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。**
“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝)，**向调用方返回一个符合预期的、可处理的备选响应**（FallBack)，**而不是长时间的等待或者抛出调用方无法处理的异常**，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

#### 服务降级（Fallback）

​	当服务器繁忙时，需要稍后再试，不让客户端等待并立刻放回一个提示，fallback

会出现服务降级的情况：

​		程序运行异常

​		超时

​		服务熔断触发服务降级

​		线程池/信号量打满也会导致服务降级

#### 服务熔断（break）

​	类比保险丝达到最大服务访问后，直接拒绝访问，限电跳闸，然后调用服务降级的方法并返回提示。服务有响应后，逐渐恢复服务。

#### ~~服务限流（flowlimit）~~

​	秒杀等高并发操作，莫一时刻大量用户涌入，排队等待，一秒N个服务，有序进行。

#### 接近实时的监控



#### 服务降级配置

@HystrixCommand



## 五、网关

### ~~Zuul~~

### gateway

动态路由：能够匹配任何请求属性

可以对路由指定Predicate（断言）和Filter（过滤器）

集成Hystrix的断路器功能

集成Spring cloud服务发现功能

易于编写的Predicate（断言）和Filter（过滤器）

请求限流功能

支持路径重写



## 六、配置中心

​	微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以—套集中式的、动态的配置管理设施是必不可少的。
SpringCloud提供了ConfigServer来解决这个问题，我们每一个微服务自己带着一个application.yml，上百个配置文件的管理....



### 动态刷新

```
curl -X POST "http://localhost:3355/actuator/refresh"
```

## 七、消息总线

在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便地广播─些需要让其他连接在该主题上的实例都知道的消息。
基本原理
ConfigClient实例都监听MQ中同一个topic(默认是springCloudβus)。当一个服务刷新数据的时候，它会把这个信息放入到Topic中这样其它监听同一—Topic的服务就能得到通知，然后去更新自身的配置。

#### Erlang安装

下载并安装，配置系统环境

添加ERLANG_HOME到环境变量中

![image-20211105203857100](C:\Users\wy\Desktop\笔记\images\image-20211105203857100.png)

并且在Path变量中添加

![image-20211105203925591](C:\Users\wy\Desktop\笔记\images\image-20211105203925591.png)

#### Rabbitmq安装

下载解压，在cmd命令下进入软件目标sbin下输入rabbitmq-plugins enable rabbitmq_management，启用管理平台（添加管理插件）rabbitmq-plugins disable rabbitmq_management 关闭管理界面

![image-20211105204106929](C:\Users\wy\Desktop\笔记\images\image-20211105204106929.png)

![image-20211105204207316](C:\Users\wy\Desktop\笔记\images\image-20211105204207316.png)

启动服务即可。

http://localhost:15672/  访问图像界面（添加管理插件才能访问）

登录账号密码默认为guest， guest



动态全局刷新

```
curl -X POST "http://localhost:3344/actuator/bus-refresh"
```



#### RabitMQ消息中间件

![image-20211106133348870](C:\Users\wy\Desktop\笔记\images\image-20211106133348870.png)

用户请求操作，只需要完成关键操作即可返回响应，剩下的操作直接放入到消息队列中，由消息中间价异步去完成，异步响应。





#### Spring cloud Sream

![image-20211106130255143](C:\Users\wy\Desktop\笔记\images\image-20211106130255143.png)

![image-20211106130357251](C:\Users\wy\Desktop\笔记\images\image-20211106130357251.png)

##### 消息生产者





##### 消息消费者





##### 分组消费与持久化

​	当有多个消费者时，存在多个消费问题，用消费分组，解决重复消费问题。

###### 自定义配置分组

spring.cloud.stream.bindings.input.group：XXX

![image-20211106145235169](C:\Users\wy\Desktop\笔记\images\image-20211106145235169.png)

设置到同一个组避免重复消费



###### 消息持久化

消费者经常关掉，重启后依旧可以重消息队列中拿去历史消息



## 八、链路监控

Sleuth

Zipkin



## 九、spring cloud alibaba

### nacos

可以做服务注册中心和配置中心

```shell
startup.cmd -m standalone   #运行单机版
```

#### nacos集群和持久化

nacos配置数据存储在内嵌数据库（derby）中，但是在集群环境中，多个内嵌数据库无法保证数据的一致性，所以nacos支持外部数据库Mysql，集群环境直接在mysql中获取配置数据。

##### 1. 配置集群配置文件

在nacos的解压目录nacos/的conf目录下，有配置文件cluster.conf，请每行配置成ip:port。（请配置3个或3个以上节点）

```plain
# ip:port
200.8.9.16:8848
200.8.9.17:8848
200.8.9.18:8848
```

##### 2. 确定数据源

###### 使用内置数据源

无需进行任何配置

##### 使用外置数据源

生产使用建议至少主备模式，或者采用高可用数据库。

###### 初始化 MySQL 数据库

[sql语句源文件](https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql)

###### application.properties 配置

[application.properties配置文件](https://github.com/alibaba/nacos/blob/master/distribution/conf/application.properties)

##### 3. 启动服务器

###### Linux/Unix/Mac

###### Stand-alone mode

```bash
sh startup.sh -m standalone
```

#### 集群模式

> 使用内置数据源

```bash
sh startup.sh -p embedded
```

> 使用外置数据源

```bash
sh startup.sh
```

![image-20211108144453881](C:\Users\wy\Desktop\笔记\images\image-20211108144453881.png)



### Sentinel

Hystrix
1)需要我们自己手工搭建监控平台
2)没有一套web界面可以给我们进行更加细粒度化得配置流控、速率控制、服务熔断、服务降级等

![image-20211108145014630](C:\Users\wy\Desktop\笔记\images\image-20211108145014630.png)



#### 流量控制

资源名:唯一名称，默认请求路径

针对来源: Sentinel可以针对调用者进行限流，填写微服务名，默认default (不区分来源)·阈值类型/单机阈值:
		**QPS**(每秒钟的请求数量)∶当调用该api的QPS达到阈值的时候，进行限流。  (**银行只让n个人进来**)

​		**线程数**:当调用该api的线程数达到阈值的时候，进行限流。      					（**银行只开n个办理窗口**）

是否集群:不需要集群。

流控模式:

​	直接: api达到限流条件时，直接限流

​	关联:当关联的资源达到阈值时，就限流自己		（**支付模块达到上限，限流下单模块**）

​	链路:只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流)【api级别的针对来源】
流控效果:
​	快速失败:直接失败，抛异常

​	Warm Up:根据codeFactor (冷加载因子，默认3)的值，从阈值/codeFactor，经过预热时长，才达到设置的QPS阈值

#### 服务降级

sentinel没有hystrix的半开功能（当熔断后，自动失败服务是否可以用，当可用请求达到阈值后，自动恢复）

RT(平均响应时间，秒级)

​	平均响应时间超出阈值且在时间窗口内通过的请求>=5，两个条件同时满足后触发降级窗口期过后关闭断路器

​	RT最大4900(更大的需要通过-Dcsp.sentinel.statistic.max.rt=XXXX才能生效)

异常比列(秒级)

​	QPS >= 5且异常比例(秒级统计）超过阈值时，触发降级;时间窗口结束后，关闭降级

异常数( DEGRADE_GRADE_EXCEPTION_CoUNT ):

​	当资源近1分钟的异常数目超过阈值之后会进行熔断。注意由于统计时间窗口是分钟级别的，若timewindow小于60s，则结束熔断状态后仍可能再进入熔断状态。

#### 热点规则 

 **@SentinelResource** 

![image-20211108161436896](C:\Users\wy\Desktop\笔记\images\image-20211108161436896.png)

```java
    @GetMapping("/testHotKey")
    @SentinelResource(value = "testHotKey", blockHandler = "deal_testHotKey")
    public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                             @RequestParam(value = "p2",required = false) String p2){
        return "testHotkey";
    }

    public String deal_testHotKey(String p1, String p2, BlockException exception){
        return "deal_testHotKey error";
    }
```

#### **限流处理函数与Controller解耦**

**blockHandlerClass参数**

```java
@GetMapping("/byResource")
@SentinelResource(value = "byResource",
        blockHandlerClass = CustomerBlockHandler.class,
        blockHandler = "handleException1")
public CommonResult byResorce(){
    return new CommonResult(200, "按资源名称限流测试ok", new Payment(2000L,"serial001"));
}
```

```java
package com.springcloud.handler;

import com.alibaba.csp.sentinel.slots.block.BlockException;
public class CustomerBlockHandler {


    public String handlerException1(String p1, String p2, BlockException exception){
        return "handlerException1";
    }
    public String handlerException2(String p1, String p2, BlockException exception){
        return "handlerException1";
    }
}
```

`@SentinelResource` 用于定义资源，并提供可选的异常处理和 fallback 配置项。 `@SentinelResource` 注解包含以下属性：

- `value`：资源名称，必需项（不能为空）

- `entryType`：entry 类型，可选项（默认为 `EntryType.OUT`）

- `blockHandler` / `blockHandlerClass`: `blockHandler` 对应处理 `BlockException` 的函数名称，可选项。blockHandler 函数访问范围需要是 `public`，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 `BlockException`。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `blockHandlerClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。

- ```
  fallback
  ```

  ：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了

   

  ```
  exceptionsToIgnore
  ```

   

  里面排除掉的异常类型）进行处理。fallback 函数签名和位置要求：

  - 返回值类型必须与原函数返回值类型一致；
  - 方法参数列表需要和原函数一致，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
  - fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注**意对应的函数必需为 static 函数**，否则无法解析。

- ```
  defaultFallback
  ```

  （since 1.6.0）：默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所以类型的异常（除了

   

  ```
  exceptionsToIgnore
  ```

   

  里面排除掉的异常类型）进行处理。若**同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效**。defaultFallback 函数签名要求：

  - 返回值类型必须与原函数返回值类型一致；
  - 方法参数列表需要为空，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
  - defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。

- `exceptionsToIgnore`（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。

#### Sentinel规则持久化

把流控规则存储到nacos中，nacos支持mysql存储。

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

```yml
spring:
  application:
    name: cloud-consumer-service

  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8719
      datasource:
        ds1:
          nacos:
            server-addr: localhost:8848
            dataID: ${spring.application.name}
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
```

![image-20211108213840046](C:\Users\wy\Desktop\笔记\images\image-20211108213840046.png)

把限流规则写进nacos中

## 10、分布式数据库事务问题

### Seata

分布式数据库，不同数据库之间没法保证数据一致性。

#### TC (Transaction Coordinator) - 事务协调者

维护全局和分支事务的状态，驱动全局事务提交或回滚。

#### TM (Transaction Manager) - 事务管理器

定义全局事务的范围：开始全局事务、提交或回滚全局事务。

#### RM (Resource Manager) - 资源管理器

管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

**XID全局唯一事务ID**

![](C:\Users\wy\Desktop\笔记\images\image-20211109151609268.png)

在相关业务中添加全局锁来保证事务的一致性

```java
@GlobalTransactional(name = "wy-create-order", rollbackFor = Exception.class)
public void create(Order order) {
log.info("-------------------开始新建订单——————————————————");
orderDao.create(order);

log.info("-------------------订单微服务修改库存——————————————————");
storageService.decrease(order.getProductId(), order.getCount());

log.info("-------------------账户微服务修改余额——————————————————");
accountService.decrease(order.getUserId(), order.getMoney());

log.info("-------------------修改订单状态——————————————————");
orderDao.update(order.getUserId(), 1);


}
```

![image-20211109212104791](C:\Users\wy\Desktop\笔记\images\image-20211109212104791.png)

before image：保存修改前的数据（回滚时使用）

after image：保存修改后的数据

分两个阶段：

​	一阶段：是个个分支本地执行sql  （这个阶段很快，因为个个数据库之间不会互相影响）

​	二阶段：判断事务整体是否有异常，只要有一个分支有异常就全部回滚，根据befor image来把修改了的数据改回来（这里需要判断是否脏读，如果其他进程修改过，那系统无法处理，只能人工根据数据手动修复）

​					否则正常完成，删除before image 和after image

TC：启动senta.service.机器

TM：标注GlobalTransactional的事务发起方

RM：各个分支数据库



## 11、全局ID问题

1、UUID

2、mysql数据库 id自增+replaceinfo

3、Redies   只增并设置增加步长

4、雪花算法



## 附录：

### 注解

#### jpa注解

使用Hibernate自动生成数据表

```java
//在entity类中添加这两个注解,并且设置主键
@Entity
@Table(name = "Payment")
public class Payment implements Serializable {
	
    @Id		//主键
    @GeneratedValue		//设置自增加
    private Long id;  //mysql中为bigint
```

并且要在yml中开启自动更新

```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: jdbc:mysql://172.27.98.87:3307/db2021?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 666666
  
  jpa:
  
    show-sql: true #执行的sql代码
    hibernate:
      ddl-auto: update  #开启自动更新数据库
```

#### lombok注解

```java
@Data   //添加@data 可以不用设置Getter,Setter,equals,canEqual,hasCode,toString方法
@AllArgsConstructor   //提供一个全参数构造函数
/*public test1(String name, String age, String sex) {
        this.name = name;
        this.age = age;
        this.sex = sex;
        }*/
@NoArgsConstructor   //提供一个无参数构造函数
/*
 public test1() {
    }
 */

@Slf4j  //private final Logger logger = LoggerFactory.getLogger(当前类名.class);
//可以省去日志实体化代码 直接使用log.info（）
```

### Mapper.xml 配置文件

```xml
<!--    useGeneratedKeys=“true”     keyProperty=“id”
    useGeneratedKeys设置为 true 时，表示如果插入的表id以自增列为主键，则允许 JDBC 支持自动生成主键，并可将自动生成的主键id返回。-->
    <insert id="create" parameterType="com.springcloud.entities.Payment" useGeneratedKeys="true" keyProperty="id">
        insert into payment(serial) values(#{serial});
    </insert>
```



### 开启Devtools 热部署

1.Adding devtolls to your project(子模块pom.xml)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

2.Adding plugin to your pom.xml(父模块pom.xml)

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
        <fork>true</fork>
        <addResources>true</addResources>
      </configuration>
    </plugin>
  </plugins>
</build>
```

3.Enable automatic build

![image-20211027161200614](C:\Users\wy\Desktop\笔记\images\image-20211027161200614.png)

4.update the value of

ctrl+shift+alt+/  选Resgistry

![image-20211027161327579](C:\Users\wy\Desktop\笔记\images\image-20211027161327579.png)

![image-20211027161608586](C:\Users\wy\Desktop\笔记\images\image-20211027161608586.png)

![image-20211027161828758](C:\Users\wy\Desktop\笔记\images\image-20211027161828758.png)

5. restart idea

   **注意：只在开放阶段启用**

### RestTemplate

（类似与httpclient）

使用restTemlate访问restful接口十分简单

（url, requestMap, ResponsBean.class）

Rest请求地址，请求参数，Http响应结果被转换成的对象类型

1.把RestTemplate注入到springboot中

```java
package com.springcloud.config;


import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {

    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
    //等同越在application.xml 中添加<bean id=”“ class="">  把类注入到spring boot中

}
```

2.在controller中使用

```java

public static final String PAYMENT_URL="http://localhost:8001";

@Resource
private RestTemplate restTemplate;  //实体化restTemplate

public CommonResult<Payment> create(Payment payment){
    return restTemplate.postForObject(PAYMENT_URL+"payment/create",payment, CommonResult.class);
}

```

### Run Dashboard

在.idea文件夹下的workspace.xml 中找到<compoent name="RunDashboard">在其中添加option内容

```xml
<component name="RunDashboard">
  <option name="configurationTypes">
    <set>
      <option value="SpringBootApplicationConfigurationType" />
    </set>
  </option>
</component>
```



### actuator

提供springboot的应用监控

| ID                 | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `auditevents`      | Exposes audit events information for the current application. Requires an `AuditEventRepository`bean. |
| `beans`            | Displays a complete list of all the Spring beans in your application. |
| `caches`           | Exposes available caches.                                    |
| `conditions`       | Shows the conditions that were evaluated on configuration and auto-configuration classes and the reasons why they did or did not match. |
| `configprops`      | Displays a collated list of all `@ConfigurationProperties`.  |
| `env`              | Exposes properties from Spring’s `ConfigurableEnvironment`.  |
| `flyway`           | Shows any Flyway database migrations that have been applied. Requires one or more `Flyway`beans. |
| `health`           | Shows application health information.                        |
| `httptrace`        | Displays HTTP trace information (by default, the last 100 HTTP request-response exchanges). Requires an `HttpTraceRepository` bean. |
| `info`             | Displays arbitrary application info.                         |
| `integrationgraph` | Shows the Spring Integration graph. Requires a dependency on `spring-integration-core`. |
| `loggers`          | Shows and modifies the configuration of loggers in the application. |
| `liquibase`        | Shows any Liquibase database migrations that have been applied. Requires one or more `Liquibase`beans. |
| `metrics`          | Shows ‘metrics’ information for the current application.     |
| `mappings`         | Displays a collated list of all `@RequestMapping` paths.     |
| `quartz`           | Shows information about Quartz Scheduler jobs.               |
| `scheduledtasks`   | Displays the scheduled tasks in your application.            |
| `sessions`         | Allows retrieval and deletion of user sessions from a Spring Session-backed session store. Requires a Servlet-based web application using Spring Session. |
| `shutdown`         | Lets the application be gracefully shutdown. Disabled by default. |
| `startup`          | Shows the [startup steps data](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.startup-tracking) collected by the `ApplicationStartup`. Requires the `SpringApplication` to be configured with a `BufferingApplicationStartup`. |
| `threaddump`       | Performs a thread dump.                                      |

添加pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### Docker

#### Docker镜像

对于使用 systemd 的系统，请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）：

```json
{"registry-mirrors":["https://docker.mirrors.ustc.edu.cn/"]}
```

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Docker常用命令

```shell
#下载镜像  :后面可指定版本
docker pull zookeeper:3.4.9
#查看所有镜像
docker images

#删除镜像
docker rmi id
#删除容器
docker rm name

#查询真正运行的容器
docker ps
#查询所有容器
docker ps -a

#开启容器
docker start zookeeper
#停止容器
docker stop zookeeper

#docker 进入mysql命令行
docker ps   #查看id
docker exec -it 89c5b9c81e74 bash 
mysql -uroot -pxxxx  #进入MySQL命令行
```



#### 1.Zookeeper

##### Zookeeper安装

```shell
#安装
docker pull zookeeper:3.4.9
#启动
docker run --privileged=true -d --name zookeeper --publish 2181:2181  -d zookeeper:3.4.9
```

##### Zookeeper本地连接

```shell
sudo docker ps # 查看zookeeper的CONTAINER ID 

sudo docker exec -it CONTAINERID /bin/bash  # 后台进入容器

cd /bin 
./zkCli.sh    #连接本地客户端
```

###### Zookeeper命令

```shell
ls /
ls /services #查看已经注册的服务
```

#### 2.Consul

##### consul安装

```shell
sudo docker pull consul:1.6.1
```

##### consul运行

```shell
sudo docker run -d -p 8500:8500 --restart=always --name=consul consul:1.6.1 agent -server -bootstrap -ui -node=1 -client='0.0.0.0'
```

- agent: 表示启动 Agent 进程。
- server：表示启动 Consul Server 模式
- client：表示启动 Consul Cilent 模式。
- bootstrap：表示这个节点是 Server-Leader ，每个数据中心只能运行一台服务器。技术角度上讲 Leader 是通过 Raft 算法选举的，但是集群第一次启动时需要一个引导 Leader，在引导群集后，建议不要使用此标志。
- ui：表示启动 Web UI 管理器，默认开放端口 8500，所以上面使用 Docker 命令把 8500 端口对外开放。
- node：节点的名称，集群中必须是唯一的，默认是该节点的主机名。
- client：consul服务侦听地址，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1所以不对外提供服务，如果你要对外提供服务改成0.0.0.0
- join：表示加入到某一个集群中去。 如：-json=192.168.0.11。

##### consul使用

前端界面：http://localhost:8500/ui/dc1/services

#### 3.Erlang

```shell
sudo docker pull erlang
```



#### 4.RabbitMQ

#### 5.Zipkin

```shell
docker pull openzipkin/zipkin
docker run -d -p 9411:9411 openzipkin/zipkin
```

```shell
# get the latest source
git clone https://github.com/openzipkin/zipkin
cd zipkin
# Build the server and also make its dependencies
./mvnw -DskipTests --also-make -pl zipkin-server clean install
# Run the server
java -jar ./zipkin-server/target/zipkin-server-*exec.jar
```



windows可以在git bash中执行活动jar包

```shell
curl -sSL https://zipkin.io/quickstart.sh | bash -s io.zipkin:zipkin-server:LATEST:slim zipkin.jar
```



#### **Jmeter**



### Git

下载项目

git clone https://github.com/wy-cug			下载git项目到本地工作目录

git add .         													把工作目录新增加的文件添加到暂存区域，更新文件索引

git commit -m	“init yml”								把暂存区域文件提交到本地仓库

git push orgin master										把本地仓库上传到远程仓库

git remote add origin 仓库地址 				      重新添加远程仓库地址

![image-20211104172216943](C:\Users\wy\Desktop\笔记\images\image-20211104172216943.png)
