TCP会话

协议TCP/IP4 层，OSI 7层模型

应用层协议

- HTTP 超文本转移协议（超文本传输协议）
  - 以太网头部帧（数据链路层+物理层）
    - IP头部（以太网数据包，网络层），目的IP地址的在路由器间拆装包
      - TCP/UDP头部（IP数据包，传输层）
        - 应用数据包（TCP/UDP数据包，应用层+表示层+会话层）
  - 以太网尾部帧（拆包发送，原因IP数据包过大，对TCP数据包拆分）

- 其他协议
  - FTP协议（文本转移协议）
  - SSH 为 [Secure Shell](http://baike.baidu.com/view/2118359.htm) 的缩写，由 IETF 的网络小组（Network Working Group）所制定；SSH 为建立在应用层基础上的安全协议。SSH 是目前较可靠，专为[远程登录](http://baike.baidu.com/view/59099.htm)会话和其他网络服务提供安全性的协议