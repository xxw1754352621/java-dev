# GC回收专题

## GC Roots 集合（**引用类型**）-根节点

- 虚拟机栈中的引用对象

- 方法区中的类静态属性引用的对象

- 方法区中的常量引用的对象

- 本地方法栈中JNI（native方法）引用的对象

  ```properties
  Local variables 本地变量
  Static variables 静态变量
  JNI References JNI引用等
  ```

  

## （**多cpu**）[垃圾回收器选择](https://hllvm-group.iteye.com/group/wiki/2865-JVM)

| 参数                                                   | 新生代            | 算法                   | 老年代                           | 算法     |              |
| ------------------------------------------------------ | ----------------- | ---------------------- | -------------------------------- | -------- | ------------ |
| -XX:+UseParallelGC/-XXUserParallelOldGC (**JDK8默认**) | parallel Scavenge | 复制                   | parallel old                     | 标记整理 | 吞吐量优先   |
| -XX:+UseConcMarSweepGc                                 | ParNew            | 复制                   | CMS+SerialOld（SerialOld是后备） | 标-清    | 响应时间优先 |
| -XX:+UseG1GC                                           | G1                | 局部复制，整体标记整理 |                                  |          | 大堆等考虑   |

## JVM参数查看

- java -XX:+PrintFlagsFinal 默认参数

- -XX:+PrintCommandLineFlags  （JVM动态适配或者人）修改的参数

- jps 和jinfo工具的使用
  -XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。
  -XX:MaxGCPauseMillis=100:设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值
  -XX:+UseParallelOldGC：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。
  -XX:ParallelGCThreads=20：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。

- ```properties
  JVM仅指定新生代垃圾收集器的情况下，默认老年代采用Serial Old垃圾收集器（带压缩）：
  -XX:+UseSerialGC
  Serial (DefNew)  + Serial Old（Serial Mark Sweep Compact）
  -XX:+UseParNewGC
  Parallel (ParNew)  +  Serial Old（Serial Mark Sweep Compact）
  
  jdk5出现CMS
  jdk6出现ParallelOldGC

  jdk8默认以下：
  -XX:+UseParallelGC
  Parallel Scavenge (PSYoungGen) + ParallelOldGC 
  
  所以以下配置说明：
  
  -XX:+UseParNewGC -XX:+UseConcMarkSweepGC
  Parallel (ParNew) + CMS（Concurrent Mark Sweep） + Serial Old（Serial Mark Sweep Compact）
  
  作者：二进制之路
  链接：https://juejin.im/post/5c27ab426fb9a049be5d9318
  
  ```

  