# mnesia：分布式数据库模块
 - 伸缩性：根据系统负载，可以在运行中过程中添加或者删除服务节点，改变系统处理规模。 
 - mnesia是一个分布式数据库模块，由多个节点构成的数据库cluster，数据表的位置对应用是透明的。
 - 透过该特性，很容易构建出一个具有高伸缩性的系统。
 - rabbitmq是一个分布式的消息中间件，在mnesia-cluster的机制上可由多个节点共同构建。
