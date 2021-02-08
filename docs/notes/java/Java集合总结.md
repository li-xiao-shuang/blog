# 集合

# ArrayList

ArrayList 实现于 List、RandomAccess 接口。可以插入空数据，也支持随机访问。其中最重要的两个属性分别是: elementData 数组，以及 size 大小。 默认初始化容量为10，每次扩容会扩容1.5倍(新容量=旧容量+旧容量>>1)。有序、非线程安全的。

**执行add(E)方法：**

```java
/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        modCount++;
        add(e, elementData, size);
        return true;
    }

/**
     * This helper method split out from add(E) to keep method
     * bytecode size under 35 (the -XX:MaxInlineSize default value),
     * which helps when add(E) is called in a C1-compiled loop.
     */
    private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;
        size = s + 1;
    }
```

- 首先记录对该列表进行结构修改的次数
- 然后执行添加元素，默认添加到末尾
- 判断数组的容量是否满了，如果是就先进行扩容
- 将元素添加到指定位置，修改size大小

**执行add(index,e)方法,添加元素到指定位置:**

```java
/**
     * Inserts the specified element at the specified position in this
     * list. Shifts the element currently at that position (if any) and
     * any subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);
        modCount++;
        final int s;
        Object[] elementData;
        if ((s = size) == (elementData = this.elementData).length)
            elementData = grow();
        System.arraycopy(elementData, index,
                         elementData, index + 1,
                         s - index);
        elementData[index] = element;
        size = s + 1;
    }
```

- check下标是否越界，并记录列表结构修改次数
- 判断数组是否需要扩容
- 通过System.arraycopy方法复制指定的元素向后移动
- 将添加的元素赋值给指定的下标 ，修改size大小

**扩容方法grow()：**

```java
    private Object[] grow() {
        return grow(size + 1);
    }
/**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     * @throws OutOfMemoryError if minCapacity is less than zero
     */
    private Object[] grow(int minCapacity) {
        return elementData = Arrays.copyOf(elementData,
                                           newCapacity(minCapacity));
    }

/**
     * Returns a capacity at least as large as the given minimum capacity.
     * Returns the current capacity increased by 50% if that suffices.
     * Will not return a capacity greater than MAX_ARRAY_SIZE unless
     * the given minimum capacity is greater than MAX_ARRAY_SIZE.
     *
     * @param minCapacity the desired minimum capacity
     * @throws OutOfMemoryError if minCapacity is less than zero
     */
    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity <= 0) {
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                return Math.max(DEFAULT_CAPACITY, minCapacity);
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return minCapacity;
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE)
            ? Integer.MAX_VALUE
            : MAX_ARRAY_SIZE;
    }
```

- 主要是通过旧的容量+旧的容量，然后右位移1位来计算新的容量
- 通过容量创建一个新的数组，进行数组复制。

# Vector

底层使用数组实现，扩容方式与List不同，如果初始化Vector时没有指定容量增量，那么会默认扩容2倍（新容量=旧容量+旧容量），如果指定了容量增量，那么扩容的容量就是（新容量=旧容量+容量增量），使用synchronized包装了类的方法，所以是线程安全的。并且有序。

**执行add(E e)方法：**

```java
/**
     * Appends the specified element to the end of this Vector.
     *
     * @param e element to be appended to this Vector
     * @return {@code true} (as specified by {@link Collection#add})
     * @since 1.2
     */
    public synchronized boolean add(E e) {
        modCount++;
        add(e, elementData, elementCount);
        return true;
    }
/**
     * This helper method split out from add(E) to keep method
     * bytecode size under 35 (the -XX:MaxInlineSize default value),
     * which helps when add(E) is called in a C1-compiled loop.
     */
    private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;
        elementCount = s + 1;
    }
```

- 判断是否需要扩容
- 赋值元素到指定的下标
- 修改容器大小

# LinkedList

采用双向链表实现，get指定索引的值会先对链表的大小进行右位移1，来判断获取的索引值在链表的上半部分还是下半部分，如果是上半部分会从头部节点开始遍历查找，如果是下半部分会从尾部节点开始遍历查找，有序、非线程安全。

**执行add(E e)方法：**

