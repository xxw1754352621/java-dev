# Clustering
### master位置分布策略
  - queue-master-locator 配置文件
    - min-masters:选择master数量最少的节点
    - client-local:选择声明队列的客户端连接的节点
    - random:随机节点
  - x-queue-master-locator 队列声明参数