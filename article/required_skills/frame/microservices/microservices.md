# 认知微服务

## 为什么微服务

基于熟悉的springboot项目来看，随着时间的积累，后台的接口必定是暴增的，当用户达到一定的体量，后台服务器的压力就会非常大，响应会趋于缓慢，而且一旦某个接口扛不住，或者其他一丁点问题的冒头，都会导致整个服务的崩溃。另外，即使自信不会出现上述问题，那么在产品迭代更新的时候也绝不会是好维护的，各种服务模块耦合在一起，十分的麻烦。自此一种化整为零的思想就冒出来了，将整个后台拆分为多个服务，一来可以横向扩展保证性能，二来也有利于产品的维护，三来整个服务也更加高可用了。

## 微服务的难题

如上所述，化整为零即可实现微服务，那么将服务拆分之后又会有哪些问题呢。

+ 如何调用接口，各服务之间如何通信。
+ 如何保证服务健康或者可用。
+ 服务异常该怎么办。
+ 如何在不中断整体服务的基础上，更新服务配置。

以上应该就是最基本的灵魂拷问了，当然也绝不会仅仅是这些。可见微服务虽然很诱人，但实现起来也不是那么轻松的。



# 如何实现微服务

首先，即使是做了拆分实现了微服务，最后的单一的服务也一定会是一个springboot的项目，那么对于这样一个大量复杂的springboot项目群，自然是越少干预越好维护，自然会想到让第三方的服务来为它们保驾护航，于是乎微服务组件就成了微服务架构不可或缺的一部分，他们是为微服务群提供支持的 "守护服务"。那么实现微服务又需要哪些守护服务呢。

1. **注册发现中心。**类比spring的IOC容器，为了集中管理所有的服务，我们开启一类服务专门用于注册微服务，并采取心跳机制让这类服务监控各微服务的健康状态，以供基础支持和。
2. **配置中心。** 用于动态拉取微服务的配置信息，实现服务配置的热更新。
3. **服务网关。** 用于拦截分发用户的请求。
4. **服务通信组件。** 为各微服务之间的调用提供支持。
5. **容灾组件。** 用于系统保护，处理异常服务，紧急避险。
6. **日志组件。** 用于日志记录，便于系统维护。

基本的微服务组件，至少也要包含1，4，5来提供对外服务。

![image-20220729065747993](https://lizhuo-file.oss-cn-hangzhou.aliyuncs.com/img/image-20220729065747993.png)

# 注册发现

## Eureka

## Zookeeper

## Consul

## Nacos

```bash
# db mysql
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```



# 通信组件

## Ribbon

## LoadBalancer

## OpenFeign

​		是一种声明式服务调用组件，在 RestTemplate 的基础上做了进一步的封装。使用时只需要声明一个接口并通过注解进行简单的配置，即可实现对 HTTP 接口的绑定。做到无感知地远程调用。

核心功能包括：远程调用、负载均衡

**极简化使用：**

```
1.引依赖 仅消费方引入即可
<dependency>
 	   <groupId>org.springframework.cloud</groupId>
		 <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

2.启动类添加注解 @EnableFeignClients

3.创建服务的对接接口
@FeignClient(value = "服务名称") // 标注该类是一个feign接口
public interface MenClient {
	@GetMapping("men/{id}")
	public Men getOne(@PathVariable("id") Integer id);
}

4.controller引入服务接口
	@Autowired
	private MenClient menClient;

5.开搞
```



## Dubbo



# 容灾组件（服务降级）

## Hystrix

## Sentienl

# 服务网关

## Zuul

## Gateway

# 配置中心

## Config

## Nacos

# 服务总线

## Bus

## Nacos