Spring Cloud Netflix 集成众多Netflix的开源软件：Eureka, Hystrix, Zuul, Archaius，组成了微服务的最重要的核心组件。

# **Zuul**

- 一方面，Zuul是接入网关，起到反向代理的作用，是外部消费者请求内部服务的唯一入口。

- 另一方面，Zuul也具备过滤功能，通过在运行时注入过滤规则可实现用户鉴权、动态路由、灰度发布、A/B测试、负载限流等功能。

Zuul的大部分功能都是通过过滤功能来完成的，Zuul可以提供四种标准类型的过滤

zuul介绍：<https://www.oudahe.com/p/44585/>

zuul高可用：<https://blog.csdn.net/gaowenhui2008/article/details/77946600>

