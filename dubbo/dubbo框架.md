# dubbo框架

## 架构图

## SPI扩展点

- 为什么需要
- 怎么实现

## 服务的导入

## 服务的引入

- 同一个JVM的服务
- 直连调用
- 远程调用

- 源码分析

  - ```properties
    proxy0#sayHello(String)
      —> InvokerInvocationHandler#invoke(Object, Method, Object[])
        —> MockClusterInvoker#invoke(Invocation)
          —> AbstractClusterInvoker#invoke(Invocation)
            —> FailoverClusterInvoker#doInvoke(Invocation, List<Invoker<T>>, LoadBalance)
              —> Filter#invoke(Invoker, Invocation)  // 包含多个 Filter 调用
                —> ListenerInvokerWrapper#invoke(Invocation) 
                  —> AbstractInvoker#invoke(Invocation) 
                    —> DubboInvoker#doInvoke(Invocation)
                      —> ReferenceCountExchangeClient#request(Object, int)
                        —> HeaderExchangeClient#request(Object, int)
                          —> HeaderExchangeChannel#request(Object, int)
                            —> AbstractPeer#send(Object)
                              —> AbstractClient#send(Object, boolean)
                                —> NettyChannel#send(Object, boolean)
                                  —> NioClientSocketChannel#write(Object)
    ```

    

  - 启动spring容器，注册bean信息

    AnnotationConfigApplicationContext

  - 创建对象

    - 饿汉式
    - 懒汉式
      - DemoService service = context.getBean("demoServiceComponent", DemoServiceComponent.class);