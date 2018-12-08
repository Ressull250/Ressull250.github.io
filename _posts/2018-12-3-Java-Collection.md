---
layout:     post
title:      Java-Collection
subtitle:   
date:       2018-12-3
author:     LiaoBo
header-img: img/header_02.jpg
catalog: true
tags:
    - Java
---
## Collection

为了规范容器的行为，统一设计，JCF定义了14种容器接口（collection interfaces），它们的关系如下图所示：
![pic]({{site.url}}/postimgs/JCF_Collection_Interfaces.png)
*Map*接口没有继承自*Collection*接口，因为*Map*表示的是关联式容器而不是集合。但Java为我们提供了从*Map*转换到*Collection*的方法，可以方便的将*Map*切换到集合视图。
上图中提供了*Queue*接口，却没有*Stack*，这是因为*Stack*的功能已被JDK 1.6引入的*Deque*取代。

```java
java.util.Collection<E> 1.2
Iterator<E> iterator()                      //返回一个用于访问集合中每一个元素的迭代器
int size()                      //返回当前存储在集合中的元素的个数
boolean isEmpty()                 //如果集合中没有元素，返回true
boolean contains(Object obj)                            //如果集合中包含了一个与obj相等的对象，返回true
boolean containsAll(Collection<?> other)                     //如果集合中包含了other集合中的所有元素，返回true
boolean add(Object element)                      //添加一个元素
boolean addAll(Collection<? extends E> other)                //添加other集合中的所有元素
boolean remove(Object obj)                    //删除与obj相等的元素
boolean removeAll(Collection<?> other)                  //删除other集合中的所有元素相等的元素
void clear()                      //从集合中删除所有元素
boolean retainAll(Collection<?> other)                         //删除与other集合中元素不同的元素
Object[] toArray()                       //返回这个集合的对象数组
<T> T[] toArray(T[] arrayToFill)                                //返回这个集合的对象数组，如果arrayToFill足够大则放入arrayToFill，否则新建一个数组存放
```



## List

```java
java.util.List<E> 1.2
ListIterator<E> listIterator()                      //返回一个列表迭代器
ListIterator<E> listIterator(int index)                    //返回一个指定位置的列表迭代器
void add(int i, E element)                      //在给定位置添加一个元素
void addAll(int i, Collection<? extends E> elements)                            //将集合中的所有元素添加到指定位置
E remove(int i)                   //删除指定位置的元素并返回这个元素
E get(int i)                         //获取指定位置的元素
E set(int i, E element)                           //用新元素取代指定位置的元素，并返回旧元素
int indexOf(Object element)                      //返回与指定元素相等的元素第一次出现的位置，没有则返回-1
int lastIndexOf(Object element)                          //返回与指定元素相等的元素最后一次出现的位置
```



### ArrayList

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

- *ArrayList*实现了*List*接口，是顺序容器，即元素存放的数据与放进去的顺序相同，**允许放入`null`元素**，

- **基于数组实现，默认大小为10**`private static final int DEFAULT_CAPACITY = 10;`

- 添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，也就是旧容量的 1.5 倍。

- get(),set()常数时间，add与插入位置有关，copy数组代价较大

#### set()

既然底层是一个数组*ArrayList*的`set()`方法也就变得非常简单，直接对数组的指定位置赋值即可。

```java
public E set(int index, E element) {
    rangeCheck(index);//下标越界检查
    E oldValue = elementData(index);
    elementData[index] = element;//赋值到指定位置，复制的仅仅是引用
    return oldValue;
}
```

#### get()

`get()`方法同样很简单，唯一要注意的是由于底层数组是Object[]，得到元素后需要进行类型转换。

```java
public E get(int index) {
    rangeCheck(index);
    return (E) elementData[index];//注意类型转换
}
```

#### add()

跟C++ 的*vector*不同，*ArrayList*没有`push_back()`方法，对应的方法是`add(E e)`，*ArrayList*也没有`insert()`方法，对应的方法是`add(int index, E e)`。这两个方法都是向容器中添加新元素，这可能会导致*capacity*不足，因此在添加元素之前，都需要进行剩余空间检查，如果需要则自动扩容。扩容操作最终是通过`grow()`方法完成的。

```java
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);//原来的1.5倍
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);//扩展空间并复制
}
```

