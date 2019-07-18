# JDK1.8的扩容方法:arrow_right:resize()

## 1.前提：介绍在put()方法下，发生**扩容**条件如下

1.1 当第一次调用put方法时，开始新建数组，**分配内存** ，  **初始化扩容**

1.2 当数组（table参数，也就是集合数组结构的容量）**小于 64**

​      1.2.1 当链表超过8个，先判断是否有必要树形化（**MIN_TREEIFY_CAPACITY**）

​      1.2.2 treeifyBin()方法

​      1.2.3 区别于1.3的判断

1.3 当值存储到集合后，判断当前 **++存储数量（个数） > 阈值**

### 两种阈值
 初始化阈值计算，和，**第一次put之后的阈值计算** =（初始化阈值*加载因子）的n次幂，不断翻倍增长

#### **初始化阈值**计算如下，计算方法tableSizeFor()：

  - 代码如下：

    - ```java
      静态工具方法：
          /* ---------------- Static utilities -------------- */
          /**
           * Returns a power of two size for the given target capacity.
           */
          static final int tableSizeFor(int cap) {
              int n = cap - 1;
              n |= n >>> 1;
              n |= n >>> 2;
              n |= n >>> 4;
              n |= n >>> 8;
              n |= n >>> 16;
              return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
          }
      ```

      

  - **初始化传参** 取接近2^n的最小值

    - 说明：**累计**的最大移位是2>>>(1+2+4+8+16)=29，因为 最大容量 MAXIMUM_CAPACITY = 1 << 30，如果初始化 介于2^29和2^30之间，在经过位或运算后，最后n + 1 = 2^30，所以**2>>>16**就终止了。
    - 例子
      - cap=3，则**初始化阈值** threshold=4
        - tableSizeFor()之后等于4

#### **后续阈值**计算=（初始化阈值*加载因子）的n次幂





## 2.resize()源码分析

### 前提

#### 两个参数的初始化

- threshold阈值
- table数组

#### 三种数据结构的扩容

- 数组节点
- 链表节点
- 红黑树节点



- ```java
  final Node<K,V>[] resize() {
          Node<K,V>[] oldTab = table;
          int oldCap = (oldTab == null) ? 0 : oldTab.length;
          int oldThr = threshold;
          int newCap, newThr = 0;
         /*****************1.开始*******************/
         /**
         * 初始化threshold 和 table 数组
         **/
          if (oldCap > 0) {
              if (oldCap >= MAXIMUM_CAPACITY) {
                  threshold = Integer.MAX_VALUE;
                  return oldTab;
              }
              else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                       oldCap >= DEFAULT_INITIAL_CAPACITY)
                  newThr = oldThr << 1; // double threshold
          }
          else if (oldThr > 0) // initial capacity was placed in threshold
              newCap = oldThr;
          else {               // zero initial threshold signifies using defaults
              newCap = DEFAULT_INITIAL_CAPACITY;
              newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
          }
          if (newThr == 0) {
              float ft = (float)newCap * loadFactor;
              newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                        (int)ft : Integer.MAX_VALUE);
          }
          threshold = newThr;
          @SuppressWarnings({"rawtypes","unchecked"})
              Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
          table = newTab;
        /******************1. 结束******************/
        /*****************2. 开始*******************/
         /**
         * 扩容，分配新的内存，移动位置
         **/
          if (oldTab != null) {
              for (int j = 0; j < oldCap; ++j) {
                  Node<K,V> e;
                  if ((e = oldTab[j]) != null) {
                      oldTab[j] = null;
                      /**
                       * 2.1 判断当前位置，只有一个元素，在原有数组位置移动
                      **/
                      if (e.next == null)
                          newTab[e.hash & (newCap - 1)] = e;
                      /**
                       * 2.2 判断红黑树节点
                      **/
                      else if (e instanceof TreeNode)
                          ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                      /**
                       * 2.3 判断是链表
                      **/
                      else { // preserve order
                          Node<K,V> loHead = null, loTail = null;
                          Node<K,V> hiHead = null, hiTail = null;
                          Node<K,V> next;
                          /**
                          *2.3.1 循环，并拼接链表
                          **/
                          do {
                              next = e.next;
                              /**
                              * 2.3.1判断是否需要保留在原来的位置，并拼接链表
                              **/
                              if ((e.hash & oldCap) == 0) {
                                  if (loTail == null)
                                      loHead = e;
                                  else
                                      loTail.next = e;
                                  loTail = e;
                              }
                              /**
                              * 2.3.1不保存在原来位置，并拼接链表
                              **/
                              else {
                                  if (hiTail == null)
                                      hiHead = e;
                                  else
                                      hiTail.next = e;
                                  hiTail = e;
                              }
                          } while ((e = next) != null);
                          /**
                          *2.3.2 设置原来位置的头结点
                          **/
                          if (loTail != null) {
                              loTail.next = null;
                              newTab[j] = loHead;
                          }
                          /**
                          *2.3.3 设置新位置（i+oldCap）的头结点
                          **/
                          if (hiTail != null) {
                              hiTail.next = null;
                              newTab[j + oldCap] = hiHead;
                          }
                      }
                  }
              }
          }
         /*****************2. 结束*******************/
          return newTab;
      }
  ```

  

