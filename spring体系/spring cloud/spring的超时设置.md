**Ribbon、Feign、Hystrix组件的超时设置**

# feign动态代理

创建连接，构建请求，发起请求，获取响应，解析响应

- 配置：

  ```yaml
  feign:
    client:
      config:
        feignName:
          connectTimeout: 5000
          readTimeout: 5000
  ```

- 源码：
- <https://mp.weixin.qq.com/s?__biz=MzU0OTk3ODQ3Ng==&mid=2247483712&idx=1&sn=4cd88761830428a2e485ac4c2cf120f9&chksm=fba6e943ccd16055344222ce9c794358e1a4a84fdf4263eaa7c91e9756597bd06e49f9b390cb&scene=21#wechat_redirect>



# ribbon负载均衡

获取consul注册的服务，负载选中服务进行调用

- 配置：

  ```yaml
  ribbon:
    ReadTimeout: 60000
    ConnectTimeout: 60000
  ```



# hystrix的线程池

隔离，熔断以及降级

- 配置：

  ```yaml
  hystrix:
    command:
      default:
        execution:
          timeout:
            enabled: true
          isolation:
            thread:
              timeoutInMilliseconds: 1000
  ```

  

- Hystrix的超时 > 其他组件的超时

参考：

<http://www.itmuch.com/spring-cloud-sum/spring-cloud-timeout/>



