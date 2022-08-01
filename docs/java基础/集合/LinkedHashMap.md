# LinkedHashMap

继承自 HashMap，最大的区别是 LinkedHashMap 是有序的，增加了 accessOrder 属性，当 accessOrder 为 true 时，每次获取节点都会把对应节点移动到链表尾部，可以用作 LRU 算法的实现。

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V> {
    // 头节点
	transient LinkedHashMap.Entry<K,V> head;
    // 尾节点
    transient LinkedHashMap.Entry<K,V> tail;
    // true : 每次获取的时候把获取的节点移动到链表尾部
    final boolean accessOrder;
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }
    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super();
        accessOrder = false;
        putMapEntries(m, false);
    }
    public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
}
```

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

### 插入数据

LinkedHashMap 没有重写 put 方法，所以还是调用的 HashMap 的put 方法，但是 LinkedHashMap  重写了 afterNodeAccess 、afterNodeInsertion 和 afterNodeRemoval 方法。

- afterNodeAccess 移动节点到链表尾部
- afterNodeInsertion 插入节点后的处理操作
- afterNodeRemoval  移除节点后的处理操作

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    ... ...
    if (e != null) { // existing mapping for key
        V oldValue = e.value;
        if (!onlyIfAbsent || oldValue == null)
            e.value = value;
            // hash 冲突后把节点移动到链表尾部
            afterNodeAccess(e);
            return oldValue;
        }
	}
    ... ...
    //完成新数据put后调用，evict = true
    afterNodeInsertion(evict);     
}
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        // 移除链表的头节点，removeNode 方法是调用的 HashMap 的，在删除节点后会再调用 LinkedHashMap 的 afterNodeRemoval 方法
        removeNode(hash(key), key, null, false, true);
    }
}
// 实现自己的 LRU 时重写次方法
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
// 移除节点
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p = (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

### 获取数据

```java
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    // accessOrder 为 ture 时移动节点到链表尾部
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}
// 移动节点到链表尾部
void afterNodeAccess(Node<K,V> e) {
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p = (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

### 获取数据，如果对应数据不存在则返回默认值

```java
public V getOrDefault(Object key, V defaultValue) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return defaultValue;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}
```

### 清空链表

```java
public void clear() {
    super.clear();
    head = tail = null;
}
```

### 手写 LRU 缓存

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int CACHE_SIZE;

    /**
     * @param cacheSize 缓存大小
     */
    public LRUCache(int cacheSize) {
        // true 表示让 linkedHashMap 按照访问顺序来进行排序，最近访问的放在头部，最老访问的放在尾部。
        super((int) Math.ceil(cacheSize / 0.75) + 1, 0.75f, true);
        CACHE_SIZE = cacheSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 当 map中的数据量大于指定的缓存个数的时候，就自动删除最老的数据。
        return size() > CACHE_SIZE;
    }
}
```