由于Java GC自动管理了内存，这里也就不需要考虑源数组释放的问题。

空间的问题解决后，插入过程就显得非常简单。

`add(int index, E e)`需要先对元素进行移动，然后完成插入操作，也就意味着该方法有着线性的时间复杂度。

#### addAll()

`addAll()`方法能够一次添加多个元素，根据位置不同也有两个版本，一个是在末尾添加的`addAll(Collection<? extends E> c)`方法，一个是从指定位置开始插入的`addAll(int index, Collection<? extends E> c)`方法。跟`add()`方法类似，在插入之前也需要进行空间检查，如果需要则自动扩容；如果从指定位置插入，也会存在移动元素的情况。
`addAll()`的时间复杂度不仅跟插入元素的多少有关，也跟插入的位置相关。

#### remove()

`remove()`方法也有两个版本，一个是`remove(int index)`删除指定位置的元素，另一个是`remove(Object o)`删除第一个满足`o.equals(elementData[index])`的元素。删除操作是`add()`操作的逆过程，需要将删除点之后的元素向前移动一个位置。需要注意的是为了让GC起作用，必须显式的为最后一个位置赋`null`值。

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; //清除该位置的引用，让GC起作用
    return oldValue;
}
```

关于Java GC这里需要特别说明一下，**有了垃圾收集器并不意味着一定不会有内存泄漏**。对象能否被GC的依据是是否还有引用指向它，上面代码中如果不手动赋`null`值，除非对应的位置被其他元素覆盖，否则原来的对象就一直不会被回收。



### LinkedList

- **基于双向链表实现，使用 Node 存储链表节点信息**
- **与ArrayList相比，linkedlist不支持随机访问，但在任意位置添加和删除元素更快**
- *LinkedList*的get和set方法都要从列表头部重新搜索，**效率极低**
- *LinkedList*同时实现了*List*接口和*Deque*接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列（*Queue*），同时又可以看作一个栈（*Stack*）。这样看来，*LinkedList*简直就是个全能冠军。当你需要使用栈或者队列时，可以考虑使用*LinkedList*，一方面是因为Java官方已经声明不建议使用*Stack*类，更遗憾的是，Java里根本没有一个叫做*Queue*的类（它是个接口名字）。关于栈或队列，现在的首选是*ArrayDeque*，它有着比*LinkedList*（当作栈或队列使用时）有着更好的性能。

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```



### Vector(了解)

- 操作与实现均与ArrayList类似，**使用了synchronized同步**

- 每次扩容操作请求其大小两倍的空间

- 可以使用 `Collections.synchronizedList();` 得到一个线程安全的 ArrayList。

- ```java
  List<String> list = new ArrayList<>();
  List<String> synList = Collections.synchronizedList(list);
  ```

- 也可以使用 concurrent 并发包下的 CopyOnWriteArrayList 类。

- ```java
  List<String> list = new CopyOnWriteArrayList<>();
  ```



### CopyOnWriteArrayList

> As name suggest CopyOnWriteArrayList creates copy of underlying ArrayList with every mutation operation e.g. add or set. Normally CopyOnWriteArrayList is very expensive because it involves costly Array copy with every write operation but **its very efficient if you have a List where Iteration outnumber mutation** e.g. you mostly need to iterate the ArrayList and don't modify it too often.
>
> **Iterator of CopyOnWriteArrayList is fail-safe and doesn't throw ConcurrentModificationException** even if underlying CopyOnWriteArrayList is modified once Iteration begins because Iterator is operating on separate copy of ArrayList. Consequently all the updates made on CopyOnWriteArrayList is not available to Iterator.

- 写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。

- 写操作需要加锁，防止并发写入时导致写入数据丢失。

- 写操作结束之后需要把原始数组指向新的复制数组。

```java
public boolean add(E e) {
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

final void setArray(Object[] a) {
    array = a;
}
@SuppressWarnings("unchecked")
private E get(Object[] a, int index) {
    return (E) a[index];
}
```

#### 适用场景

CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

但是 CopyOnWriteArrayList 有其缺陷：

- 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。

所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。



## Queue

```java
public interface Queue<E> extends Collection<E> {

    boolean add(E e);

    boolean offer(E e);

    E remove();

    E poll();

    E element();

    E peek();
}
//一组抛异常，一组返回false OR null
```

