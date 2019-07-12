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



- MybatisAutoConfiguration

  - 创建AutoConfiguredMapperScannerRegistrar

    - ClassPathMapperScanner
      - Searching for mappers annotated with @Mapper
      - Creating MapperFactoryBean with name 'indexTestMapper' and 'com.example.rabbitmq.mybatis.IndexTestMapper' mapperInterface
      - Enabling autowire by type for MapperFactoryBean with name 'indexTestMapper'

  - 创建SqlSessionFactory

    - ###### 创建SqlSessionFactoryBean来创建SqlSessionFactory

    - ```java
      @Override
      public void afterPropertiesSet() throws Exception {
        this.sqlSessionFactory = buildSqlSessionFactory();
      }
      ```

      - 调用org.mybatis.spring.SqlSessionFactoryBean#afterPropertiesSet方法创建 SqlSessionFactory
        - 说明其他属性设置完后，设置属性sqlSessionFactory

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



- MapperFactoryBean（创建的代理，控制开放和关闭 session）

  - SqlSessionDaoSupport

  - ```java
      @Override
      public T getObject() throws Exception {
        return getSqlSession().getMapper(this.mapperInterface);
      }
    ```

    - DefaultSqlSession

      - MapperRegistry

        - org.apache.ibatis.binding.MapperRegistry#getMapper

      - MapperProxyFactory

        - ```java
          public T newInstance(SqlSession sqlSession) {  
              final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);  
              return newInstance(mapperProxy);
          }
          此方法实现代理sqlsession执行真正的sql语句
          ```

- SqlSessionUtils

  - Creating a new SqlSession
  - SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7babed9e] was not registered for synchronization because synchronization is not active
  - Closing non transactional SqlSession

- LoggingCache
  - Cache Hit Ratio [com.example.rabbitmq.mybatis.IndexTestMapper]: 0.0
- HikariDataSource
  - HikariPool-1 - Starting...
  - HikariPool-1 - Start completed.
- SpringManagedTransaction
  - JDBC Connection [HikariProxyConnection@326802793 wrapping com.mysql.cj.jdbc.ConnectionImpl@12faeada] will not be managed by Spring
- SqlSessionUtils
  - Closing non transactional SqlSession

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

