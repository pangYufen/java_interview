# java集合框架

容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对（两个对象）的映射表。

![img](https://i0.hdslb.com/bfs/article/8b9c752cfcb420c39be87b16f7a886b99531f557.png@1312w_852h.webp)

## 一 Collection

![img](https://cyc2018.github.io/CS-Notes/pics/6_2001550476096035.png)

### [1. Set](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%AE%B9%E5%99%A8?id=_1-set)

- TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。
- HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。
- LinkedHashSet：具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序。

### [2. List](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%AE%B9%E5%99%A8?id=_2-list)

- ArrayList：基于动态数组实现，支持随机访问。
- Vector：和 ArrayList 类似，但它是线程安全的。
- LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

### [3. Queue](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%AE%B9%E5%99%A8?id=_3-queue)

- LinkedList：可以用它来实现双向队列。
- PriorityQueue：基于堆结构实现，可以用它来实现优先队列。

## 二 Map

![img](https://cyc2018.github.io/CS-Notes/pics/2_2001550426232419.png)

- TreeMap：基于红黑树实现。
- HashMap：基于哈希表实现。
- HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。
- LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。

## 三  ArrayList源码分析

![img](http://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQx9c3H2ThLmjteAsRafPdY6mG4UFW69INULFogAGFQnsX70Drt6AvF7ybn0osc46MMTPvAaur9JnQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](http://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQx9c3H2ThLmjteAsRafPdY6XTtTLfv7mtVLYPib5hQHdAmFJdPib2ibtTctxAHKjibFmsOKAsxXpcoeTw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 1. 添加元素

​	构造ArrayList的时候，默认的底层数组大小是10。底层数组的大小不够了怎么办？答案就是扩容，这也就是为什么一直说ArrayList的底层是基于动态数组实现的原因，动态数组的意思就是指底层的数组大小并不是固定的，而是根据添加的元素大小进行一个判断，不够的话就动态扩容

~~~java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
~~~

添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，也就是旧容量的 1.5 倍。

扩容操作需要调用 `Arrays.copyOf()` 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

![img](http://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQx9c3H2ThLmjteAsRafPdY6Aiat81PJmoIYZlbPoCCvTmxibR3b4jZOiahtbHDdIRbvT7iaPaAo7kEgKg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**插入元素:在指定位置插入元素**

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```

看到插入的时候，按照指定位置，把从指定位置开始的所有元素利用System,arraycopy方法做一个整体的复制，向后移动一个位置（当然先要用ensureCapacity方法进行判断，加了一个元素之后数组会不会不够大），然后指定位置的元素设置为需要插入的元素，完成了一次插入的操作。用图表示这个过程是这样的：

![img](http://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQx9c3H2ThLmjteAsRafPdY6qx75kgpLIjseQWt6VRNZ9F815YqIHHDOpjZLW9pUbhur6ibvjbrpkIQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.删除元素

~~~java
public E remove(int index) {
	//检查给定的index是否在范围之内
    rangeCheck(index);
    modCount++;
    //获得要删除的元素
    E oldValue = elementData(index);
    //计算出要移动的数字的个数
    int numMoved = size - index - 1;
    if (numMoved > 0)
    	//从elementData数组的index后的数字每个都向前移一位
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
~~~

需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(N)，可以看出 ArrayList 删除元素的代价是非常高的。最后一个位置的元素指定为null，这样让gc可以去回收它。

![img](http://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQx9c3H2ThLmjteAsRafPdY6bxGesvicWJF2iawoZXabCUosp1Yqw1MibMuoicsAu4QNTKPGwbTWWyu8Dg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 3.Fail-Fast

~~~java
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
~~~

modCount 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException。

## 4.序列化

**transient关键字的用法：**

 我们都知道一个对象只要实现了Serilizable接口，这个对象就可以被序列化，，java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

```java
package CollectionTest;

import java.io.*;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class Test01 {
    public static class User implements Serializable{
        private static final long serialVersionUID = 8294180014912103005L;
        String username;
        transient String  password;


        public String getUsername() {
            return username;
        }

        public String getPassword() {
            return password;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public void setPassword(String password) {
            this.password = password;
        }
    }
    public static void main(String[] args) {
    
        User user = new User();
        user.setUsername("zhangsan");
        user.setPassword("123");

        System.out.println("read before Serializable: ");
        System.out.println("username:"+user.getUsername());
        System.out.println("password"+user.getPassword());

        try {
            ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("E:/user.txt"));
            os.writeObject(user); // 将User对象写进文件
            os.flush();
            os.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        try {
            ObjectInputStream is = new ObjectInputStream(new FileInputStream("E:/user.txt"));
            // 从流中读取User的数据
            User user1 = (User) is.readObject();
            is.close();

            System.out.println("\nread after Serializable: ");
            System.out.println("username:"+user1.getUsername());
            System.out.println("password"+user1.getPassword());
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

~~~
read before Serializable: 
username:zhangsan
password123

read after Serializable: 
username:zhangsan
passwordnull
~~~

密码字段为null，说明反序列化时根本没有从文件中获取到信息。

ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。

**保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化。**

~~~java
transient Object[] elementData; // non-private to simplify nested class access
~~~

**ArrayList实现了Serializable接口，这意味着ArrayList是可以被序列化的，用transient修饰elementData意味着我不希望elementData数组被序列化。这是为什么？因为序列化ArrayList的时候，ArrayList里面的elementData未必是满的，比方说elementData有10的大小，但是我只用了其中的3个，那么是否有必要序列化整个elementData呢？显然没有这个必要，因此ArrayList中重写了writeObject方法**

ArrayList 实现了 writeObject() 和 readObject() 来控制只序列化数组中有元素填充那部分内容。每次序列化的时候调用这个方法，先调用defaultWriteObject()方法序列化ArrayList中的非transient元素，elementData不去序列化它，然后遍历elementData，只序列化那些有的元素，这样：

1、加快了序列化的速度

2、减小了序列化之后的文件大小

~~~java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}

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
~~~

序列化时需要使用 ObjectOutputStream 的 writeObject() 将对象转换为字节流并输出。而 writeObject() 方法在传入的对象存在 writeObject() 的时候会去反射调用该对象的 writeObject() 来实现序列化。反序列化使用的是 ObjectInputStream 的 readObject() 方法，原理类似。

~~~
ArrayList list = new ArrayList();
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
oos.writeObject(list);
~~~

**ArrayList的优缺点：**

1、ArrayList底层以数组实现，是一种随机访问模式，再加上它实现了RandomAccess接口，因此查找也就是get的时候非常快

2、ArrayList在顺序添加一个元素的时候非常方便，只是往数组里面添加了一个元素而已

不过ArrayList的缺点也十分明显：

1、删除元素的时候，涉及到一次元素复制，如果要复制的元素很多，那么就会比较耗费性能

2、插入元素的时候，涉及到一次元素复制，如果要复制的元素很多，那么就会比较耗费性能

因此，ArrayList比较适合顺序添加、随机访问的场景。

## 四 Vector源码分析

## 1.同步

它的实现与 ArrayList 类似，但是使用了 synchronized 进行同步。

~~~java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}
~~~

## 2.与ArrayList的比较

- Vector 是同步的，因此开销就比 ArrayList 要大，访问速度更慢。最好使用 ArrayList 而不是 Vector，因为同步操作完全可以由程序员自己来控制；

- Vector 每次扩容请求其大小的 2 倍空间，而 ArrayList 是 1.5 倍。

  Vector可以指定增长因子，如果该增长因子指定了，那么扩容的时候会每次新的数组大小会在原数组的大小基础上加上增长因子；如果不指定增长因子，那么就给原数组大小*2，源代码是这样的:

  ~~~
  int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                   capacityIncrement : oldCapacity);
  ~~~

## 3.替代方案

可以使用 `Collections.synchronizedList();` 得到一个线程安全的 ArrayList。

~~~
List<String> list = new ArrayList<>();
List<String> synList = Collections.synchronizedList(list);
~~~

也可以使用 concurrent 并发包下的 CopyOnWriteArrayList 类。

~~~
List<String> list = new CopyOnWriteArrayList<>();
~~~

### CopyOnWriteArrayList

**读写分离**

写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。

写操作需要加锁，防止并发写入时导致写入数据丢失。

写操作结束之后需要把原始数组指向新的复制数组。

~~~java
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
~~~

**适用场景**

CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

但是 CopyOnWriteArrayList 有其缺陷：

- 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。

所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景

## 四 LinkedList

## 1.概述

基于双向链表实现，***LinkedList*同时实现了*List*接口和*Deque*接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列（*Queue*），同时又可以看作一个栈（*Stack*）。LinkedList**的基本存储单元，它是LinkedList中的一个内部类：

~~~
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
~~~

每个链表存储了 first 和 last 指针：

~~~
transient Node<E> first;
transient Node<E> last;
~~~

![img](https://cyc2018.github.io/CS-Notes/pics/09184175-9bf2-40ff-8a68-3b467c77216a.png)

**四个关注点在LinkedList上的答案**

![img](http://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQypLx0HCkPgu5K8OA2jp9qia3dKd71vQI9DgbbiaZnsjhDgfv1mEZlibiaibmyiaQgXCq8InYlp7G4uLX5g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2 添加元素

*add()*方法有两个版本，一个是add(E e)，该方法在*LinkedList*的末尾插入元素，因为有last指向链表末尾，在末尾插入元素的花费是常数时间。只需要简单修改几个相关引用即可；另一个是add(int index, E element)，该方法是在指定下表处插入元素，需要先通过线性查找找到具体位置，然后修改相关引用完成插入操作。

![img](https://pic2.zhimg.com/80/v2-7971b8bdf34d125e7d58aae574f58091_hd.png)

- add(E e)，一个参数方法，默认添加到list最后面

  ~~~java
   public boolean add(E e) {
          linkLast(e);
          return true;
      }
      void linkLast(E e) {
          final Node<E> l = last;
          final Node<E> newNode = new Node<>(l, e, null);     //新建node节点，pre指向list中的last节点
          last = newNode;
          if (l == null)          //如果list为空，新增节点赋值给first
              first = newNode;
          else
              l.next = newNode;   //如果不为空，list中的last指向新增节点，完成新增节点动作
          size++;
          modCount++;
      }
  ~~~

- add(int index, E element)，把元素element添加到下标index位置

  **先根据index找到要插入的位置；2.修改引用，完成插入操作。**

~~~java
 public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)  //如果index等于size，添加到list最后面
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
    Node<E> node(int index) {
        // assert isElementIndex(index);
		//index在前半部分，从前往后搜
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            //index在后半部分，从后往前搜
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        //找到当前index位置节点的前一个节点
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);  //插入到当前index上的节点和它的pred节点中间
        succ.prev = newNode;
        if (pred == null) //要插入的位置为第一个位置，即双链表的头部
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
~~~

**因为链表双向的，可以从开始往后找，也可以从结尾往前找，具体朝那个方向找取决于条件index < (size >> 1)，也即是index是靠近前端还是后端。**

- addAll(int index, Collection<? extends E> c)

~~~java
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);  //检查下标合法性

    Object[] a = c.toArray();   //把集合转换为数组
    int numNew = a.length;      //添加元素数量
    if (numNew == 0)
        return false;

    Node<E> pred, succ;
    if (index == size) {    //把集合c插入到最后面
        succ = null;
        pred = last;
    } else {
        succ = node(index);     //插入中间
        pred = succ.prev;
    }

    for (Object o : a) {    //循环插入节点
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    if (succ == null) {     //如果插入最后面，则last节点是最后一个插入的节点
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;     //list大小加上增加集合的元素数量
    modCount++;         //修改次数加1
    return true;
}

~~~

- addFirst(E e) `linkFirst插入到最前面`

  ~~~java
   private void linkFirst(E e) {
          final Node<E> f = first;
          final Node<E> newNode = new Node<>(null, e, f);
          first = newNode;
          if (f == null) //链表为空
              last = newNode;
          else
              f.prev = newNode;
          size++;
          modCount++;
      }
  ~~~

- addLast(E e) `和add(E e)一样`

## 3.get获取元素

- get(int index) //获取下标index上的元素

  get(int index)得到指定下标处元素的引用，通过调用上文中提到的node(int index)方法实现。

  ~~~java
   public E get(int index) {
          checkElementIndex(index);
          return node(index).item;    //node(index)获取下标index的node节点
      }
  ~~~

- getFirst() //获取第一个元素（first节点的值）

  ~~~
  public E getFirst() {
          final Node<E> f = first;
          if (f == null)
              throw new NoSuchElementException();
          return f.item;
      }
  ~~~

- getLast() //获取最后一个元素（last节点的值）

  ~~~
  public E getLast() {
          final Node<E> l = last;
          if (l == null)
              throw new NoSuchElementException();
          return l.item;
      }
  ~~~

## 4.删除元素

![img](https://pic4.zhimg.com/80/v2-9e88026a0e0f0185df0c6fedebf0a41b_hd.png)

1.先找到要删除元素的引用，2.修改相关引用，完成删除操作。在寻找被删元素引用的时候remove(Object o)调用的是元素的equals方法，而remove(int index)使用的是下标计数，两种方式都是线性时间复杂度。在步骤2中，两个revome()方法都是通过unlink(Node<E> x)方法完成的。这里需要考虑删除元素是第一个或者最后一个时的边界情况。

~~~java
 public E remove() {
        return removeFirst();
    }
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
 	public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }

    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // 断开删除节点的连接，帮助垃圾回收
        first = next;
        if (next == null) //删除节点的下一个节点为空	
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }

~~~

~~~java
 public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index)); //删除下标index上的节点
    }
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) { //如果是头节点，first指向next节点，即删除当前节点
            first = next;
        } else {
            prev.next = next;   //否则，前一个节点的next指向下一个节点
            x.prev = null;
        }

        if (next == null) { //如果是尾节点，last指向前一个节点，即删除当前节点
            last = prev;
        } else {
            next.prev = prev;   //否则，下一个节点的prev指向前一个节点
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
~~~

- remove(Object o)`删除第一个匹配到的元素o`

  ~~~java
   /**
      * 删除list中最前面的元素o（如果存在），从first节点往后循环查找
      * null的equals方法总是返回false，所以需要分开判断
      */
      public boolean remove(Object o) {
          if (o == null) {
              for (Node<E> x = first; x != null; x = x.next) {    //从first节点往后循环，删除第一个匹配的元素，结束循环
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
  ~~~

## 5.一些队列操作

- **peek方法：**list中的第一个元素，不删除，如果list为空，返回null

~~~
  public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
~~~

- **poll方法：**返回list中的第一个元素，并删除，**如果list为空，返回null**

```
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
```

- peekFirst()，同peek()
- peekLast()，返回最后一个元素，不删除
- pollFirst()，同poll()
- pollLast()，返回最后一个元素，并删除

## 6.一些栈操作

- push(E e)方法，添加元素，**相当于在双链表的头部添加元素**

~~~
public void push(E e) {
        addFirst(e);
}
~~~

- pop()方法，删除list中的第一个元素，**如果list为空，抛出NoSuchElementException异常**

~~~
  public E pop() {
        return removeFirst();
    }
~~~

## 7.其他方法

- clear()方法，删除list中的所有元素

  ~~~java
     public void clear() {
          for (Node<E> x = first; x != null; ) {  //循环list，把item、next、prev赋值为null，帮助垃圾回收
              Node<E> next = x.next;
              x.item = null;
              x.next = null;
              x.prev = null;
              x = next;
          }
          first = last = null;
          size = 0;
          modCount++;
      }
  ~~~

- clone()方法，复制一个相同的list

- contains(Object o)方法，判断list中是否包含元素o

- indexOf(Object o)方法，返回元素o的下标

  ~~~java
  public int indexOf(Object o) {
      //不存在返回-1
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
  ~~~

- set(int index, E element)方法，给下标index上的元素赋值，返回原有的值

  ~~~java
   public E set(int index, E element) {
          checkElementIndex(index);
          Node<E> x = node(index); //找到下标为index的元素
          E oldVal = x.item;
          x.item = element;
          return oldVal; //返回旧值
      }
  ~~~

- toArray()方法，把list中的元素复制到新数组中并返回该数组

- toArray(T[] a)方法，把list中的元素复制到指定的数组中

## 五 HashMap源码分析

之前的List，讲了ArrayList、LinkedList，就前两者而言，反映的是两种思想：

- ArrayList以数组形式实现，顺序插入、查找快，插入、删除较慢
- LinkedList以链表形式实现，顺序插入、查找较慢，插入、删除方便

## 1.定义

~~~
public class HashMap<K,V> extends AbstractMap<K,V>
        implements Map<K,V>, Cloneable, Serializable
~~~

![img](https://user-gold-cdn.xitu.io/2017/4/16/10d03cbdca138d775feee9af3ea1d4cb?imageView2/0/w/1280/h/960/ignore-error/1)

HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。HashMap继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口。HashMap 的实现不是同步的，这意味着它不是线程安全的。它的key、value都可以为null。此外，HashMap中的映射不是有序的。

![img](http://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQwibZwgqHG6eMbLa2ibSsPYwNNWPqqdMo3DpCZdy1bxicLRk5QfdiaH6ae3iaNSxZoVGpqMchWRHyB1pJQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**重要的常量属性：**

~~~java
//默认容量16 必须为2的n次幂
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
//最大容量 2的30次方
static final int MAXIMUM_CAPACITY = 1 << 30;
//默认加载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
//链表转成红黑树的阈值
static final int TREEIFY_THRESHOLD = 8;
//红黑树转为链表的阈值
static final int UNTREEIFY_THRESHOLD = 6;
// Entry数组，长度必须为2的n次幂
    transient Entry[] table;
//存储方式由链表转成红黑树的容量的最小阈值
static final int MIN_TREEIFY_CAPACITY = 64;
//HashMap中存储的键值对的数量
transient int size;
//扩容阈值，当size>=threshold时，就会扩容
int threshold;
//HashMap的加载因子
final float loadFactor;
~~~

**加载因子默认为0.75，当HashMap中存储的元素的数量大于(容量×加载因子)，也就是默认大于16*0.75=12时，HashMap会进行扩容的操作。**

可以看出HashMap底层是用Entry数组存储数据，同时定义了初始容量，最大容量，加载因子等参数，至于为什么容量必须是2的幂，加载因子又是什么，下面再说，先来看一下Entry的定义。

~~~java
static class Entry<K,V> implements Map.Entry<K,V> {
        final K key ; 
        V value;
        Entry<K,V> next; // 指向下一个节点
        final int hash;

        Entry( int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

        public final K getKey() {
            return key ;
        }

        public final V getValue() {
            return value ;
        }

        public final V setValue(V newValue) {
           V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            Object k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                Object v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null && v1.equals(v2)))
                    return true;
            }
            return false;
        }

        public final int hashCode() {
            return (key ==null   ? 0 : key.hashCode()) ^
                   ( value==null ? 0 : value.hashCode());
        }

        public final String toString() {
            return getKey() + "=" + getValue();
        }

        // 当向HashMap中添加元素的时候调用这个方法，这里没有实现是供子类回调用
        void recordAccess(HashMap<K,V> m) {
        }

        // 当从HashMap中删除元素的时候调动这个方法 ，这里没有实现是供子类回调用
        void recordRemoval(HashMap<K,V> m) {
        }
}
~~~

Entry是HashMap的内部类，它继承了Map中的Entry接口，它定义了键(key)，值(value)，和下一个节点的引用(next)，以及hash值。很明确的可以看出Entry是什么结构，它是单线链表的一个节点。**也就是说HashMap的底层结构是一个数组，而数组的元素是一个单向链表。**

![img](https://user-gold-cdn.xitu.io/2017/4/16/a82e44351e087753b8488c151ca37fbc?imageView2/0/w/1280/h/960/ignore-error/1)

为什么会有这样的设计？之前介绍的List中查询时需要遍历所有的数组，为了解决这个问题HashMap采用hash算法将key散列为一个int值，这个int值对应到数组的下标，再做查询操作的时候，拿到key的散列值，根据数组下标就能直接找到存储在数组的元素。但是由于hash可能会出现相同的散列值，为了解决冲突，**HashMap采用将相同的散列值存储到一个链表中，也就是说在一个链表中的元素他们的散列值绝对是相同的**。找到数组下标取出链表，再遍历链表是不是比遍历整个数组效率好的多呢？

### 拉链法的工作原理

~~~
HashMap<String, String> map = new HashMap<>();
map.put("K1", "V1");
map.put("K2", "V2");
map.put("K3", "V3");
~~~

- 新建一个 HashMap，默认大小为 16；
- 插入 <K1,V1> 键值对，先计算 K1 的 hashCode 为 115，使用除留余数法得到所在的桶下标 115%16=3。
- 插入 <K2,V2> 键值对，先计算 K2 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6。
- 插入 <K3,V3> 键值对，先计算 K3 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6，插在 <K2,V2> 前面。

应该注意到链表的插入是以头插法方式进行的，例如上面的 <K3,V3> 不是插在 <K2,V2> 后面，而是插入在链表头部。

查找需要分成两步进行：

- 计算键值对所在的桶；
- 在链表上顺序查找，时间复杂度显然和链表的长度成正比。

![img](https://cyc2018.github.io/CS-Notes/pics/cf779e26-0382-4495-8463-f1e19e2e38a0.jpg)

## 2.HashMap的构造函数

```java
/**
     * 构造一个指定初始容量和加载因子的HashMap
     */
    public HashMap( int initialCapacity, float loadFactor) {
        // 初始容量和加载因子合法校验
        if (initialCapacity < 0)
            throw new IllegalArgumentException( "Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException( "Illegal load factor: " +
                                               loadFactor);

        // Find a power of 2 >= initialCapacity
        // 确保容量为2的n次幂，是capacity为大于initialCapacity的最小的2的n次幂
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;

        // 赋值加载因子
        this.loadFactor = loadFactor;
        // 赋值扩容临界值
        threshold = (int)(capacity * loadFactor);
        // 初始化hash表
        table = new Entry[capacity];
        init();
    }

    /**
     * 构造一个指定初始容量的HashMap
     */
    public HashMap( int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * 构造一个使用默认初始容量(16)和默认加载因子(0.75)的HashMap
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
        table = new Entry[DEFAULT_INITIAL_CAPACITY];
        init();
    }

    /**
     * 构造一个指定map的HashMap，所创建HashMap使用默认加载因子(0.75)和足以容纳指定map的初始容量。
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        // 确保最小初始容量为16，并保证可以容纳指定map
        this(Math.max(( int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY ), DEFAULT_LOAD_FACTOR);
        putAllForCreate(m);
}
```

在这里提到了两个参数：初始容量，加载因子。这两个参数是影响HashMap性能的重要参数，其中容量表示哈希表中桶的数量，初始容量是创建哈希表时的容量，加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度，它衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。对于使用链表法的散列表来说，查找一个元素的平均时间是O(1+a)，因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。系统默认负载因子为0.75，一般情况下我们是无需修改的。

## 3. HashMap的主要方法

#### 3.1 put方法

HashMap会对null值key进行特殊处理，总是放到table[0]位置。put过程是先计算hash然后通过hash。table.length取摸计算index值，然后将key放到table[index]位置，当table[index]已存在其它元素时，会在table[index]位置形成一个链表，将新添加的元素放在table[index]，原来的元素通过Entry的next进行链接，这样以链表形式解决hash冲突问题，当元素数量达到临界值(capactiy*factor)时，则进行扩容，是table数组长度变为table.length*2

~~~java
//如果原来存在相同的key-value，原来的value会被替换掉
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {  
    //如果表没有初始化，则以阈值threshold的容量初始化表
        inflateTable(threshold);
    }
    if (key == null)   
    //如果key值为null，调用putForNullKey方法，所以hashmap可以插入key和value为null的值
        return putForNullKey(value);
    //计算key的hash值
    int hash = hash(key); 
    //计算hash值对应表的位置，即表下标
    int i = indexFor(hash, table.length); 
       //遍历table[i]位置的链表，查找相同的key，若找到则使用新的value替换掉原来的oldValue并返回oldValue
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
        //如果hash值相等并且（key值相等或者key的equals方法相等），
        //则覆盖，返回旧的value
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    //修改字数+1
    modCount++; 
   //若没有在table[i]位置找到相同的key，则添加key到table[i]位置，新的元素总是在table[i]位置的第一个元素，原来的元素后移
    addEntry(hash, key, value, i);  
    return null;
}

 void addEntry(int hash, K key, V value, int bucketIndex) {
    //添加key到table[bucketIndex]位置，新的元素总是在table[bucketIndex]的第一个元素，原来的元素后移
    Entry<K,V> e = table[bucketIndex];
      // 头插法，链表头部指向新的键值对
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    //判断元素个数是否达到了临界值，若已达到临界值则扩容，table长度翻倍
        if (size++ >= threshold)
            resize(2 * table.length);
    }
~~~

**先看hash(key)方法：**

~~~java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
~~~

我们可以发现，当key=null时，也是有hash值的，是0，所以，HashMap的key是可以为null的，对比HashTable源码我们可以知道，HashTable的key直接进行了hashCode，如果key为null时，会抛出异常，所以HashTable的key不可以是null。

我们还能发现hash值的计算，首先计算出key的hashCode()为h，然后与h无条件右移16位后的二进制进行按位异或(^)得到最终的hash值，这个hash值就是键值对存储在数组中的位置。

**接着看indexFor()方法:确定桶的下标**

~~~java
static int indexFor(int h, int length) {
    // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
    return h & (length-1);
}
~~~

根据hash值确定在table中的位置，length为2的倍数 HashMap的扩容是基于2的倍数来扩容的，从这里可以看出，对于indexFor方法而言，其具体实现就是通过一个计算出来的code值和数组长度-1做位运算，那么对于2^N来说，长度减一转换成二进制之后就是全一（长度16，len-1=15,二进制就是1111），所以这种设定的好处就是说，对于计算出来的code值得每一位都会影响到我们索引位置的确定，其目的就是为了能让数据更好的散列到不同的桶中。

HashMap 允许插入键为 null 的键值对。但是因为无法调用 null 的 hashCode() 方法，也就无法确定该键值对的桶下标，只能通过强制指定一个桶下标来存放。HashMap 使用第 0 个桶存放键为 null 的键值对。

~~~java
 private V putForNullKey(V value) {
        // 取出数组第1个位置（下标等于0）的节点，如果存在则覆盖不存在则新增，和上面的put一样不多讲，
        for (Entry<K,V> e = table [0]; e != null; e = e. next) {
            if (e.key == null) {
                V oldValue = e. value;
                e. value = value;
                e.recordAccess( this);
                return oldValue;
            }
        }
        modCount++;
        // 如果key等于null，则hash值等于0
        addEntry(0, null, value, 0);
        return null;
    }
~~~

#### 3.2扩容基本原理

设 HashMap 的 table 长度为 M，需要存储的键值对数量为 N，如果哈希函数满足均匀性的要求，那么每条链表的长度大约为 N/M，因此平均查找次数的复杂度为 O(N/M)。

为了让查找的成本降低，应该尽可能使得 N/M 尽可能小，因此需要保证 M 尽可能大，也就是说 table 要尽可能大。HashMap 采用动态扩容来根据当前的 N 值来调整 M 值，使得空间效率和时间效率都能得到保证。

和扩容相关的参数主要有：capacity、size、threshold 和 load_factor

| 参数       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| capacity   | table 的容量大小，默认为 16。需要注意的是 capacity 必须保证为 2 的 n 次方。 |
| size       | 键值对数量。                                                 |
| threshold  | size 的临界值，当 size 大于等于 threshold 就必须进行扩容操作。 |
| loadFactor | 装载因子，table 能够使用的比例，threshold = capacity * loadFactor。 |

**resize方法**

resize方法在hashmap中并没有公开，这个方法实现了非常重要的hashmap扩容，具体过程为：先创建一个容量为table.length*2的新table，修改临界值，然后把table里面元素计算hash值并使用hash与table.length*2重新计算index放入到新的table里面.这里需要注意下是用**每个元素的hash全部重新计算index**，而不是简单的把原table对应index位置元素简单的移动到新table对应位置

~~~java
void resize( int newCapacity) {
        // 当前数组
        Entry[] oldTable = table;
        // 当前数组容量
        int oldCapacity = oldTable.length ;
        // 如果当前数组已经是默认最大容量MAXIMUM_CAPACITY ，则将临界值改为Integer.MAX_VALUE 返回
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        // 使用新的容量创建一个新的链表数组
        Entry[] newTable = new Entry[newCapacity];
        // 将当前数组中的元素都移动到新数组中
        transfer(newTable);
        // 将当前数组指向新创建的数组
        table = newTable;
        // 重新计算临界值
        threshold = (int)(newCapacity * loadFactor);
    }

    /**
     * Transfers all entries from current table to newTable.
     */
    void transfer(Entry[] newTable) {
        // 当前数组
        Entry[] src = table;
        // 新数组长度
        int newCapacity = newTable.length ;
        // 遍历当前数组的元素，重新计算每个元素所在数组位置
        for (int j = 0; j < src. length; j++) {
            // 取出数组中的链表第一个节点
            Entry<K,V> e = src[j];
            if (e != null) {
                // 将旧链表位置置空
                src[j] = null;
                // 循环链表，挨个将每个节点插入到新的数组位置中
                do {
                    // 取出链表中的当前节点的下一个节点
                    Entry<K,V> next = e. next;
                    // 重新计算该链表在数组中的索引位置
                    int i = indexFor(e. hash, newCapacity);
                    // 将下一个节点指向newTable[i]
                    e. next = newTable[i];
                    // 将当前节点放置在newTable[i]位置
                    newTable[i] = e;
                    // 下一次循环
                    e = next;
                } while (e != null);
            }
        }
}
~~~

transfer方法中，由于数组的容量已经变大，也就导致hash算法indexFor已经发生变化，原先在一个链表中的元素，在新的hash下可能会产生不同的散列值，so所有元素都要重新计算后安顿一番。注意在do while循环的过程中，每次循环都是将下个节点指向newTable[i] ，是因为如果有相同的散列值i，上个节点已经放置在newTable[i]位置，这里还是下一个节点的next指向上一个节点.

Map中的元素越多，hash冲突的几率也就越大，数组长度是固定的，所以导致链表越来越长，那么查询的效率当然也就越低下了。还记不记得同时数组容器的ArrayList怎么做的，扩容！而HashMap的扩容resize，需要将所有的元素重新计算后，一个个重新排列到新的数组中去，这是非常低效的，和ArrayList一样，在可以预知容量大小的情况下，提前预设容量会减少HashMap的扩容，提高性能。

再来看看加载因子的作用，如果加载因子越大，数组填充的越满，这样可以有效的利用空间，但是有一个弊端就是可能会导致冲突的加大，链表过长，反过来却又会造成内存空间的浪费。所以只能需要在空间和时间中找一个平衡点，那就是设置有效的加载因子。我们知道，很多时候为了提高查询效率的做法都是牺牲空间换取时间，到底该怎么取舍，那就要具体分析了。

#### 3.3 get方法

同样当key为null时会进行特殊处理，在table[0]的链表上查找key为null的元素。get的过程是先计算hash然后通过hash与table.length取模计算index值，然后遍历table[index]上的链表，直到找到key，然后返回

~~~java
public V get(Object key) {
        if (key == null)
            return getForNullKey();//处理null值
        int hash = hash(key.hashCode());//计算hash
    //在table[index]遍历查找key，若找到则返回value，找不到返回null
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
                return e.value;
        }
        return null;
}

private V getForNullKey() {
    if (size == 0) {
        return null;
    }
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null)
            return e.value;
    }
    return null;
}

~~~

- getEntry方法比较简单，先找hash值在表中的位置，再循环链表查找Entry，如果存在，返回Entry，否则返回null

~~~java
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
~~~

#### 3.4 remove方法

