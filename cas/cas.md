# sso单点登录的实现方案-cas服务器

- sso面临的问题
  - 同域sso
    - 同一个域名下的资源访问实现单点登录
      - 浏览器的cookie和服务器的session实现
  - 父域sso
    - 同一个顶级域名
      - 比如两个产品的地址分别为 a.dxy.cn 和 b.dxy.cn，那么 cookie 的**域**设置为 dxy.cn 即
  - 跨域sso
    - 不同的域名不一样

- cas实现方案

  - 流程参考：https://juejin.im/post/5a002b536fb9a045132a1727#heading-6

  - 关键组件

    - sso服务器（包含cas服务器 + 统一的登录页面）
  - servic应用服务器
    - 浏览器的

  - 票据

    - TGT：Ticket Grangting Ticket 
  
      TGT 是 CAS 为用户签发的登录票据，拥有了 TGT，用户就可以证明自己在 CAS 成功登录过。TGT 封装了 Cookie 值以及此 Cookie 值对应的用户信息。当 HTTP 请求到来时，CAS 以此 Cookie 值（TGC）为 key 查询缓存中有无 TGT ，如果有的话，则相信用户已登录过。
  
      
  
  - TGC：Ticket Granting Cookie
  
      CAS Server 生成TGT放入自己的 Session 中，而 TGC 就是这个 Session 的唯一标识（SessionId），以 Cookie 形式放到浏览器端，是 CAS Server 用来明确用户身份的凭证。
  
      
  
    - ST：Service Ticket 
  
      ST 是 CAS 为用户签发的访问某一 service 的票据。用户访问 service 时，service 发现用户没有 ST，则要求用户去 CAS 获取 ST。用户向 CAS 发出获取 ST 的请求，CAS 发现用户有 TGT，则签发一个 ST，返回给用户。用户拿着 ST 去访问 service，service 拿 ST 去 CAS 验证，验证通过后，允许用户访问资源。
  
  - 常用配置
  
    - [票据](https://apereo.github.io/cas/5.2.x/installation/Configuring-Ticketing-Components.html)（持久化和过期策略）
  
      - TGT票据设置
  
        - [TicketRegistry](https://apereo.github.io/cas/5.2.x/installation/Configuration-Properties.html#ticket-registry) - Provides for durable ticket storage.
  
        - [ExpirationPolicy](https://apereo.github.io/cas/5.2.x/installation/Configuration-Properties.html#tgt-expiration-policy) - Provides a policy framework for ticket expiration semantics.
  
          - ```properties
            Provides a hard-time out as well as a sliding window.
            # Set to a negative value to never expire tickets
            # cas.ticket.tgt.maxTimeToLiveInSeconds=28800
            # cas.ticket.tgt.timeToKillInSeconds=7200
            ```
  
      - [ST票据设置](https://apereo.github.io/cas/5.2.x/installation/Configuration-Properties.html#service-tickets-behavior)
      
        - ```properties
          # cas.ticket.st.maxLength=20
          # cas.ticket.st.numberOfUses=1
          # cas.ticket.st.timeToKillInSeconds=10
          ```
      
      - [TGC设置](https://apereo.github.io/cas/5.2.x/installation/Configuration-Properties.html#ticket-granting-cookie)
      
        - ```properties
          # cas.tgc.path=
          # cas.tgc.maxAge=-1
          # cas.tgc.domain=
          # cas.tgc.name=TGC
          ```
      
      - 票据之间的关系如图：
      
      - ![9](https://github.com/xxw1754352621/java-dev/img/9.png)