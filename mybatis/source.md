- spring+mybatis的结合使用

  - mybatis-spring-boot-autoconfigure（启动扫描注入mybatis相关类）

    ```properties
    自动创建
    1.sqlSessionFactory(sql会话创建工厂)
    2.sqlSessionTemplate(sql会话执行模板)
      - 包含sqlSessionFactory和sqlSessionProxy代理类
    3.AutoConfiguredMapperScannerRegistrar(相关配置和mapper类扫描并注入)
      - org.apache.ibatis.binding.MapperRegistry#addMapper
    
    ```

    

    - mybatis
    - mybatis-spring
      - mybatis和spring的结合
    - spring-boot
      - 引入spring相关的核心组件（spring-cloud等扩展组件单独引入）
    - spring-boot-autoconfigure
      - jdbc包，加载HiKariCP等数据源信息供mybatis使用（默认）
    - spring-boot-start-jdbc
      - HiKariCP
      - spring-jdbc
        - spring-bean
        - spring-core
        - spring-tx



- sqlsessionfactory

  - ```java
      @Override
      public void afterPropertiesSet() throws Exception {
        this.sqlSessionFactory = buildSqlSessionFactory();
      }
    ```

- sqlSession

  - ```properties
    DefaultSqlSession
    SqlSessionManager
    SqlSessionTemplate
    ```

- sqlSessionTemplate

  ```java
    @Override
    public <T> T getMapper(Class<T> type) {
      return getConfiguration().getMapper(type, this);
    }
  ```

  

- 

- 读取配置

```java
  public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
      configurationElement(parser.evalNode("/mapper"));
      configuration.addLoadedResource(resource);
      bindMapperForNamespace();
    }

    parsePendingResultMaps();
    parsePendingChacheRefs();
    parsePendingStatements();
  }
```

