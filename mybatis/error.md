# 错误集合

1.sha25_passord[认证](https://blog.csdn.net/u013360850/article/details/80373604)

```properties
com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Public Key Retrieval is not allowed
```

# 缓存使用 

- 引入mysql驱动包
- 定位mybatis文件

```properties
mybatis.mapperLocations=classpath:mapper/*.xml
```

- **查询结果对象**实现serializable接口

```java
org.apache.ibatis.cache.CacheException: Error serializing object.  Cause: java.io.NotSerializableException: com.example.rabbitmq.domain.dos.IndexTest

at org.apache.ibatis.cache.decorators.SerializedCache.serialize(SerializedCache.java:97)
```

- 长时间断开连接

```
Caused by: com.mysql.cj.exceptions.CJCommunicationsException: Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.

Caused by: java.net.SocketException: Software caused connection abort: recv failed

```


