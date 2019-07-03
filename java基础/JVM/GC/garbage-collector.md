# 前提

三个概念

1.并行（**Parallel** ）与并发

- 是否同一时间运行（单核与多核CPU）

2.吞吐量

- 代码运行时间 /（代码运行时间+GC时间）

3.**Minor** GC和**Full** GC（Major GC）

- Full GC
  - 指发生在老年代的GC，出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。Major GC的速度一般会比Minor GC慢10倍以上。

# 新生代收集器

serial收集器（单线程）

- （单核CPU）单线程
- 复制算法
- 垃圾收集时，STW（stop the word）
- 老年代收集：配合serial old 和CMS 收集器

ParNew收集器（多线程）

- Par 是 Parallel （并行）

- 同上，只是多线程，充分利用多核CPU，从而并行
- **-XX:ParallerGCThreads**
  - 它默认开启的收集线程数与CPU的数量相同，在CPU非常多的情况下可使用参数设置。

Parallel Scavenge收集器

- 同ParNew
- 老年代收集：配合serial old 和Parallel old 收集器
- 可控制吞吐量（**吞吐量优先**收集器）
  - **GC自适应的调节策略（GC Ergonomics）**
    - **-XX:+UseAdaptiveSizePolicy**，这是一个开关参数，打开参数后，就不需要手工指定新生代的大小（-Xmn）、Eden和Survivor区的比例（-XX:SurvivorRatio）、晋升老年代对象年龄（-XX:PretenureSizeThreshold）等细节参数了。

# 老年代收集器

[Serial old 收集器](https://crowhawk.github.io/2017/08/15/jvm_3/)

- 标记整理算法（Mark-Compact）
- 单线程
- 此收集器的主要意义也是在于给Client模式下的虚拟机使用。如果在Server模式下，它还有两大用途：
  - 在JDK1.5 以及之前版本（Parallel Old诞生以前）中与Parallel Scavenge收集器搭配使用。
  - 作为CMS收集器的后备预案，在并发收集发生**Concurrent Mode Failure**时使用

Parallel old 收集器（JDK1.6）

- 标记整理算法（Mark-Compact）
- 多线程
- 配合Parallel Scavenge
  - 在**注重吞吐量**以及**CPU资源敏感**的场合，都可以优先考虑Parallel Scavenge加Parallel Old收集器。

CMS（**Concurrent Mark Sweep**）收集器（JDK1.5）

- 标记清理算法（Mark-Sweep）
- 获取最短回收停顿时间
  - **并发收集**、**低停顿**
- 步骤
  - **初始标记（CMS initial mark）**：（**单线程**）仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，需要“Stop The World”。
  - **并发标记（CMS concurrent mark）**：进行**GC Roots Tracing**的过程，在整个过程中耗时最长。
  - **重新标记（CMS remark）**：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。此阶段也需要“Stop The World”。
  - **并发清除（CMS concurrent sweep）**

# 跨代收集器

G1（JDK1.8）

- 标记-整理+复制算法

- **面向服务端应用**
- 特点
  - **并行与并发** G1 能充分利用多CPU、多核环境下的硬件优势，使用多个CPU来缩短“Stop The World”停顿时间，部分其他收集器原本需要停顿Java线程执行的GC动作，G1收集器仍然可以通过并发的方式让Java程序继续执行。
  - **分代收集** 与其他收集器一样，分代概念在G1中依然得以保留。虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但它能够采用不同方式去处理新创建的对象和已存活一段时间、熬过多次GC的旧对象来获取更好的收集效果。
  - **空间整合** G1从整体来看是基于**“标记-整理”**算法实现的收集器，从局部（两个Region之间）上来看是基于**“复制”**算法实现的。这意味着G1运行期间不会产生内存空间碎片，收集后能提供规整的可用内存。此特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次GC。
  - **可预测的停顿** 这是G1相对CMS的一大优势，降低停顿时间是G1和CMS共同的关注点，但G1除了降低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在GC上的时间不得超过N毫秒，这几乎已经是实时Java（RTSJ）的垃圾收集器的特征了。

[ZGC（JDK11）](https://juejin.im/post/5bade237e51d450ea401fd71)

- **着色指针Colored Pointer**
  - 引用做手脚
  - 内存在Compact被移动时，引用颜色就会发生变化
- **读屏障Load Barrier** 
  - 基于着色指针
  - 屏障就会先把指针更新为有效地址再返回



