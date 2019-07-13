问题提出：

- 命名空间（表级）的更新，对于写频繁的表不够友善
- 

MybatisAutoConfiguration 

- :arrow_right:SqlSessionFactory

  - InitializingBean#afterPropertiesSet

    ```jaba
      public SqlSessionFactory build(Configuration config) {
        return new DefaultSqlSessionFactory(config);
      }
    ```

  - 使用HikariDataSource数据源

  - 通过SqlSessionFactoryBean创建

  - MapperProxy

    - ```
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

      

-  :arrow_right:SqlSessionTemplate

- :arrow_right:AutoConfiguredMapperScannerRegistrar  
  - Searching for mappers annotated with @Mapper