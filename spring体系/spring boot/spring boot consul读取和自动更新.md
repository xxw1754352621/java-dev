# spring boot consul读取和自动更新

- ## 配置初始化

  - 加载consul配置（多个）ConsulPropertySourceLocator#locate
  - ![1555999630624](C:\Users\suxw\AppData\Roaming\Typora\typora-user-images\1555999630624.png)
  - ![1556000019945](C:\Users\suxw\AppData\Roaming\Typora\typora-user-images\1556000019945.png)
  - GET请求，host:port/v1/kv/css/应用名

- ## 定时监听变化的配置

  - ```
    配置：spring.cloud.consul.config.watch.delay:1000
    ```

  - ![1556001353456](C:\Users\suxw\AppData\Roaming\Typora\typora-user-images\1556001353456.png)

  - ![](https://github.com/xxw1754352621/java-dev/blob/master/%E7%9F%A5%E8%AF%86%E6%98%9F%E7%90%83.jpg)

- ## 加载资源