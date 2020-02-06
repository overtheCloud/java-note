## HashMap

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 初始化容器大小
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    // 最大容器大小
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 链表转二叉树的条件：链表节点数量达到8
    static final int TREEIFY_THRESHOLD = 8;
    // 二叉树转链表的条件：链表节点数量达到6
    static final int UNTREEIFY_THRESHOLD = 6;
    // 容器转二叉树的最小容量
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 实际存放数据的 Node 数组
    transient Node<K,V>[] table;
    // 键值对的 set
    transient Set<Map.Entry<K,V>> entrySet;
    // 实际存放的数据个数
    transient int size;
    // 修改次数
    transient int modCount;
    // 等于容量 * 负载因子
    int threshold;
    // 负载因子
    final float loadFactor;
    
    // 无参构造函数
    public HashMap() {
        // 设置默认负载因子，容量达到(负载因子 * 最大容量 = 16 * 0.75 = 12)时扩容
        this.loadFactor = DEFAULT_LOAD_FACTOR; 
    }
    public HashMap(int initialCapacity) {
        // 设置初始化容量
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
        this.loadFactor = loadFactor;
        // 设置扩容阈值为2的倍数中大于等于且最接近初始容量值的值，比如初始容量为3或者4时，初始容量值为4
        this.threshold = tableSizeFor(initialCapacity);
    }
    // 使用另一个 map 初始化
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
}
```

#### 节点对象

```java
// 链表节点
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

```java
// 二叉树节点
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
    
}
```

#### 插入数据

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 理解为哈希后对容器大小取模，此处是计算 key 在数组中的位置
    // 如果 key 对应位置没有数据则创建节点放入此位置
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // 如果 key 对应位置有值则创建节点添加到链表头部，并判断是否需要转二叉树
        Node<K,V> e; K k;
        // 如果链表头节点的 key 和插入数据的 key 相等则覆盖头节点
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果是二叉树则插入树节点
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 如果是链表则插入新节点到尾部
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 如果链表长度达到 7 则转成二叉树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // 无实现，LinkedHashMap 要实现的方法
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 插入完成后判断是否需要扩容
    if (++size > threshold)
        resize();
    // 无实现，LinkedHashMap 要实现的方法
    afterNodeInsertion(evict);
    return null;
}
```

#### 获取数据

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 如果不是第一个节点，则遍历链表或者二叉树找到对应节点
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

#### 扩容，HasmMap 默认扩容2倍且扩容后不会变小

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    // 计算新容器的大小和扩容阈值，默认扩容2倍
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 扩容一倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    } else if (oldThr > 0) // initial capacity was placed in threshold
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
    // 复制原容器的数据到新容器中
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

#### 链表转红黑树 `TreeNode.treeifyBin(Node<>K, V[] tab, int hash)`

```java
// 第一步：单链表转双向链表
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
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
            hd.treeify(tab);
    }
}
// 第二步：双向链表转搜索二叉树
final void treeify(Node<K,V>[] tab) {
    TreeNode<K,V> root = null;
    // this 是根节点
    for (TreeNode<K,V> x = this, next; x != null; x = next) {
        next = (TreeNode<K,V>)x.next;
        x.left = x.right = null;
        // 根节点
        if (root == null) {
            x.parent = null;
            // 根节点初始是黑色的
            x.red = false;
            root = x;
        }
        // 其他节点
        else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph;
                K pk = p.key;
                // 判断节点是上一节点的左节点还是右节点， -1 表示左节点，1 表示 右节点
                if ((ph = p.hash) > h) 
                    dir = -1;
                else if (ph < h) 
                    dir = 1;
                else if ((kc == null && 
                          // k 实现 Comparable 则 kc 为 k 的 class ， 否则为 null
                          (kc = comparableClassFor(k)) == null) ||
                         // 返回 k.compareTo(pk)
                         (dir = compareComparables(kc, k, pk)) == 0)
                    // 如果 k 和 pk 是同一类对象，则调用系统函数判断 k 和 pk 的哈希值大小 
                    dir = tieBreakOrder(k, pk);
			   // 将当前节点设置为上一节点的左节点或右节点
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

// 插入节点后判断是否需要移动节点使树平衡
static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root, TreeNode<K,V> x) {
    x.red = true;
    for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {、
        // 如果当前节点父节点为 null 表示当前节点是根节点
        if ((xp = x.parent) == null) {
            x.red = false;
            return x;
        }
        // 如果父节点是黑色或者祖父节点是根节点
        else if (!xp.red || (xpp = xp.parent) == null)
            return root;
        // 如果父节点是祖父节点的左节点
        if (xp == (xppl = xpp.left)) {
            // 如果祖父节点的右节点不为空且是红色，则父节点和叔叔节点变黑，祖父节点变红
            if ((xppr = xpp.right) != null && xppr.red) {
                xppr.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }
            else {
                if (x == xp.right) {
                    // 左旋转
                    root = rotateLeft(root, x = xp);
                    xpp = (xp = x.parent) == null ? null : xp.parent;
                }
                if (xp != null) {
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        // 右旋转
                        root = rotateRight(root, xpp);
                    }
                }
            }
        }
        // 如果父节点是祖父节点的右节点
        else {
            if (xppl != null && xppl.red) {
                xppl.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }
            else {
                if (x == xp.left) {
                    // 右旋转
                    root = rotateRight(root, x = xp);
                    xpp = (xp = x.parent) == null ? null : xp.parent;
                }
                if (xp != null) {
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        // 左旋转
                        root = rotateLeft(root, xpp);
                    }
                }
            }
        }
    }
}
// 左旋转
static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root, TreeNode<K,V> p) {
    TreeNode<K,V> r, pp, rl;
    if (p != null && (r = p.right) != null) {
        if ((rl = p.right = r.left) != null)
            rl.parent = p;
        if ((pp = r.parent = p.parent) == null)
            (root = r).red = false;
        else if (pp.left == p)
            pp.left = r;
        else
            pp.right = r;
        r.left = p;
        p.parent = r;
    }
    return root;
}
// 右旋转
static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root, TreeNode<K,V> p) {
    TreeNode<K,V> l, pp, lr;
    if (p != null && (l = p.left) != null) {
        if ((lr = p.left = l.right) != null)
            lr.parent = p;
        if ((pp = l.parent = p.parent) == null)
            (root = l).red = false;
        else if (pp.right == p)
            pp.right = l;
        else
            pp.left = l;
        l.right = p;
        p.parent = l;
    }
    return root;
}
static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
    int n;
    if (root != null && tab != null && (n = tab.length) > 0) {
        int index = (n - 1) & root.hash;
        TreeNode<K,V> first = (TreeNode<K,V>)tab[index];
        if (root != first) {
            Node<K,V> rn;
            tab[index] = root;
            TreeNode<K,V> rp = root.prev;
            if ((rn = root.next) != null)
                ((TreeNode<K,V>)rn).prev = rp;
            if (rp != null)
                rp.next = rn;
            if (first != null)
                first.prev = root;
            root.next = first;
            root.prev = null;
        }
        assert checkInvariants(root);
    }
}
```



### HashMap 在 1.8 与 1.7 的实现差异

1. hash 函数

```java
// 1.8 扰动一次，性能比 1.7 高 
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
// 1.7
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

2. 1.7 数据结构时数组加链表。1.8 数据结构是数组加链表，链表长度到 8 时链表转红黑树，链表长度缩短到 6 时红黑树转链表。

3. 1.7 中插入链表是头插法，1.8中插入链表是尾插法