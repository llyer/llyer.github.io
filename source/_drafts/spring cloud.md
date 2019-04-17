---
title: 五层协议
tags:
---

spring cloud 是一套标准，他是有不少实现的

比如说  Netflix 生态 

- ribbon 客户端负载均衡

- Feign 客户端服务消费

- eureka 服务发现

- hystrix 服务熔断，并发高峰时，熔断一部分服务。比如双十一期间，不能退货等等，把核心资源都放到销售处理上


国内的由 alibaba 生态  

- dubbo 


5. 谈谈你对 Dubbo 和 Spring Cloud 的认识(两者关系)
具体可以看公众号-阿里巴巴中间件的这篇文章:独家解读：Dubbo Ecosystem - 从微服务框架到微服务生态

Dubbo 与 Spring Cloud 并不是竞争关系，Dubbo 作为成熟的 RPC 框架，其易用性、扩展性和健壮性已得到业界的认可。未来 Dubbo 将会作为 Spring Cloud Alibaba 的 RPC 组件，并与 Spring Cloud 原生的 Feign 以及 RestTemplate 进行无缝整合，实现“零”成本迁移。

在阿里巴巴的微服务解决方案中，Dubbo、Nacos 和 Sentinel，以及后续将开源的微服务组件，都是 Dubbo EcoSystem 的一部分。我们后续也会将 Dubbo EcoSystem 集成到 Spring Cloud 的生态中。


rpc 和 restful 的相同点是服务间调用 


rpc 的优势是服务端内部调用


restful 的有点是外部服务调用


小公司根本就不在乎这点性能差异，全是 restful


rpc 和 restful 的区别，rpc 的实现就是 dubbo


spring cloud 一般用 restful


