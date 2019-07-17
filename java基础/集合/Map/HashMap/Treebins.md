# JDK1.8的 树节点 以及 红黑树 算法实现1

## 1.前提，put方法下的操作

- 1.1 数组结构大于64

- 1.2 链表长度大于8

- 1.3 二叉查找树 和 红黑树

  - 二叉查找树

    ```properties
    若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
    若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
    任意节点的左、右子树也分别为二叉查找树。
    没有键值相等的节点（no duplicate nodes）。
    ```

  - 红黑树（在二叉查找树的基础上）

    ```properties
    每个结点要么是红的要么是黑的。  
    根结点是黑的。  
    每个叶结点（叶结点即指树尾端NIL指针或NULL结点）都是黑的。  
    如果一个结点是红的，那么它的两个儿子都是黑的。  
    对于任意结点而言，其到叶结点树尾端NIL指针的每条路径都包含相同数目的黑结点。 
    ```

    

## 2.普通元素节点 Node 和树节点 TreeNode

```java
    /**
     * 普通元素节点 Node，单向线性链表，遍历自增
     *
     * Basic hash bin node, used for most entries.  (See below for
     * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
     */
    static class Node<K,V> implements Map.Entry<K,V> {
            final int hash;
            final K key;
            V value;
            Node<K,V> next;
    }

    // Tree bins

    /**
     * 树节点 TreeNode ，单向线性链表
     *
     * Entry for Tree bins. Extends LinkedHashMap.Entry (which in turn
     * extends Node) so can be used as extension of either regular or
     * linked node.
     */
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
    }
```



## 2.链表结构的treeifyBin方法

2.1 当链表长度大于8

```java
   /**
     * put方法中，新增的值放在链表后面
     * 
     */
for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //长度大于7个数量（binCount为 0到6）
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
```

```java
    /**
     * Replaces all linked nodes in bin at index for given hash unless
     * table is too small, in which case resizes instead.
     * 是否需要树形化
     */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //数组长度小于64，数组自增
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        //开始树形化
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                //找到根节点hd，也就是TreeNode链表，第一次树形化
                hd.treeify(tab);
        }
    }
```

```java
       /**
         * Forms tree of the nodes linked from this node.
         * @return root of tree
         * 构造红黑树
         */
        final void treeify(Node<K,V>[] tab) {
            TreeNode<K,V> root = null;
            for (TreeNode<K,V> x = this, next; x != null; x = next) {
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;
                if (root == null) {
                    x.parent = null;
                    x.red = false;
                    root = x;
                }
                else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    for (TreeNode<K,V> p = root;;) {
                        int dir, ph;
                        K pk = p.key;
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);

                        TreeNode<K,V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            root = balanceInsertion(root, x);
                            break;
                        }
                    }
                }
            }
            moveRootToFront(tab, root);
        }
```



## 3.树节点的putTreeVal方法