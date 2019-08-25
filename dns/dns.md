# DNS[解析](https://juejin.im/post/59c6201df265da064428b835#heading-1)

- DNS域名解析服务器
  - 跟linux文件结构类似，树结构
  - 分布式存储
  - 种类
    - 递归域名解析器（DNS本地服务器）
      - 缓存
        - 不是每次都递归，采用谷歌等大厂的服务器，8.8.8.8等
      - 递归解析 - 获取权威域名解析服务器的IP
        - 智能解析
          - 智能解析依赖EDNS协议，这是google 起草的DNS扩展协议， 修改比较简单，就是在DNS包里面添加origin client IP, 这样nameserver 就能根据client IP 返回距离client 比较近的server IP 
    - 权威域名服务器
      - 根域名解析服务器
        - 存储13个顶级顶级域名解析器的IP
      - 其他域名解析服务器
        - **经过上一级授权**

图片说明如下:

![7](https://github.com/xxw1754352621/java-dev/img/7.png)



# DNS权威域名服务器搭建

- ## bind配置文件

- ## zone配置文件

  

## DNS负载均衡 vs LVS专业负载均衡

和 LVS 这种专业负载均衡工具相比，在DNS层做负载均衡有以下特点:

1. 实现非常简单
2. 默认只能通过RR方式调度
3. DNS 对后端服务不具备健康检查
4. DNS 故障恢复时间比较长（DNS服务之间有缓存）
5. 可负载的rs数量有限（受DNS response包大小限制)



# [顶级域名|二级域名|子域名|父域名](https://zhuanlan.zhihu.com/p/27290218)

如图：

![8](https://github.com/xxw1754352621/java-dev/img/8.png)