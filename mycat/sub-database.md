# mycat的分库分表

## [实现](https://uule.iteye.com/blog/2436039)

```properties
schema.xml

1.数据库配置
<schema name="库名">
  <table name="表名", datanode="节点1,节点2"，rule="分片规则"/>
</schema>
2.分片设置
<dataNode name=“节点1” dataHost="物理数据库配置1" database="库名"/>
<dataNode name=“节点2” dataHost="物理数据库配置2" database="库名"/>
3.物理数据库配置
<dataHost name="物理数据库配置1">
 <writeHost>
 <readHost host="" , url="",userName="",password="">
</dataHost>

```

```properties
rule.xml
<tableRule name="mod-long">
        <rule>
            <columns>分片字段</columns>
            <algorithm>mod-long</algorithm>
        </rule>
</tableRule>
    
<function name="mod-long" class="io.mycat.route.function.PartitionByMod">
        <!-- how many data nodes -->
        <property name="count">2</property>
 </function>
```

## [分库分表的一些原则](https://juejin.im/entry/5a3bca62f265da431c707b9d)

- 非分片字段查询
  - Mycat无法计算路由，便发送到所有节点上执行
- 分页排序
  - limit a 限制数量查询
    - 查询（a）所有库后
      - 无排序，每次返回的结果不一样
      - 有排序下，找最小堆排序
  - limit a,b 分页查询
    - 查询（a+b）所有结果后，再一起做分页
- join查询
  - 在同一库，能查出结果，否则查不出数据（同一次查询）
- 分布式事务（弱一致性事务）
  - 保证prepare阶段数据一致性
    - 提交后，等待下游返回结果前，下游宕机，一直等待，直到超时，但是其他DB成功无法撤销
  - 