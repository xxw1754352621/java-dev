# AMQP帧
  - 帧类型
    -  协议头帧用于连接到rabbitmq，进使用一次。
    -  方法帧携带发送给rabbitmq或者从rabbitmq接收到的rpc请求或者响应
    -  内容帧包含一条消息的大小和属性。
    -  消息体帧包含消息的内容
    -  心跳帧在客户端与rabbitmq直接进行传递，作为一种校验机制确保连接的两端都可用并且正常工作。
    - 一条消息包含：方法 + 内容 + 消息体
  - 信道编号
  - 以字节为单位的帧大小
  - 帧有效载荷payload
  - 结束字节标志（ASCII值206）