```java
/**
     * Appends the specified element to the end of this list.
     *
     * <p>This method is equivalent to {@link #addLast}.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
/**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

- 默认添加到链表的最后面，先取出lastNode的一个临时变量。
- 将要添加的元素包装成一个newNode节点，将newNode节点的前置节点设置为当前链表的最后一个节点
- 将newNode设置为新的last节点
- 判断lastNode是否为空，如果为空，那么此时添加的是链表的第一个节点，那么直接设置firstNode等于newNode。如果不是就将newNode链接到lastNode后边
- 修改改链表大小及结构修改次数

**执行get(int index)方法:**

```java
/**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
/**
     * Returns the (non-null) Node at the specified element index.
     */
    Node<E> node(int index) {
        // assert isElementIndex(index);

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

- 先判断要获取的索引是否超出链表大小
- 通过将size左位移一位来判断index是在链表的上半部分还是下半部分
- 如果在上半部分则通过头节点开始遍历查找
- 如果在下半部分则通过尾节点开始遍历查找

# HashSet

底层使用HashMap实现,所有添加到set中的元素最终都会添加到map的key中，value用一个final的object对象填充。无序不重复，非线程安全

# TreeSet

底层使用NavigableMap实现,所有添加到set中的元素最终都会添加到map的key中，value用一个final的object对象填充。有序不重复，非线程安全

# HashMap

HashMap初始容量为16的Note数组，数组内的链表容量大于8时会自动转换为红黑树，只有当这个数大于2并值至少有8个才能满足树的假设，当这个值缩小到6的时候就会转换为链表。

**在hashMap中get操作：**

```java
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    /**
     * Implements Map.get and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @return the node, or null if none
     */
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

- 计算key的hash值，判断get的元素是不是firstNoe，如果直接返回
- 如果不是firstNode,那么判断是否是树，如果是的话通过遍历树查找。
- 否则遍历链表找到key相等的值。

**在hashMap中put操作：**

```java
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

- 计算key的hash值，算出元素在底层数组中的下标位置。如果下标位置为空，直接插入。
- 通过下标位置定位到底层数组里的元素（也有可能是链表也有可能是树）。
- 取到元素，判断放入元素的key是否==或equals当前位置的key，成立则替换value值，返回旧值。
- 如果是树，循环树中的节点，判断放入元素的key是否==或equals节点的key，成立则替换树里的value，并返回旧值，不成立就添加到树里。
- 否则就顺着元素的链表结构循环节点，判断放入元素的key是否==或equals节点的key，成立则替换链表里value，并返回旧值，找不到就添加到链表的最后。
- 精简一下，判断放入HashMap中的元素要不要替换当前节点的元素，key满足以下两个条件即可替换：
  - **hash值相等。**
  - **==或equals的结果为true。**

# LinkedHashMap

主体的实现都是借助于 HashMap 来完成的，只是对其中的 recordAccess(), addEntry(), createEntry() 进行了重写。

总的来说 `LinkedHashMap` 其实就是对 `HashMap` 进行了拓展，使用了双向链表来保证了顺序性。因为是继承与 `HashMap` 的，所以一些 `HashMap` 存在的问题 `LinkedHashMap` 也会存在，比如不支持并发等。

`LinkedHashMap` 的排序方式有两种：

- 根据写入顺序排序。
- 根据访问顺序排序。

# ConcurrentHashMap

## jdk1.7实现

![%E9%9B%86%E5%90%88%206550f84d83244b8e84fe1bf2b22a7720/Untitled.png](https://tva1.sinaimg.cn/large/008eGmZEly1gng6z2y6irj30dw0730v0.jpg)

如图所示，是由 `Segment` 数组、`HashEntry` 数组组成，和 `HashMap` 一样，仍然是数组加链表组成。

`ConcurrentHashMap` 采用了分段锁技术，其中 `Segment` 继承于 `ReentrantLock`。不会像 `HashTable` 那样不管是 `put` 还是 `get` 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 `CurrencyLevel` (Segment 数组数量)的线程并发。每当一个线程占用锁访问一个 `Segment` 时，不会影响到其他的 `Segment`。

### get方法

`ConcurrentHashMap` 的 `get` 方法是非常高效的，因为整个过程都不需要加锁。

只需要将 `Key` 通过 `Hash` 之后定位到具体的 `Segment` ，再通过一次 `Hash` 定位到具体的元素上。由于 `HashEntry` 中的 `value` 属性是用 `volatile` 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值(**[volatile 相关知识点](https://github.com/crossoverJie/Java-Interview/blob/master/MD/Threadcore.md#%E5%8F%AF%E8%A7%81%E6%80%A7)**)。

### put 方法

内部利用HashEntry类存储数据。

虽然 HashEntry 中的 value 是用 `volatile` 关键词修饰的，但是并不能保证并发的原子性，所以 put 操作时仍然需要加锁处理。

首先也是通过 Key 的 Hash 定位到具体的 Segment，在 put 之前会进行一次扩容校验。这里比 HashMap 要好的一点是：HashMap 是插入元素之后再看是否需要扩容，有可能扩容之后后续就没有插入就浪费了本次扩容(扩容非常消耗性能)。

而 ConcurrentHashMap 不一样，它是在将数据插入之前检查是否需要扩容，之后再做插入操作。

## JDK1.8 实现

![%E9%9B%86%E5%90%88%206550f84d83244b8e84fe1bf2b22a7720/Untitled%201.png](https://tva1.sinaimg.cn/large/008eGmZEly1gng6yyf4jmj30lp0drgp3.jpg)

1.8 中的 ConcurrentHashMap 数据结构和实现与 1.7 还是有着明显的差异。其中抛弃了原有的 Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性。也将 1.7 中存放数据的 HashEntry 改为 Node，但作用都是相同的。其中的 val和next 字段都用了 volatile 修饰，保证了可见性。

### put方法

![%E9%9B%86%E5%90%88%206550f84d83244b8e84fe1bf2b22a7720/Untitled%202.png](https://tva1.sinaimg.cn/large/008eGmZEly1gng6ytzyv6j30oc0rb7hq.jpg)

- 根据 key 计算出 hashcode 。
- 判断是否需要进行初始化。
- `f` 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
- 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。
- 如果都不满足，则利用 synchronized 锁写入数据。
- 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树

### get方法

![%E9%9B%86%E5%90%88%206550f84d83244b8e84fe1bf2b22a7720/Untitled%203.png](https://tva1.sinaimg.cn/large/008eGmZEly1gng6yo59lpj30o409hwj0.jpg)

- 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
- 如果是红黑树那就按照树的方式获取值。
- 都不满足那就按照链表的方式遍历获取值。