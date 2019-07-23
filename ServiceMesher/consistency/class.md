# 一致性种类



## 内部一致性

- ACID（单机事务）
  - 这里的一致性（C），代表数据库状态的一致性，区别于外部一致性

## 外部一致性

描述 **副本问题** 中的一致性的

- 弱一致性
  - 数据更新后，保证【所有】其他副本【最终】读取到最新的数据，但不保证具体的时间
- 最终一致性（推崇）
  - 数据更新后，保证【所有】其他副本【最终】在【某个时间段后】读取到最新的数据
- 强一致性
  - 数据更新后，保证【所有】其他副本，都马上读取到最新的数据



参考：

https://www.jianshu.com/p/2c30d1fe5c4e

https://www.cnblogs.com/xrq730/p/4944768.html

https://cloud.tencent.com/developer/article/1015442

https://www.hollischuang.com/archives/663

[http://blog.kongfy.com/2016/08/%E8%A2%AB%E8%AF%AF%E7%94%A8%E7%9A%84%E4%B8%80%E8%87%B4%E6%80%A7/](http://blog.kongfy.com/2016/08/被误用的一致性/)