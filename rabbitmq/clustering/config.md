# [single machine](https://www.rabbitmq.com/clustering.html)
```java
In order to run multiple RabbitMQ nodes on a single machine, 
it is necessary to make sure the nodes have distinct node names, 
data store locations, log file locations, 
and bind to different ports, including those used by plugins. 
See RABBITMQ_NODENAME, RABBITMQ_NODE_PORT, and RABBITMQ_DIST_PORT 
in the Configuration guide, as well as RABBITMQ_MNESIA_DIR, RABBITMQ_CONFIG_FILE, and RABBITMQ_LOG_BASE
in the File and Directory Locations guide.
```



# 设置hosts
```properties
127.0.0.1 kaka-01
127.0.0.1 kaka-02
```

# 复制三份安装目录

# 修改配置文件:home/etc/rabbitmq.config

修改rabbitmq_management服务的端口

```java
  {listener,[{port,15672},
              {ip,"127.0.0.1"},
              {ssl,false}]
   }
```

# 修改环境变量执行文件：home/sbin/rabbitmq-env.bat

修改rabbit节点端口，节点名称（rabbit服务名称@hostname），配置文件，基础用户目录

```properties
set RABBITMQ_CONFIG_FILE=!RABBITMQ_HOME!\etc\rabbitmq
set RABBITMQ_BASE=!RABBITMQ_BASE!\rabbitmq-cluster
set RABBITMQ_NODENAME=rabbit2@kaka-02
set RABBITMQ_NODE_PORT=5673
```

依次修改其他节点

错误提示：

```java
ERROR: node with name "rabbit" already running on "kaka-02"
```

协议如下：

| Protocol   | Bound to | Port  |
| ---------- | -------- | ----- |
| amqp       | 0.0.0.0  | 5672  |
| amqp       | 0.0.0.0  | 5672  |
| amqp       | ::       | 5672  |
| clustering | ::       | 25672 |
| http       | 0.0.0.0  | 15672 |
| http       | ::       | 15672 |



# 常用命令（省略 -n 节点操作）

Usage:
rabbitmqctl [-n <node>] [-l] [-q] <command> [<command options>]

General options

| short            | long          | description                    |
| -----------------|---------------|------------                    |
| -n <node>        | --node <node> | connect to node <node>         |
| -l               | --longnames   | use long host names            |
| -q               | --quiet       | suppress informational messages|
| -s               | --silent      | suppress informational messages|
                                   

- rabbitmq-server  -n rabbit1@kaka-01 -**detached** 
  - 后端运行，依次运行其他三个节点
- rabbitmqctl  -n rabbit2@kaka-02 **stop_app**
  - 应用停止运行，提示Stopping rabbit application on node rabbit2@kaka-02 ...
- rabbitmqctl **start_app**
  - 应用运行
- rabbitmqctl -n rabbit1@kaka-01 **stop**
  - 停止运行节点，提示：Stopping and halting node rabbit@kaka-01 ...
- rabbitmqctl **join_cluster** **--ram** rabbit1@kaka-01
  - 加入节点1
  -  --ram 内存节点，默认--disc
- rabbitmqctl -n rabbit1@kaka-01 **reset**
  - 清除节点1，初始化
- rabbitmqctl  -n rabbit1@kaka-01 **cluster_status**
  - 当前节点集群状态
- rabbitmqctl  **forget_cluster_node** rabbit1@kaka-01
  - 从节点1的集群中退出
- rabbitmqctl **change_cluster_node_type** disc
  - 改变当前节点为磁盘节点
  - 注意：先停止应用（stop_app），再进行操作
- rabbitmq-plugins enable **rabbitmq_management**
  - 启动rabbitmq管理应用（15276）
# 启动要求
  - 启动：磁盘节点 到 内存节点
  - 关闭：内存节点 再到 磁盘节点
