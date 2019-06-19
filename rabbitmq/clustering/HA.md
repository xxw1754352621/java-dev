# 高可用队列（[HA](https://www.rabbitmq.com/ha.html)）

  - rabbitmqctl set_policy [-p Vhost] Name Pattern Definition [Priority]

```properties
-p Vhost： 可选参数，针对指定vhost下的queue进行设置
Name: policy的名称
Pattern: queue的匹配模式(正则表达式)
Definition：镜像定义，包括三个部分ha-mode, ha-params, ha-sync-mode
	ha-mode:指明镜像队列的模式，有效值为 all/exactly/nodes
		all：表示在集群中所有的节点上进行镜像
		exactly：表示在指定个数的节点上进行镜像，节点的个数由ha-params指定
		nodes：表示在指定的节点上进行镜像，节点名称通过ha-params指定
	ha-params：ha-mode模式需要用到的参数
	ha-sync-mode：进行队列中消息的同步方式，有效值为automatic和manual
priority：可选参数，policy的优先级
```

# 镜像交换机的意义