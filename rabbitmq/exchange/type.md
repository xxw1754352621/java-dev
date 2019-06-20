# 消息路由模式
  - binding key（队列绑定的键）
  - routing key（生产者发送的路由键）
# fanout
  - 所有绑定exchange的路由器
# direct
  - 绑定指定路由器和指定路由键的队列
# topic
  - 绑定指定路由器和【特殊路由键】的队列
    - 路由键可以有*和#,例如A.*.C和A.#
      - *表示两个.之间的符号模糊匹配
      - \#表示后面的任意符号模糊匹配
# header
  - 不依赖于routing key与binding key的匹配规则来路由消息
  - 在绑定Queue与Exchange时指定一组键值对以及x-match参数，x-match参数是字符串类型，
    可以设置为any或者all。如果设置为any，意思就是只要匹配到了headers表中的任何一对键值即可，all则代表需要全部匹配。

------

```properties
    （1）fanout：会把发送到交换机A的消息发送个全部与A绑定的队列；
    （2）direct：根据路由键和绑定键去匹配交换机的指定队列；
    （3）topic：是direct的拓展，利用路由键和绑键模糊匹配队列；
    （4）header：根据发生消息的headers属性中的键值对进行匹配
```

