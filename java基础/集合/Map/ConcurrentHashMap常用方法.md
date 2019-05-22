### 简介说明

​		在日常使用中，ConcurrentHashMap 被使用频率最高的应该就是 get 和 put 方法了，本章我们将要来详细解读它的 get、put 方法，探究 ConcurrentHashMap 在并发情况下如何保证存取数据的安全 。



***说明：该源码来自于jdk_1.8.0_162***



### get 方法

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 && (e = tabAt(tab, (n - 1) & h)) != null) {
        //如果所要找的元素就在数组上，直接返回结果
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //遇到 hash < 0 可能能有两种情况：
        //    (1) 该节点是TreeBin节点(红黑树代理节点，其hash值固定为：-2);
        //    (2) ConcurrentHashMap正在扩容当中，并且该hash桶已迁移完毕，该位置被放置了FWD节点;
        //1、如果是TreeBin节点，调用TreeBin类的find方法，具体是以链表方式遍历还是红黑树方式遍历视情况而定(后面细说)
        //2、如果正在扩容中，则跳转到扩容后的新数组上去查找，TreeBin和Node节点都有对应的find方法，具体什么节点类型则调用对应节点类型的find方法(后面细说)
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        //如果前面两种情况都不满足，说明该hash桶上面连接的是普通链表结构，使用while循环去遍历该链表节点
        while ((e = e.next) != null) {
            if (e.hash == h && ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

**接下来我们来看看非扩容情况下 eh < 0 这种情况（其他两种情况比较简单，这里不做展开）：**

###### 1、如果 e 节点为 TreeBin 节点，则使用 TreeBin 节点的 find 操作，该操作在下面的第2种情况中讲解 。

###### 2、如果 e 节点为 ForwardingNode 节点，遇到查询操作时直接将 get 操作转发到扩容后的新数组 nextTable 上去，接下来就是两种节点的 find 操作：

**|：nextTable上对应位置的节点为普通Node节点：**

```java
//使用 do...while 循环遍历普通链表，查找到则返回，找不到返回 null，比较简单
Node<K,V> find(int h, Object k) {
    Node<K,V> e = this;
    if (k != null) {
        do {
            K ek;
            if (e.hash == h && ((ek = e.key) == k || (ek != null && k.equals(ek))))
                return e;
        } while ((e = e.next) != null);
    }
    return null;
}
```

**||：nextTable上对应位置的节点为TreeBin节点：**

**这里面涉及到 TreeBin 类里面维护的锁这方面的内容，这里我同时也将该锁做一个详细的讲解：**

```java
final Node<K,V> find(int h, Object k) {
    if (k != null) {
        //使用 for 循环遍历红黑树，如果在以链表方式遍历遍历红黑树的过程中，发现锁状态变为 0 或者 READER (即对红黑树的改动已经完成)
        //则放弃使用链表方式遍历红黑树，而改为使用红黑树遍历，即便在原来已经使用链表方式遍历了一部分的情况下
        for (Node<K,V> e = first; e != null; ) {
            int s; K ek;
            
            //volatile int lockState; -->  lockState 不为 0 则说明该红黑树处于锁定状态，具体看下面的3种锁的含义
            //lockState的3种数值状态含义： 
            //    1 - (WRITER) - 二进制 001  --> 写锁
            //    2 - (WAITER) - 二进制 010  --> 等待写锁
            //    4 - (READER) - 二进制 100  --> 读锁
            
            //如果当前处于加锁状态，即有线程正在对红黑树进行写操作或者等待写操作，为了减少锁的竞争以便写操作尽快完成，查找操作将会以链表的方式去遍历红黑树节点
            if (((s = lockState) & (WAITER|WRITER)) != 0) {
                if (e.hash == h && ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
                e = e.next;
            }
            //如果当前没有加锁(写锁/等待写锁)，则先将 lockState + 4(READER)，也就是读锁的值叠加，然后以红黑树方式去遍历红黑树节点
            else if (U.compareAndSwapInt(this, LOCKSTATE, s, s + READER)) {
                TreeNode<K,V> r, p;
                try {
                    p = ((r = root) == null ? null : r.findTreeNode(h, k, null));
                } finally {
                    //进入该else if分支的每条线程在查找结束后将 lockState - 4(READER)
                    //U.getAndAddInt(this, LOCKSTATE, -READER) 这个操作是：LOCKSTATE = lockState - READER；内部使用 do...while 不断循环更新，直到更新成功为止，更新成功后返回更新之前的旧值
                    //如果：U.getAndAddInt(this, LOCKSTATE, -READER) == (READER|WAITER)；说明：当前线程是最后一条读线程|有线程持有等待锁 并且有等待写线程被阻塞着，则唤醒该等待线程，让阻塞的写线程去竞争写锁
                    Thread w;
                    if (U.getAndAddInt(this, LOCKSTATE, -READER) == (READER|WAITER) && (w = waiter) != null)
                        //这里唤醒的是contendedLock方法中因获取不到锁而使用LockSupport.park方法阻塞起来的线程
                        LockSupport.unpark(w);
                }
                return p;
            }
        }
    }
    return null;
}
```

**说明：**上面我们说到了在调用 TreeBin 节点的 find 方法时，通过对当前 TreeBin 内部自己实现的锁的状态判断，从而选择相应的遍历方式的做法，总结起来就是如下两种情况：

***(1)*** 当 get 操作遇到 WAITER 或者 WRITER 锁时直接以链表的方式去遍历查找目标元素，如果在查找期间 WAITER 和 WRITER 锁全部被释放掉了，则放弃使用链表方式遍历红黑树，而改为使用红黑树遍历，即便在原来已经使用链表方式遍历了一部分链表节点的情况下 。

***(2)*** 当 get 操作遇到没有 WAITER 和 WRITER 锁时，直接将 lockState 叠加上 4(READER) 的值，期间可能会有多个读线程在执行 get 操作，而当这些读取操作全部完成后，如果发现有等待写的线程被阻塞的话，该度线程会负责将等待写线程唤醒，让阻塞的写线程去竞争写锁 。

**问题：**如果在 lockState 为 READER 或者为其叠加值期间一直有读线程进来，写操作岂不是一直被阻塞？



**WAITER/WRITER：**前面我们说到了在 find 方法中线程如何对红黑树进行 READER 也就是读锁的设置，里面还涉及到了另外两个锁，那就是 WAITER(等待写锁) 和 WRITER(写锁)，下面我们来详细了解一下这两个锁是在什么情况下被设置的，以及他们之间有哪些互斥的关系 。

`先来看一段加锁的代码：`

```java
//该方法用于设置写锁(WRITER)，也就是在红黑树需要重构的情况下
private final void lockRoot() {
    //使用CAS方式设置lockState的值，设置成功则该方法执行完毕，如果设置失败则说明当前红黑树已经被其他线程正在修改
    if (!U.compareAndSwapInt(this, LOCKSTATE, 0, WRITER))
        //尝试占有写锁失败后调用contendedLock方法进行后续循环尝试获取写锁的动作
        contendedLock(); // offload to separate method
}
```

**说明：**该 lockRoot 方法看名字就知道他的目的是将红黑树的 root 节点锁住，该方法在两个地方被调用到；

***一是：***putTreeVal 红黑树插入元素后需要自平衡时，需要调用该方法将红黑树根节点锁住，然后再重新调整红黑树从而重新达到平衡状态 。

***二是：***removeTreeNode 对红黑树元素进行删除时需要将根节点锁住，再进行删除操作 。



`接下来我们继续看contendedLock方法：`

```java
//遇到设置写锁失败时才进入到该方法(不存在 写-写 阻塞，只存在 读-写 阻塞，因为数组上hash桶的修改操作已经使用synchronized同步锁互斥)，通过 for 循环继续尝试获取写锁
private final void contendedLock() {
    boolean waiting = false;
    //该循环是个死循环，只能通过内部第一个if分支的return退出
    for (int s;;) {
        //WAITER 二进制为：010
        //所以 ~WAITER 二进制为：101
        //所以 ((s = lockState) & ~WAITER) == 0 意思为当前红黑树没有任何线程对其持有 读锁 和 写锁，此时可以与其他线程竞争 写锁
        if (((s = lockState) & ~WAITER) == 0) {
            //竞争 写锁，如果竞争成功，拿到写锁将等待线程对象设置为null，获取写锁成功，退出方法，否则继续循环
            if (U.compareAndSwapInt(this, LOCKSTATE, s, WRITER)) {
                //如果之前有设置等待线程，则将其设置为null
                if (waiting)
                    waiter = null;
                return;
            }
        }
        //如果竞争写锁失败，并且当前红黑树黑没有等待写的线程，则尝试将当前线程设置为等待线程
        //(s & WAITER) == 0：lockState & 010 = 000；说明 lockState 不能存在3位二进制的中间数值等于1的情况，也就是目前还没有线程持有等待锁
        else if ((s & WAITER) == 0) {
            //竞争等待锁，如果获取到等待锁则将当前线程设置为等待线程并将等待标识设置为true
            //(s | WAITER)：这里可能出现两种情况的或运算：001|010=011; 100|010=110; 根据运算结果可知其实际上为一个lockState数值的叠加操作
            if (U.compareAndSwapInt(this, LOCKSTATE, s, s | WAITER)) {
                waiting = true;
                waiter = Thread.currentThread();
            }
        }
        //如果当前线程之前已经设置为了等待线程，并且经过循环后依旧未能竞争到写锁，为了避免一直循环耗费不必要的CPU资源，使用park方法使当前线程阻塞
        else if (waiting)
            //使线程进入阻塞状态，与这里相对应的是TreeBin中的find方法里面的LockSupport.unpark唤醒方法
            LockSupport.park(this);
    }
}
```

**说明：**线程在 lockRoot 方法设置 写锁 失败后进入该 contendedLock 方法继续死循环进行获取 写锁 的操作，直至获取到 写锁 才会退出该死循环 。进入该方法后首先进行 if 分支判断尝试获取 写锁 。如果获取失败则尝试进行第二个操作，设置为等待锁，如果当前红黑树已经有等待线程存在，则继续循环，直到获取到等待锁(或者写锁)，如果获取到等待锁，将当前线程设置为等待线程，并继续循环，如果循环后依旧未能竞争到写锁，为了避免一直循环耗费不必要的CPU资源，使用 LockSupport.park 方法使当前线程进入阻塞状态 。该阻塞状态的解除为上面说过的 find 方法中的 LockSupport.unpark(w) 操作 。

**注意：**因为红黑树内部不存在 写-写 互斥，只存在 读-写 互斥，所以 LockSupport.unpark 方法只需要在读相关的方法(例如：find)中存在即可，也就是只需要在读相关的方法中唤醒被阻塞的线程，而不需要在写的方法中存在(例如：putTreeVal) 。



`最后我们看一下unlockRoot方法：`

```java
//将 lockState 设置为 0 即可释放锁
private final void unlockRoot() {
    lockState = 0;
}
```

**说明：**该方法是伴随着 lockRoot 成对出现的，也就意味着该方法释放的是 写锁，同样也是在 putTreeVal 和 removeTreeNode 方法中有调用到 。



**总结一下，get 方法总共会遇到如下3种情形：**

(1) 非扩容情况下：遇到 get 操作，通过计算 (n - 1) & h 定位到具体的hash桶位置，如果数组上的hash桶就是目标元素则直接返回即可 。如果当前hash桶为普通Node节点链表，则使用普通链表方式去遍历该链表查找目标元素 。如果定位到的hash桶为TreeBin节点，则根据TreeBin内部维护的红黑树锁来确定具体采用哪种方式遍历查找元素，如果如果红黑树锁的状态为 写锁 / 等待写锁，则使用链表方式去遍历查找目标元素，而反之红黑树锁状态为 无锁 / 读锁，则使用红黑树方式去遍历查找目标元素，红黑树锁只存在 读-写 互斥而不存在 写-写 互斥 。

(2) 集合正在扩容并且当前hash桶正在迁移中：遇到 get 操作，在扩容过程期间会形成 hn 和 ln链，形成这两条中间链是使用的类似于复制引用的方式，也就是说 ln 和 hn 链是复制出来的，而非原hash桶的链表剪切过去的，所以原来 hash 桶上的链表并没有受到影响，因此从迁移开始到迁移结束这段时间都是可以正常访问原数组 hash 桶上面的链表，具体访问方式同上面的 (1) 点 。

(3) 集合扩容还未结束但是当前hash桶已经迁移完成：遇到 get 操作，每迁移完一个hash桶后当前hash桶的位置都会被替换成 ForwardingNode 节点，遇到 get 操作时直接将查找操作转发到新的数组上去，也就是直接到新数组上面查找目标元素，具体的查找方式依旧跟上面的 (1) 点相同 。





### **put 方法**

```java
//put方法实际上是调用的putVal方法
public V put(K key, V value) {
    return putVal(key, value, false);
}
```

`接下来继续看 putVal 方法：`

```java
//该方法为put等方法实际调用的方法，第三个参数onlyIfAbsent用于控制是否覆盖已有的旧值
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    //首先对key的原始hash值进行扰动，使元素在数组上面的分布更加均匀
    //这部分的详细内容可以参考我的另一篇博客：https://blog.csdn.net/ZOKEKAI/article/details/90085517
    int hash = spread(key.hashCode());
    int binCount = 0;
    //进入一个for的死循环，遇到扩容的情况时先帮助扩容，然后继续循环，直至扩容完毕后再执行插入元素的操作
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        //如果当前ConcurrentHashMap还未初始化则先进行初始化操作
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        //如果定位到的数组位置为null，则直接使用CAS的方式将元素设置到该位置上
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            //使用CAS的方式将元素设置到该位置上，如果设置失败，也就是期间该位置被其他线程先一步设置值到了该位置，则会继续循环，使用后面的链表的方式插入元素
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //如果发现定位到的节点的hash值为 -1(MOCED)，说明该集合正处于扩容阶段，并且定位到的该hash桶已迁移完毕，该则优先帮助扩容集合进行扩容，扩容完毕后再继续循环，直至成功插入元素为止
        else if ((fh = f.hash) == MOVED)
            //进行帮助扩容操作，扩容部分的详细内容可以参考我的另一篇博客：https://blog.csdn.net/ZOKEKAI/article/details/90051567
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            //经过了上面的判断，到达了这里，说明集合不在扩容期间，并且定位到的数组上的hash桶不为空，则先将该数组上的位置的节点锁住
            synchronized (f) {
                //这里之所以需要再次判断我们之前锁住的f节点是否和现在i位置上的节点是否相等目的是为了防止在加锁的这段时间该位置i上的节点被替换或者删除，从而可能引起的错误
                if (tabAt(tab, i) == f) {
                    //fh大于0说明该节点为普通的Node链表节点
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //如果链表中存在与新增的Node节点的key相同的节点，则根据onlyIfAbsent属性决定是否覆盖链表中该节点的旧value
                            if (e.hash == hash && ((ek = e.key) == key || (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            //如果遍历完了链表的所有节点都没有与新增的Node节点有相同key值的节点，则采用尾插法将Node节点拼接在链表的最后
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value, null);
                                break;
                            }
                        }
                    }
                    //如果定位到的节点为红黑树的TreeBin节点，则以红黑树的方式插入新元素
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        //使用红黑树方式插入新元素，执行putTreeVal方法时，如果发现目标位置为null，则直接插入，如果目标位置已经有值，则仅会返回旧节点，然后根据onlyIfAbsent属性决定是否覆盖链表中该节点的旧value
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            //这里面的binCount属性的值是上面的普通Node链表的长度
            if (binCount != 0) {
                //如果链表长度大于或者等于TREEIFY_THRESHOLD(这里是8)，调用treeifyBin方法尝试将其转化为红黑树，这里为什么用尝试而不用肯定的说法，下面会解释
                if (binCount >= TREEIFY_THRESHOLD)
                    //treeifyBin的操作是，判断当前数组长度是否小于64，如果小于64则会进行扩容，而不会转化为红黑树，只有数组长度大于或等于64时遇到链表长度达到8时才会将链表转化为红黑树
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    //最后将集合容量进行加1操作，计数操作后面会详解
    addCount(1L, binCount);
    return null;
}
```

`接下来我们继续看红黑树的插入方法：`

```java
//该方法如果发现要插入的位置为null，则直接插入，如果目标位置已经有值，则将该旧节点返回给调用者
final TreeNode<K,V> putTreeVal(int h, K k, V v) {
    Class<?> kc = null;
    boolean searched = false;
    for (TreeNode<K,V> p = root;;) {
        int dir, ph; K pk;
        //如果红黑树根节点为null，则直接将该新节点设置为红黑树根节点
        if (p == null) {
            first = root = new TreeNode<K,V>(h, k, v, null, null);
            break;
        }
        else if ((ph = p.hash) > h)
            dir = -1;
        else if (ph < h)
            dir = 1;
        else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
            return p;
        else if ((kc == null && (kc = comparableClassFor(k)) == null) || (dir = compareComparables(kc, k, pk)) == 0) {
            if (!searched) {
                TreeNode<K,V> q, ch;
                searched = true;
                if (((ch = p.left) != null && (q = ch.findTreeNode(h, k, kc)) != null) || ((ch = p.right) != null && (q = ch.findTreeNode(h, k, kc)) != null))
                    return q;
            }
            dir = tieBreakOrder(k, pk);
        }

        TreeNode<K,V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            TreeNode<K,V> x, f = first;
            first = x = new TreeNode<K,V>(h, k, v, f, xp);
            if (f != null)
                f.prev = x;
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            if (!xp.red)
                x.red = true;
            else {
                //不符合红黑树结构要求，需要进行插入自平衡操作，操作前需要进行加锁操作，调用lockRoot方法争夺写锁，具体实现请看上文对红黑树锁的介绍
                lockRoot();
                try {
                    root = balanceInsertion(root, x);
                } finally {
                    //红黑树进行平衡重构后释放持有的写锁
                    unlockRoot();
                }
            }
            break;
        }
    }
    //最后使用递归方法进行红黑树结构检查，检查修改过后的树是否依旧符合红黑树结构要求
    assert checkInvariants(root);
    return null;
}
```



**总结一下，put 方法总共会遇到如下几种情形：**

***(1)*** 集合还未初始化：进行集合的初始化操作，该操作会将设置一个全局的初始化标识 sizeCtl = -1，当其他线程检测到 sizeCtl 的值为 -1 时就会使用 Thread.yield() 方法让出 CPU 资源，让初始化线程能够更快完成初始化操作，同时也保证了只能有一条线程对集合进行初始化 。

***(2)*** 定位到的目标位置在数组上，并且该位置的值为 null：为了避免线程安全问题，使用CAS方式将元素直接设置到该数组位置上 。

***(3)*** 定位到的目标位置在数组上，并且该位置的值为 ForwardingNode 节点：说明此时集合在扩容中，并且当前定位到的节点的hash桶已经迁移完毕，此时执行put操作的线程会优先加入到扩容大军里面去，加速扩容速度，待扩容完成后再继续循环插入新元素 。

***(4)*** 定位到的目标位置在数组上，并且该位置已经有其他值：先锁住位于数组上的头结点 。如果节点类型是普通链表节点，使用尾插法在末尾拼接上新的节点 。如果节点类型是 TreeBin 节点，调用 TreeBin 的 putTreeVal 方法 。putTreeVal 方法具体做法为，如果目标位置为 null，则直接添加进去元素，如果目标位置已经有值，则返回旧值，根据 onlyIfAbsent 属性决定是否覆盖该红黑树上面的旧值 。