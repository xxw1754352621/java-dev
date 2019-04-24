# spring boot consul读取和自动更新

- ## 配置初始化

  - 加载consul配置（多个）ConsulPropertySourceLocator#locate
  - ![1](https://github.com/xxw1754352621/java-dev/blob/master/img/1.png)
  - ![2](https://github.com/xxw1754352621/java-dev/blob/master/img/2.png)
  - GET请求，host:port/v1/kv/css/应用名

- ## 定时监听变化的配置

  - ```
    配置：spring.cloud.consul.config.watch.delay:1000
    ```

  - ![3](https://github.com/xxw1754352621/java-dev/blob/master/img/3.png)

- ## 加载资源

  - 