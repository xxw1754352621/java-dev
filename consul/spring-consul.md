# spring-consul

## 服务注册与发现

### 服务注册

1.容器启动完毕，事件发布

```java
// Last step: publish corresponding event.
finishRefresh();Lifecycle的

新版本中，容器启动之后SmartApplicationListener ，不在使用DiscoveryLifecycle管理注册
```

2.DiscoveryLifecycle在容器启动后 ，注册到注册中心

应用服务信息：（[Spring容器生命周期回调接口 —> LifeCycle](http://www.ishenping.com/ArtInfo/1618723.html)）

```properties
o.s.c.consul.serviceregistry.ConsulServiceRegistry - Registering service with consul: NewService{id='pds-om-10-10-106-29-9537', name='pds-om', tags=[contextPath=/om], address='10.10.106.29', port=9537, enableTagOverride=null, check=Check{script='null', interval='10s', ttl='null', http='http://10.10.106.29:9537/om/health', tcp='null', timeout='null', deregisterCriticalServiceAfter='1m', tlsSkipVerify=null, status='null'}, checks=null}
```



**以下consul公司提供的API**

注册请求：

```json
{"ID":"pds-om-10-10-106-29-9537","Name":"pds-om","Tags":["contextPath\u003d/om"],"Address":"10.10.106.29","Port":9537,"Check":{"Interval":"10s","HTTP":"http://10.10.106.29:9537/om/health","DeregisterCriticalServiceAfter":"1m"}}

DeregisterCriticalServiceAfter不健康服务1分钟后，去掉注册信息

注册的最后，事件发布完毕，Discovery Client has been initialized
```

PUT请求

```java
RawResponse rawResponse = rawClient.makePutRequest("/v1/agent/service/register", json, tokenParam);
```

注销注册：

```java
RawResponse rawResponse = rawClient.makePutRequest("/v1/agent/service/deregister/" + serviceId, "", tokenParam);
```



### 核心问题

问题1：服务手动删除/宕机状态，如何自动重新注册

```properties
1）重启
2）手动，重新注册
3）宕机状态，定时检查，恢复健康
```

问题2：调用consul注册失败，采用[spring-retry](https://juejin.im/post/5b6ac0a06fb9a04f8a21b192#heading-41)组件

```properties
1）重试介绍：https://blog.csdn.net/u011116672/article/details/77823867
2）配置RetryOperationsInterceptor拦截器，拦截异常，异常才会重试
```

