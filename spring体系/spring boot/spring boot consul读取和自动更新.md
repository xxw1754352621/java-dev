# spring boot consul读取和自动更新

- ## 配置初始化

  - 加载consul配置（多个）ConsulPropertySourceLocator#locate

    ```properties
    spring.profiles.active=dev            #The "active profiles" property name.
    spring.profiles.include=test,uat      #The "includes profiles" property name.
    ```

    加载的配置上下文路径

    ![4](https://github.com/xxw1754352621/java-dev/blob/master/img/4.png)

    

  - ![2](https://github.com/xxw1754352621/java-dev/blob/master/img/2.png)

    

  - GET请求获取，host:port/v1/kv/css/应用名

    ![1](https://github.com/xxw1754352621/java-dev/blob/master/img/1.png)

- ## 定时监听变化的配置

  - ```properties
    spring.cloud.consul.config.watch.delay:1000      #固定延时时间
    ```

    ![3](https://github.com/xxw1754352621/java-dev/blob/master/img/3.png)
    

- ## 加载资源

  - 加载多元化配置
  
    ![5](https://github.com/xxw1754352621/java-dev/blob/master/img/5.png)
  
  - 重新记载consul配置的资源
  
    ```java
    org.springframework.cloud.context.refresh.ContextRefresher#refresh
    org.springframework.cloud.bootstrap.config.PropertySourceBootstrapConfiguration#initialize
    重新加载consul配置
    ```
  
    
  
    