# LinkedList

双向链表实现的 List，查询时间复杂度 O(n)， 插入和删除时间复杂度 O(1)。实现 Deque 接口使得其拥有队列的特性。非线程安全，使用 `Collections.synchronizedList(List<T> list)` 得到线程安全的集合。

```java
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable {
    // 元素个数
	transient int size = 0;
    // 双向链表的头节点
    transient Node<E> first;
    // 双向链表的尾节点
    transient Node<E> last;
    /*
    * 无参构造参数
    */ 
    public LinkedList() {}
    /*
    * 用集合初始化
    */
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
    private static class Node<E> {
        // 元素数据
        E item;
        // 下一个节点
        Node<E> next;
        // 上一个节点
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
}
```

#### 添加元素

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}
/*
* 从尾部插入元素
*/
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    // 尾节点为空表示空链表
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

移除元素

```java
/*
* 根据节点元素数据移除节点
* 循环链表找到节点后移除该节点
*/
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
/**
* 将节点的上一个节点的 next 指向下一个节点，将下一个节点的 prev 指向上一个节点
**/
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
/*
* 根据索引值移除节点
*/
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
/*
* 查找索引值对应的节点
*/ 
Node<E> node(int index) {
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

#### 获取元素

```java
public E get(int index) {
    // 检查索引值是否超过范围
    checkElementIndex(index);
    return node(index).item;
}
```

#### 替换节点数据

```java
public E set(int index, E element) {
    // 检查索引值是否超过范围
    checkElementIndex(index);
    // 获取索引值对应节点
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```

#### 获取数据所在节点的索引值

```java
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```

### 继承自 Deque 的接口

```java
// 获取第一个节点数据
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}
// 获取最后一个节点数据
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
// 移除第一个节点
 public E removeFirst() {
     final Node<E> f = first;
     if (f == null)
         throw new NoSuchElementException();
     return unlinkFirst(f);
 }
// 移除最后一个节点
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}
// 添加数据到第一个节点
public void addFirst(E e) {
    linkFirst(e);
}
// 添加数据到最后一个节点
public void addLast(E e) {
    linkLast(e);
}
// 获取第一个节点数据
 public E peek() {
     final Node<E> f = first;
     return (f == null) ? null : f.item;
 }
// 获取第一个节点数据
public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
// 获取最后一个节点数据
public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}
// 获取第一个节点数据，如果链表为空抛出异常
public E element() {
    return getFirst();
}
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}
// 移除第一个节点并返回第一个节点数据
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
// 移除第一个节点并返回第一个节点数据
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
// 移除最后一个节点并返回最后一个节点数据
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}
// 移除第一个节点并返回第一个节点数据，如果链表为空抛出异常
public E remove() {
    return removeFirst();
}
 public E removeFirst() {
     final Node<E> f = first;
     if (f == null)
         throw new NoSuchElementException();
     return unlinkFirst(f);
 }
// 添加数据到最后一个节点
public boolean offer(E e) {
    return add(e);
}
// 添加数据到第一个节点
 public boolean offerFirst(E e) {
     addFirst(e);
     return true;
 }
// 添加数据到最后一个节点
public boolean offerLast(E e) {
    addLast(e);
    return true;
}
// 进栈：向栈顶添加节点
public void push(E e) {
    addFirst(e);
}
// 出栈：从栈顶移除节点
public E pop() {
    return removeFirst();
}
```

#### 

```java

```

