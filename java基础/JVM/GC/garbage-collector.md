前提，三个概念

1.并行（**Parallel** ）与并发

- 是否同一时间运行（单核与多核CPU）

2.吞吐量

- 代码运行时间 /（代码运行时间+GC时间）

3.**Minor** GC和**Full** GC（Major GC）

# 新生代收集器

serial收集器（单线程）

- （单核CPU）单线程
- 复制算法
- 垃圾收集时，STW（stop the word）
- 老年代收集：配合serial old 和CMS 收集器

ParNew收集器（多线程）

- Par 是 Parallel （并行）

- 同上，只是多线程，充分利用多核CPU，从而并行

Parallel Scavenge收集器（JDK1.7）

- 同ParNew
- 老年代收集：配合serial old 和Parallel old 收集器
- 可控制吞吐量
  - **GC自适应的调节策略（GC Ergonomics）**
    - **-XX:+UseAdaptiveSizePolicy**，这是一个开关参数，打开参数后，就不需要手工指定新生代的大小（-Xmn）、Eden和Survivor区的比例（-XX:SurvivorRatio）、晋升老年代对象年龄（-XX:PretenureSizeThreshold）等细节参数了。

# 老年代收集器

Serial old 收集器

Parallel old 收集器

CMS收集器（JDK1.6）

# 跨代收集器

G1（JDK1.8）

ZGC（JDK11）



