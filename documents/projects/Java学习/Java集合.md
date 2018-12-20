# 一、概览
Java集合主要包括Collection和Map两类，其中Collection存储对象的集合，Map存储键值对的映射表。
## Collection
继承关系如图所示
![2018-12-12.21.52.00-Collection.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-12.21.52.00-Collection.png)
### 1.Set
- TreeSet：基于红黑树实现，执行有序性操作，但是查询效率不如HashSet，HashSet查询时间复杂度为O(1)，TreeSet查询复杂度为O(logN)
- HashSet：基于哈希表实现，支持快速查找，但不支持有序查找，且失去了元素的插入顺序信息，因此使用iterator遍历的结果是不确定的。
- LinkedHashSet：具有HashSet查询效率，且内部维护双向链表维护元素插入顺序。
### 2.List
- ArrayList：基于动态数组实现，支持随机访问。
- Vetor：和ArrayList类似，线程安全。
- LinkedList：基于双向链表实现，可以快速实现删除和插入元素，但是不能随机访问，只能顺序访问。可作为栈、队列和双向队列
### Queue
- LinkedList：用于实现双向队列
- PriorityQueue:基于堆结构的实现，可以用来实现优先队列
## Map
继承关系如图所示
![2018-12-12.22.34.56-Map.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-12.22.34.56-Map.png)
- TreeMap：基于红黑树实现。
- HashMap：基于哈希表实现。
- HashTable：和HashMap一样，线程安全的。不推荐使用。如果需要线程安全推荐使用ConcurrentHashMap，效率更高。
- LinkedHashMap：使用双向链表维护元素顺序，顺序为插入顺序或者最近最少使用顺序（LRU）

# 二、容器中设计模式
## 1.迭代器模式
![2018-12-12.23.17.03-iterator.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-12.23.17.03-iterator.png)
Collection集成Iterable接口，其中iterator()方法能够产生一个Iterator对象，通过该对象可以遍历Collection中对象。
从JDK1.5开始用使用foreach遍历Iterator聚合对象。
## 2.适配器模式
java.util.Arrays#asList()可以把数组类型转换为List对象
```java
	@SafeVarargs
    public static <T> List<T> asList(T... a)
```
# 三、源码分析
## ArrayList
### 1.概览
实现了RandomAccess接口支持随机访问，ArrayList是基于数组实现的。
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```
数组的默认大小是10.
```java
private static final int DEFAULT_CAPACITY = 10;
```
### 2.扩容
添加元素时使用`ensureCapacityInternal(int minCapacity)`保证容量足够，如果容量不够则使用grow方法来扩容。新的数组容量为原先数组的1.5倍。扩容操作需要`Arrays.copyOf(elementData, newCapacity)`方法使用将原数组复制到新数组。复制操作代价巨大，建议在初始化是确定好数组大小，减少扩容操作。
```java
	// 保证数组容量足够
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            : DEFAULT_CAPACITY;
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 扩容容量为原先的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        // 扩容时需要将原数组复制到新数组
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
### 3.删除元素
调用`System.arraycopy(elementData, index+1, elementData, index, numMoved)`将index+1后面的元素复制到index位置上，操作时间复杂度O(N),删除代价非常大。
```java
public E remove(int index) {
	rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
         System.arraycopy(elementData, index+1, elementData, index, numMoved);
         elementData[--size] = null; // clear to let GC do its work
         return oldValue;
    }
```
### 4.Fail-fast
`modCount`用于记录ArrayList结构变化的次数。结构变化是指添加或删除至少一个元素的操作。在进行序列化或者迭代操作时需要比较操作前后`modCount`是否改变，若改变则抛出`ConcurrentModificationException`.
```java
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```
### 5.序列化
ArrayList是基于数组实现的，因此保存的元素不一定都会被使用，因此没必要对所有的元素序列化。保存数组`elementData`是用transient关键字修饰的，该关键字声明的数组默认不能被序列化。
```java
transient Object[] elementData;
```
ArrayList实现了writeObject()和readObject()来控制只序列化有元素填充的那部分内容。
```java
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    /**
     * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is,
     * deserialize it).
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```
序列化需要使用ObjectOutputStream的writeObject()方法将对象转换为字节流输出。而writeObject()方法在传入的对象存在writeObject()的时候会反射调用对象的writeObject()方法实现序列化。反序列化使用ObjectInputStream的readObject()方法实现。
## Vector
### 1.同步
原理与ArrayList类似，但是使用synchronized关键字进行同步。
### 2.与ArrayList对比
- Vector是线程同步的，开销大，访问操作速度慢
- Vector每次扩容是2倍，ArrayList是1.5倍
### 3.替代方案：解决线程安全
- 使用`Collections.synchronizedList(list)`得到一个线程安全的List
```java
List<String> list = new ArrayList<>();
List<String> stringList = Collections.synchronizedList(list);
```
- 使用`java.util.concurrent.CopyOnWriteArrayList`
## CopyOnWriteArrayList
### 特性：读写分离
写操作在一个复制的数组上进行，读操作在原数组上进行。<br>
写操作需要加锁，防止并发写入导致写数据丢失。<br>
写操作结束后需要原始数据数组指向新的数组。
```java
    public boolean add(E e) {
    	// 写操作前先加锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
    // 写操作结束后设置数组
    final void setArray(Object[] a) {
        array = a;
    }
```
### 适用场景：使用读多写少的应用场景
**缺陷：**
- 内存占用：写操作的时候需要复制一个新的数组。
- 数据不一致：读操作不能读取实时数据。
**不适用**内存敏感及实时性要求很高的场景。
## LinkedList
### 1.概览
基于双向链表实现
```java
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

每个链表存储了first和last指针
```java
    transient Node<E> first;
    transient Node<E> last;
```
![2018-12-14.21.08.35-LinkedList.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-14.21.08.35-LinkedList.png)
### 2.与ArrayList对比
- ArrayList基于动态数组实现，访问更快。LinkedList基于双向链表实现，插入删除更快。
- ArrayList支持动态访问，LinkedList不支持。
## HashMap
### 1.HashMap的数据结构
HashMap在实现上使用了数组+链表+红黑树三个数据结构（当链表的长度大于8的时候转化为红黑树），存储结构如下图所示
![2018-12-14.21.12.07-HashMap数据结构.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2018-12-14.21.12.07-HashMap%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png)
HashMap是通过计算key的hashCode来存储位置，但是存在哈希冲突的问题。为解决该问题，HashMap通过链地址的方式将hashCode相同的记录放在同一链表上。链表长度大于8的时候则会链表转换为红黑树。节点数据结构如图所示。
```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;	// 哈希值，用于确定记录位置
        final K key;	// 记录key
        V value;		// 记录value
        Node<K,V> next;	// 指向下一个节点

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
存储结构之一数组
```java
transient Node<K,V>[] table;
```

下面是几个比较重要的成员变量
```java
// 记录Map的大小
transient int size;
// 扩容阈值 
int threshold;
// 负载因子，默认为0.75
final float loadFactor;
// 初始是16，大小必须为2的倍数
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
// 最大大小
static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
通过下列方法返回最接近给定值的2的n次方。
```java
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




