---
title: SpringCloud微服务架构实战读书笔记①
date: 2019-05-29 15:27:56
tags: JAVA
---

# 微服务架构概述

### 微服务设计原则
- 单一设计原则： 单一职责原则指的是一个单元（类、方法或者是服务）关注的应该只是整个系统中单独且有界限的一部分。
- 服务自治原则： 服务自治指的是每个微服务都应具备独立的业务能力、依赖与运行环境， 服务与服务之间需要高度解耦， 每个服务从开发、测试、构建到部署都应该可以独立的运行， 而不需要依赖其它服务。
- 轻量级通信机制： 微服务之间应该通过轻量级的通信机制进行交互。
- 微服务粒度： 根据合理的粒度划分微服务， 而并非越小越好。

<!--more-->

# Spring Cloud 实战

### 服务提供者与服务消费者

#### 服务提供者
定义： 服务的被调用方， 即为其它服务提供服务的服务


#### 服务消费者
定义： 服务的消费方， 即依赖其它服务的服务


#### Spring Boot Actuator
提供了一些监控的端点

#### 硬编码的问题
硬编码指的是把网络地址嵌套在代码中， 如
```java
@GetMapping("/user/{id}")
```
在不使用注册中心的前提下， 最好将路径提取出来， 放在配置文件中，使用分布式配置中心统一去管理。

### 服务注册与发现
![image.png](https://upload-images.jianshu.io/upload_images/13918038-ea202a8b6cfc0a3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 各个微服务在启动时，将自己的网络地址等信息注册到服务发现组件中，服务发现组件会存储这些信息。
- 服务消费者可以从服务发现组件查询服务提供者的网络地址，并使用该地址调用服务提供者的接口。
- 各个微服务与服务发现组件使用一定机制（如心跳）进行通信。服务发现组件若长时间服务与某服务实例通信，就会注销该实例。
- 微服务网络地址发生变更， 会重新注册到服务发现组件。

综上所述， 服务发现组件应该具备如下功能：
- 服务注册表： 服务发现组件的核心，它用来记录各个微服务的信息，例如微服务的名称、IP、端口等等，该表提供查询API和管理API， 查询API用于查询可用的微服务实例， 管理API用于管理服务的注册与注销。
- 服务注册与服务发现： 服务注册是指在微服务启动时，将自己的信息注册到服务发现组件上的过程。服务发现是指查询可用微服务列表及网络地址的机制。
- 服务检查： 服务发现组件使用一定机制定时检测已注册的服务。

### Eureka

#### Region and Avaliability Zone
![image.png](https://upload-images.jianshu.io/upload_images/13918038-23d39681d04f8e38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Eureka 工作原理
![image.png](https://upload-images.jianshu.io/upload_images/13918038-a502e8b0978441bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- MakeRemoteCall 是RESTFul API的调用行为
- us-east-1c、us-east-1d等都是Avaliability Zone, 他们都属于us-east-1这个region
- Eureka 包含两个组件：一个是Eureka server， 另一个是Eureka client， 在Eureka集群的情况下， Eureka Server也是Eureka Client， 多个Eureka Server之间通过相互复制的方式来实现服务注册表中的数据同步。

几个比较重要的配置：
- eureka.client.registerWithEureka：表示是否将自己注册到EurekaServer
- eureka.client.fetchRegistry：表示是否从Eureka Server获取注册信息，默认为true。
- eureka.client.serviceUrl.defaultZone：设置与EurekaServer交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka；多个地址间可使用,分隔。

#### 将微服务注册到Eureka Server中
几个比较重要的配置
- Spring.application.name: 用于注册到Eureka Server上的应用名称
- eureka.instance.prefer-ip-address=true 表示将自己的IP注册到Eureka Server中， 若为false， 一般会将操作系统的hostname注册到Eureka Server中。

#### 为Eureka Server添加安全认证的支持， 可以使用Spring Security
![image.png](https://upload-images.jianshu.io/upload_images/13918038-26f6b2a271ef2fc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

   将eureka.client.serviceUrl.defaultZone配置为http://user:password@EUREKA_HOST:EUREKA_PORT/eureka/的形式, 即可成功注册。
![image.png](https://upload-images.jianshu.io/upload_images/13918038-3ee039946a47b453.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Eureka 元数据
Eureka 的元数据包括两种，分别是标准元数据和自定义元数据。

- 标准元数据指的是主机名、IP地址、端口号、状态页和健康检查等信息，这些信息都会被发布在服务注册表中，用于服务之间的调用。
- 自定义元数据， 可以使用eureka.instance.metadata-map配置，这些元数据可以被远程客户端访问， 通过提前的约定使得远程客户端知道该如何使用

#### Eureka Server的REST端点
![image.png](https://upload-images.jianshu.io/upload_images/13918038-0d8849fe445595af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/13918038-1420a77c674b973d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Eureka 的自我保护模式
![image.png](https://upload-images.jianshu.io/upload_images/13918038-65a9f43bcd1a020a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 默认情况下， Eureka Server心跳检测的默认限度是90s, 超过之后没有收到回复将会注销该服务
- 但在极端情况下， 如网络故障， 则会出现短时间内大规模注销的情况， 这时Eureka server将会启动自我保护模式， 在该模式下， 不会注销任何服务， 当网络恢复时， Eureka server会自动退出自我保护模式。
- eureka.server.enable-self-preservation=false禁用自我保护模式

#### 多网卡环境下的IP选择
建议直接使用eureka.instance.ipAddress手动指定IP地址， 特别是在Docker容器中

#### Eureka 健康检查
我们上面所说的心跳之类的，都只是Eureka与服务之间的通信是否正常，而通信正常不代表服务正常， 如服务不能与数据库进行连接， 此时便需要更加细化的检查了。


### 使用Ribbon实现客户端负载均衡
在Spring Cloud中，当Ribbon与Eureka配合使用时，Ribbon可自动从Eureka Server获取服务提供者的地址列表，并基于负载均衡算法，请求其中一个服务提供者的实例。
![image.png](https://upload-images.jianshu.io/upload_images/13918038-6285ccb925df4da5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
为RestTemplate添加@LoadBalanced注解

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
  return new RestTemplate();
}
```

相关注意事项：
- 不能将restTemplate.getForObject与loadBalancedClient.choose写在同一个方法中，因为前者已经有选择的意思，前者包括后者，restTemplate实际上已经是Ribbon客户端本身，已经包含Choose行为
- 虚拟主机名不能包括 _ 字符  

#### Ribbon配置自定义(翻下官方文档自己配)


#### 脱离Eureka使用Ribbon
有一些遗留的微服务，它们没有注册到Eureka Server上， 这个时候， 仍然可以使用Ribbon实现负载均衡
![image.png](https://upload-images.jianshu.io/upload_images/13918038-ff1babc38ad51a81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置：microservice-provider-user.ribbon.listOfServers
![image.png](https://upload-images.jianshu.io/upload_images/13918038-9ecbeafbb4bb8f63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 饥饿加载
Spring cloud 为每个名称的Ribbon Client维护一个子应用上下文，这个上下文默认的懒加载的，指定名称的Ribbon Client第一次请求的时候，对应的上下文才会被加载，因此第一次加载时，会比较慢，此时，我们可以配置懒加载
![image.png](https://upload-images.jianshu.io/upload_images/13918038-749611dfbfde1bea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样，对于名为client1、client2的Ribbon Client，将在启动时加载对应的子应用程序上下文。
