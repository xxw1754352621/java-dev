- # spring+mybatis的结合使用

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



- # MybatisAutoConfiguration

  - 创建AutoConfiguredMapperScannerRegistrar

    - 代理带有@Mapper注解的类
    - ClassPathMapperScanner
      - Searching for mappers annotated with @Mapper
      - **Creating MapperFactoryBean** with name 'indexTestMapper' and 'com.example.rabbitmq.mybatis.**IndexTestMapper**' mapperInterface
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

- # sqlSession

  - ```properties
    DefaultSqlSession
    SqlSessionManager
    SqlSessionTemplate
    ```

- # sqlSessionTemplate

  ```java
    @Override
    public <T> T getMapper(Class<T> type) {
      return getConfiguration().getMapper(type, this);
    }
  
    //
    private class SqlSessionInterceptor implements InvocationHandler {
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
        //调用SqlSessionUtils创建sqlSession
        //Gets an SqlSession from Spring Transaction Manager 
        //or creates a new one if needed
        SqlSession sqlSession = getSqlSession(
            SqlSessionTemplate.this.sqlSessionFactory,
            SqlSessionTemplate.this.executorType,
            SqlSessionTemplate.this.exceptionTranslator);
   
      }
  ```


- # MapperFactoryBean

  创建的代理，控制开放和关闭 session

  - SqlSessionDaoSupport
  - MapperProxyFactory

    ```java
    public T newInstance(SqlSession sqlSession) {  
          final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);  
          return newInstance(mapperProxy);
      }
      此方法实现代理sqlsession执行真正的sql语句
    ```
    - DefaultSqlSession

      - MapperRegistry

        - org.apache.ibatis.binding.MapperRegistry#getMapper
          - 注册类信息

- # MapperProxy

  - 每个@Mapper类生成一个代理类

  - 由此代理类执行方法

  - ```java
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (Object.class.equals(method.getDeclaringClass())) {
          try {
            return method.invoke(this, args);
          } catch (Throwable t) {
            throw ExceptionUtil.unwrapThrowable(t);
          }
        }
        final MapperMethod mapperMethod = cachedMapperMethod(method);
        return mapperMethod.execute(sqlSession, args);
      }
    ```

    - MapperMethod
      - **通过sqlSessionTemplate执行真正sql方法**

- # MappedStatement

- # CachingExecutor

  Executor的子类

  - 执行查询方法

  ```java
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
        //创建真正的sql
      BoundSql boundSql = ms.getBoundSql(parameterObject);
        //创建缓存key
      CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
        //查询
      return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }
  ```

  - FlushCachePolicy

    - 二级缓存刷新策略

      - ```java
         /** <code>false</code> for select statement; <code>true</code> for insert/update/delete statement. */
            DEFAULT
        ```

  - TransactionalCache

    - 二级缓存的事务支持

  - BaseExecutor

    - 已经开启二级缓存（@CacheNamespace 类注解）
    - 如果二级缓存没有结果，执行BaseExecutor.query()
      - 如果一级缓存没有结果（localCache本地缓存），直接查询数据库

- # BoundSql


  - 创建真正的sql

- # CacheKey


  - 创建缓存key

- # SqlSessionUtils

  - Creating a new SqlSession
  - **[SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7babed9e] was not registered for synchronization because synchronization is not active](https://blog.csdn.net/haoyifen/article/details/51172647)**
    - 没有启用事务注解
  
- # LoggingCache
  
  - Cache Hit Ratio [com.example.rabbitmq.mybatis.IndexTestMapper]: 0.0
  
- # HikariDataSource
  
  - HikariPool-1 - Starting...
  - HikariPool-1 - Start completed.
  
- # SpringManagedTransaction
  
  - JDBC Connection [HikariProxyConnection@326802793 wrapping com.mysql.cj.jdbc.ConnectionImpl@12faeada] will not be managed by Spring
  
- # SqlSessionUtils
  
  
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