- Deque（双端队列）接口继承了Queue接口，有**ArrayQueue**和**LinkedList**实现，即可做队列，也可做栈
- PriorityQueue 优先队列，基于堆实现，增删改查操作都是log(n)的复杂度



### ArrayQueue

- ArrayQueue基于数组利用**头尾指针**来实现

- ```java
  public class ArrayDeque<E> extends AbstractCollection<E>
                             implements Deque<E>, Cloneable, Serializable
  {
      transient Object[] elements; // non-private to simplify nested class access
  
      transient int head;
  
      transient int tail;
  
      /**
       * The minimum capacity that we'll use for a newly created deque.
       * Must be a power of 2.
       */
      private static final int MIN_INITIAL_CAPACITY = 8;
  }
  ```

- 每次扩容数组扩大两倍，扩容需复制数组，代价较高，在用于栈和队列时性能要优于LinkedList

- 通过确定初始容量大小避免数组复制



### PriorityQueue

- 在数组上用堆思想实现，不允许null元素
- **优先队列的作用是能保证每次取出的元素都是队列中值最小的**（Java的优先队列每次取最小元素，C\++的优先队列每次取最大元素）。**元素大小的评判可以通过元素本身的自然顺序**（*implents Comparable*），**也可以通过构造时传入的比较器**（*Comparator*）。
- 构造时传入**Comparator.reverseOrder()**即为最大堆
- 元素的增删改查即是堆的增删改查操作



## Map

### TreeMap

```java
  public class TreeMap<K,V>
      extends AbstractMap<K,V>
      implements NavigableMap<K,V>, Cloneable, java.io.Serializable
  {
      private final Comparator<? super K> comparator;
  
      private transient Entry<K,V> root;
  
      private transient int size = 0;
      
      private transient int modCount = 0;
  }
```

- Notes:
  - Key必须实现Comparable接口，或者在构造时传入一个Comparator
  - 基于红黑树实现，增删改查均为log(n)的复杂度





### HashMap

*HashMap*实现了*Map*接口，**即允许放入`key`为`null`的元素，也允许插入`value`为`null`的元素**；除该类未实现同步外，其余跟`Hashtable`大致相同；跟*TreeMap*不同，该容器不保证元素顺序，根据需要该容器可能会对元素重新哈希，元素的顺序也会被重新打散，因此不同时间迭代同一个*HashMap*的顺序可能会不同。
根据对冲突的处理方式不同，哈希表有两种实现方式，一种开放地址方式（Open addressing），另一种是冲突链表方式（Separate chaining with linked lists）。**Java *HashMap*采用的是冲突链表方式**。



HashMap 允许插入键为 null 的键值对。但是因为无法调用 null 的 hashCode() 方法，也就无法确定该键值对的桶下标，只能通过强制指定一个桶下标来存放。HashMap 使用第 0 个桶存放键为 null 的键值对。

​```java
private V putForNullKey(V value) {
​    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
​        if (e.key == null) {
​            V oldValue = e.value;
​            e.value = value;
​            e.recordAccess(this);
​            return oldValue;
​        }
​    }
​    modCount++;
​    addEntry(0, null, value, 0);
​    return null;
}
```

使用链表的头插法，也就是新的键值对插在链表的头部，而不是链表的尾部。

​```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}

void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    // 头插法，链表头部指向新的键值对
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```



### ConcurrentHashMap



### WeakHashMap

http://www.importnew.com/23182.html

https://blog.csdn.net/kaka0509/article/details/73459419



## Set

- **HashSet**和**TreeSet**都用了**适配者模式**，里面包含一个HashMap和SortedMap

- ```java
  public class HashSet<E>
      extends AbstractSet<E>
      implements Set<E>, Cloneable, java.io.Serializable
  {
      private transient HashMap<E,Object> map;
  
      // Dummy value to associate with an Object in the backing Map
      private static final Object PRESENT = new Object();
  
      /**
       * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
       * default initial capacity (16) and load factor (0.75).
       */
      public HashSet() {
          map = new HashMap<>();
      }
  }
  ```

- 对set的增删改查操作都是基于map的接口实现的，所有如果要用set的话不如直接用map少一层接口调用，除了代码可读性要变差一点