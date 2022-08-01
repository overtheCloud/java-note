# ArrayList

ArrayList 底层是数组实现，支持随机读写，查找时间复杂度 O(1)，新增和删除时间复杂度 O(n)。

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
    // 初始容量
    private static final int DEFAULT_CAPACITY = 10;
    // 实际存放数据的数组，transient 表示不参与序列化
    transient Object[] elementData;
    // 数组中元素个数
    private int size;
    
    /*
    *  无参构造函数返回空数组
    */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    
    /*
    * 有参构造函数指定容量大小
    */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
        }
    }
    
    /*
    * 用集合初始化
    */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // 如果目标集合元素不是 Object 则转换成 Object
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
    
    /*
    * 具体实现是 System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
    */
     public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
}
```

#### 添加元素

```java
public boolean add(E e) {
    // 是否扩容
    ensureCapacityInternal(size + 1);  
    // 添加新元素
    elementData[size++] = e;
    return true;
}
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
/*
* 获取容量
*/
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 空数组则取默认容量大小和添加元素后的容器大小的最大值
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
private void ensureExplicitCapacity(int minCapacity) {
    // 修改次数 + 1
    modCount++;

    // 如果当前数组元素个数超过数组容量则扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
/*
* @param minCapacity 最小容量
*/
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    // newCapacity = 1.5 * oldCapacity
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 取扩容后数组大小为 1.5 倍原数组大小和最小容量的最大值
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 数组最大容量限定为 Integer.MAX_VALUE（0x7fffffff）	
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 移动数据元素到新数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

#### 删除元素

```java
public E remove(int index) {
    // 判断 index 是否超过数组元素个数
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);
	// 移动删除元素后的元素到新位置
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

#### 获取元素

```java
public E get(int index) {
     // 判断 index 是否超过数组元素个数
    rangeCheck(index);

    return elementData(index);
}
```

#### 替换元素

```java
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

#### 与 Vector 的区别

```java
public class Vector<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {... ...}
```

Vector 的方法使用 `synchronized` 保证同步，线程安全，但是效率低。List 线程不安全，可以通过 `Collections.synchronizedList(List<T> list)` 将 list 转换成线程安全的。

