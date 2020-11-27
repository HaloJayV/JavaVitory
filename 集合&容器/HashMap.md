[TOC]



# jdk7 HashMap

**数组**
数组存储区间是连续的，占用内存严重，故空间复杂的很大。但数组的二分查找时间复杂度小，为O(1)；
数组的特点是：寻址容易，插入和删除困难；

* HashMap 里面实现一个静态内部类 Entry，Entry 包含四个属性：key，value，hash值和用于单向链表的 next。Entry 就是 HashMap 键值对实现的一个基础 bean，我们上面说到 HashMap的基础就是一个线性数组，这个数组就是 Entry[]，Map 里面的内容都保存在 Entry[] 里面

**链表**
链表存储区间离散，占用内存比较宽松，故空间复杂度很小，但时间复杂度很大，为O(N)。
链表的特点是：寻址困难，插入和删除容易。

* ```
  static int indexFor(int h, int length) {  
     return h & (length-1);   // 比 hash(key)%(length -1) 更高效的方法
  }  
  ```

* 在1.7是这样的头插法: 当put时要插入的元素的索引有重复时，会发生hash冲突碰撞，此时链表发挥作用：新插入的Entry的next指针指向旧的Entry，并通过修改当前索引位置的引用地址指向新的Entry，链表向下移动从而取代旧Entry的位置，放在数组相应索引位置


**哈希表**
那么我们能不能综合两者的特性，做出一种寻址容易，插入删除也容易的数据结构？答案是肯定的，这就是我们要提起的哈希表。哈希表（(Hash table）既满足了数据的查找方便，同时不占用太多的内容空间，使用也十分方便。

* 每个数据对象的hash对应唯一一个值，但是一个hash值不一定对应唯一的数据对象。如果两个不同对象的 hash**哈希冲突**。

哈希表是由`数组+链表`组成的，一个长度为16的数组中，每个数组中元素存储的是一个链表的头结点。

<img src="../../../../Software/Typora/Picture/image-20200707133532781.png" alt="image-20200707133532781"  />

* 源码解析

![image-20200706213120193](../../../../Software/Typora/Picture/image-20200706213120193.png)

```
public HashMap() {
 		// 另外3个构造方法都会调用带有2个参数的构造方法：HashMap(int initialCapacity, float loadFactor)
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR); 
}
```

```java
// 另外3个构造方法都会调用该方法
public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        // Find a power of 2 >= initialCapacity
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;
        this.loadFactor = loadFactor;
        threshold = (int)Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        table = new Entry[capacity];
        useAltHashing = sun.misc.VM.isBooted() &&
                (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
        init();
    }
```

* ### put源码

```java
// 如果put的key在map中已经存在则旧value会被覆盖，然后返回被覆盖的value
public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        // 使用头插法的原因是：插入结点的速度比尾插法更快
        // 遍历当前数组索引 i 上的链表从而寻找到该链表的尾结点
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            // 如果插入当前链表的结点它的hashCode和key都相等
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;  // 先取出oldValue，最后要返回该值
                e.value = value;		// 覆盖oldValue
                e.recordAccess(this);  // hashMap用不到，LinkedList采用得到该方法
                return oldValue;  // 返回被覆盖的值
            }
        }
        modCount++; // 表示修改次数
        addEntry(hash, key, value, i); // 数组扩容
        return null;
    }
```

#### put头插法的问题

数据插入使用头插法，会造成一个问题：

在resize扩容时，里面有一个resize方法又调用了transfer方法，把里面一些Entry进行了rehash，在这个过程中可能造成一个链表的循环，可能导致在下一次Get时候出现要给死循环； 也有可能因为没有加锁，在多个线程并发情况下，不能保证数据安全

* 数组table容量初始化（inflateTable）     

  在第一个元素插入 HashMap 的时候做一次数组的初始化，就是先确定初始的数组大小，并计算数组扩容的阈值。

```java
private void inflateTable(int toSize) {
    // 保证数组大小一定是 2 的 n 次方。
    // 比如这样初始化：new HashMap(20)，那么处理成初始数组大小是 32
    int capacity = roundUpToPowerOf2(toSize);
    // 计算扩容阈值：capacity * loadFactor
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    // 算是初始化数组吧
    table = new Entry[capacity];
    initHashSeedAsNeeded(capacity); //ignore
}
```

## 计算具体数组位置（indexFor）

* 取 hash 值的低 n 位。如在数组长度为 32 的时候，其实取的就是 key 的 hash 值的低 5 位，作为它在数组中的下标位置。

```
static int indexFor(int hash, int length) {
    // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
    return hash & (length-1);
}
```

## 添加节点到链表中（addEntry）

* 找到数组下标后，会先进行 key 判重，如果没有重复，就准备将新值放入到链表的表头。
* 主要逻辑就是先判断是否需要扩容，需要的话先扩容，然后再将这个新的数据插入到扩容后的数组的相应位置处的链表的表头。

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    // 如果当前 HashMap 大小已经达到了阈值，并且新值要插入的数组位置已经有元素了，那么要扩容
    if ((size >= threshold) && (null != table[bucketIndex])) {
        // 扩容，后面会介绍一下
        resize(2 * table.length);
        // 扩容以后，重新计算 hash 值
        hash = (null != key) ? hash(key) : 0;
        // 重新计算扩容后的新的下标
        bucketIndex = indexFor(hash, table.length);
    }
    // 往下看
    createEntry(hash, key, value, bucketIndex);
}
// 这个很简单，其实就是将新值放到链表的表头，然后 size++
void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```

## 数组扩容（resize）

* 插入新值的时候，如果当前的 size 已经达到了阈值，并且要插入的数组位置上已经有元素，那么就会触发扩容，扩容后，数组大小为原来的 2 倍。

* ##### 扩容就是用一个新的大数组替换原来的小数组，并将原来数组中的值迁移到新的数组中。

* ##### 数组扩容是为了分散链表，让链表变短而使得get效率提高，加快查询效率

* 由于是**双倍扩容**，迁移过程中，会将原来 table[i] 中的链表的所有节点，分拆到新的数组的 `newTable[i]`和 `newTable[i + oldLength]` 位置上。如原来数组长度是 16，那么扩容后，原来 table[0] 处的链表中的所有元素会被分配到新数组中 `newTable[0]` 和 `newTable[16]` 这两个位置。

* 毛病：在多线程扩容时，不论是否有链表，并发都有可能会出现**循环链表**的情况，导致在get时会循环遍历。起因是扩容时使用头插法，导致next在结点e 之前，使得数据顺序发生变化
  
  * 原因：当两个线程并发put进一个相同的entity，此时就会形成一个链表环
  
  图解：https://blog.csdn.net/qq_35688140/article/details/100772864
  
  ```
  扩容时使用transfer
  transfer不同于put时的判断hash冲突，而是直接使用头插法，如果没有冲突，则next为null。
  e.next = newTable[i];
  newTable[i] = e;
  如果线程B设置了newTable[i] = e，线程A执行e.next = newTable[i]；
  则会造成e.next = e；不论是否有链表，都会造成死循环。
  ```
  
  * 解决方法：防止hash扩容，具体可以控制阈值threshold大于所需要的数组。还可以加锁、并发安全控制。

```java
	void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }
        createEntry(hash, key, value, bucketIndex);
    }
```

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
    // 新的数组
    Entry[] newTable = new Entry[newCapacity];
    // 将原来数组中的值迁移到新的更大的数组中  transfer()底层是通过双重循环数组直到非空元素，然后转移到新的数组相对应的索引位置
    transfer(newTable, initHashSeedAsNeeded(newCapacity)); // 元素转移
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```

## get 过程分析

- 根据 key 计算 hash 值。
- 找到相应的数组下标：`hash & (length - 1)`。
- 遍历该数组位置处的链表，直到找到相等(==或equals)的 key。

```
public V get(Object key) {
    // 之前说过，key 为 null 的话，会被放到 table[0]，所以只要遍历下 table[0] 处的链表就可以了
    if (key == null)
        return getForNullKey();
    // 
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}
```

```
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    int hash = (key == null) ? 0 : hash(key);
    // 确定数组下标，然后从头开始遍历链表，直到找到为止
    for (Entry<K,V> e = table[indexFor(hash, table.length)];   // 定位数组下标，遍历链表
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;  // 通过比较key和hash是否相等，相等则表示找到，返回entry
    }
    return null;
}
```

#### remove

```
final Entry<K,V> removeEntryForKey(Object key) {
        int hash = (key == null) ? 0 : hash(key);
        int i = indexFor(hash, table.length);
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                modCount++;  
                size--;
                if (prev == e)
                    table[i] = next;
                else
                    prev.next = next;
                e.recordRemoval(this);
                return e;
            }
            prev = e;
            e = next;
        }

        return e;
    }
```





* 1.7的插入存储底层实现：

  * 数组(默认长度为16bit，源码默认容量 = 1 << 4， 最大容量限制：1 << 30)：

    * 数组中还可以存放数组，每个数组中的一个存储单元存储的是 Entry(属性包括：key, value, hash，Node<K,V> next(数组+链表)) 结构的数据，Entry的实现是一个链表Node(属性和Entry相同)。其中元素在数组中的存储索引下标可以通过：(hash % arr.length-1) 得出

    * ```
      put(key, value){  // put底层伪代码
      	int hashcode = key.hashcode();
      	int index = hashcode % table.legnth;
      	table[index] = new Entry(key, value, table[index]); // 头插法，新结点指向数组上对应索引的元素
      }
      ```
      

![image-20200706144230305](../../../../Software/Typora/Picture/image-20200706144230305.png)

# jdk8 HashMap

* 1.7之前put是头插法，1.8之后是尾插法，头插法会出现链表循环的问题，后者不会，但都不是线程安全

* 数组：一段连续的存储单元来存储结构,插入慢O(n)查找快O(1)； 链表Node：一种物理存储单元上非连续、非顺序的存储结构，插入快O(1)查找慢O(n)

* HashMap线程不安全，底层没有锁，线程安全的HashMap：Collections.synchronizedMap()、new ConcurrentHashMap<>();

* HashMap底层是：数组+Node链表+的红黑树，每个数据单元都是一个Node结构。容量为16，负载因子0.75(即装载的内存超过容量的3/4会自动扩容，HashMap扩容为当前的2倍，即2^(4+1)，ArrayList扩容量是原来的一半).

* hashMap的散列表，采用懒加载机制，只有第一次put时才会创建，而不是new就创建

*  Node结构中包含key、value、next、hash字段，其中next字段是当发生hash冲突时，当前桶位中的node与冲突node连成一个链表要用的字段

* hash值是通过key.hashCode()二次加工得到的，加工原则是key.hashCode() ^ (key.hashCode() >>> 16) ， 即高16位和低16位的异或。这样做主要是为了加大散列程度的散列化，也是因为hash寻址算法的缘故：

* HashMap可以存放key为空的元素：map.put(null, "xxx") 当key为null时，JVM会单独分配一片空间用来存放该值，再次put一个key为null的值时会覆盖之前的

* **capacity**：当前数组容量，始终保持 `2^n`，可以扩容，扩容后数组大小为当前的 2 倍。

* **loadFactor**：负载因子，默认为`0.75`。

* **threshold**：扩容的阈值，等于capacity * loadFactor。

* 1、**扩容的数组的长度为什么保持 2^n？**
  其实这是为了保证通过hash方式获取下标的时候分布均匀。数组长度为2的n次幂的时候，不同的key 算得的 index 相同的几率较小，那么数据在数组上分布就比较均匀，也就是说碰撞的几率小，相对的，查询的时候就不用遍历某个位置上的链表，这样查询效率也就较高了。

  2、**为什么负载因子的值默认为 0.75？**
  加载因子是表示Hash表中元素的填满的程度。

  加载因子越大，填满的元素越多，空间利用率越高，但冲突的机会加大了。
  反之,加载因子越小，填满的元素越少，冲突的机会减小，但空间浪费多了。
  冲突的机会越大，则查找的成本越高。反之，查找的成本越小。

  因此,必须在 "冲突的机会"与"空间利用率"之间寻找一种平衡与折衷。

* Java7 HashMap ，查找的时候，根据 hash 值我们能够快速定位到数组的具体下标，但是之后的话，需要顺着链表一个个比较下去才能找到我们需要的，时间复杂度取决于链表的长度，为 O(n)。

**为了降低这部分的开销，在 Java8 中，当链表中的元素达到了 8 个，且散列表数组长度已经达到64，会将链表转换为红黑树，否则进行数组扩容，在这些位置进行查找的时候可以降低时间复杂度为 O(logN)。**

![image-20200707132701633](../../../../Software/Typora/Picture/image-20200707132701633.png)

#### 链表转化为红黑树有两个条件：

putVal时进行判断，链表长度到达8调用treeifyBin 转换红黑树的方法

```java
if (binCount >= TREEIFY_THRESHOLD - 1) // TREEIFY_THRESHOLD 值为8
     treeifyBin(tab, hash);
```

但在转换红黑树前还会进行判断：数组长度是否小于64，小于则散列表resize扩容，大于等于64则链表转红黑树

```java
// 链表转红黑树
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 但在转换红黑树前还会进行判断：数组长度是否小于64
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY) // MIN_TREEIFY_CAPACITY值为64
        resize(); 	// 散列表扩容
    else if ((e = tab[index = (n - 1) & hash]) != null) { // 数组长度大于64，链表转为红黑树
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
```
1.当数组其中一个索引位置上的链表长度大于8时，调用转换红黑树方法treeifyBin

2.当前散列表数组长度已经达到64，两个指标都要达到，否则就算slot内部链表长度达到8也不会链转树，而是发生一次resize散列表扩容



#### 红黑树 RB-Tree

* 二叉查找树是不平衡的，可能会变成链表，因此需要采用红黑树（RB-Tree），插入和查询的 O(N) = log(N)
* 红黑树只需要保证黑色结点高度平衡，且满足以下5个条件
* ![image-20200706145621016](../../../../Software/Typora/Picture/image-20200706145621016.png)



<img src="../../../../Software/Typora/Picture/image-20200706151620960.png" alt="image-20200706151620960"  />

#### 红黑树插入结点规律：

* 跟结点是黑色的，不用进行调整
* 父节点是红色：
  * 1.叔叔的空的，旋转 + 变色(祖父结点和父节点变色)
  * 2.叔叔的红色，父节点+叔叔节点变黑色，祖父结点变红色
  * 3.叔叔是黑色，旋转+变色

#### 红黑树主要特性

​		  (1）每个节点要么是黑色，要么是红色。（节点非黑即红）　　
　　（2）根节点是黑色。　　
　　（3）每个叶子节点（NIL）是黑色。 　　
　　（4）如果一个节点是红色的，则它的子节点必须是黑色的。（也就是说父子节点不能同时为红色）　　

​	    （5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。（这一点是平衡的关键）

### 红黑树的插入

```java
static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                                    TreeNode<K,V> x) {
            x.red = true;
            for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
                if ((xp = x.parent) == null) { // 要插入的结点无父节点，因此设置为黑色
                    x.red = false; // 新结点设置为黑色
                    return x;
                }
                else if (!xp.red || (xpp = xp.parent) == null)  // 情况0：父节点是黑色的，或者无祖父结点，直接返回根节点
                    return root;
                if (xp == (xppl = xpp.left)) {   // 如果父节点是祖父结点的左结点
                    if ((xppr = xpp.right) != null && xppr.red) { // 情况2：叔叔结点不为空且是红色
                        xppr.red = false;  // 叔叔结点变黑
                        xp.red = false;  // 父节点变黑
                        xpp.red = true;  // 祖父节点变红
                        x = xpp; // 递归到祖父结点，直到满足红黑树条件
                    }
                    else {  // 情况1或3：没有叔叔结点或者叔叔结点是黑色的
                        if (x == xp.right) {  // 新结点作为右节点时
                            root = rotateLeft(root, x = xp);  // 先左旋后，新节点变为父节点xp 
                            xpp = (xp = x.parent) == null ? null : xp.parent; // 
                        }
                        if (xp != null) {  // 新结点作为左节点时
                            xp.red = false; // 父节点变为黑色
                            if (xpp != null) {
                                xpp.red = true;
                                root = rotateRight(root, xpp);
                            }
                        }
                    }
                }
                else {
                    if (xppl != null && xppl.red) {
                        xppl.red = false;
                        xp.red = false;
                        xpp.red = true;
                        x = xpp;
                    }
                    else {
                        if (x == xp.left) {
                            root = rotateRight(root, x = xp);
                            xpp = (xp = x.parent) == null ? null : xp.parent;
                        }
                        if (xp != null) {
                            xp.red = false;
                            if (xpp != null) {
                                xpp.red = true;
                                root = rotateLeft(root, xpp);
                            }
                        }
                    }
                }
            }
        }
```

Java7 中使用 Entry 来代表每个 HashMap 中的数据节点，Java8 中使用 Node，基本没有区别，都是 key，value，hash 和 next 这四个属性，不过，Node 只能用于链表的情况，红黑树的情况需要使用 TreeNode。

我们根据数组元素中，第一个节点数据类型是 Node 还是 TreeNode 来判断该位置下是链表还是红黑树的。

* ### Put方法：

和 Java7 稍微有点不一样的地方就是，Java7 是先扩容后插入新值的，Java8 先插值再扩容

```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// 第三个参数 onlyIfAbsent 如果是 true，那么只有在不存在该 key 时才会进行 put 操作
// 第四个参数 evict 我们这里不关心
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 第一次 put 值的时候，会触发下面的 resize()，类似 java7 的第一次 put 也要初始化数组长度
    // 第一次 resize 和后续的扩容有些不一样，因为这次是数组从 null 初始化到默认的 16 或自定义的初始容量
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 找到具体的数组下标，如果此位置没有值，那么直接初始化一下 Node 并放置在这个位置就可以了
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);

    else {// 数组该位置有数据
        Node<K,V> e; K k;
        // 首先，判断该位置的第一个数据和我们要插入的数据，key 是不是"相等"，如果是，取出这个节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果该节点是代表红黑树的节点，调用红黑树的插值方法，本文不展开说红黑树
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 到这里，说明数组该位置上是一个链表
            for (int binCount = 0; ; ++binCount) {
                // 插入到链表的最后面(Java7 是插入到链表的最前面)
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);   // 1.8改用尾插法插入链表的尾部
                    // TREEIFY_THRESHOLD 为 8，所以，如果新插入的值是链表中的第 9 个，此时binCount= 8
                    // 会触发下面的 treeifyBin，也就是将链表转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash); // 树化：链表转为红黑树
                    break;
                }
                // 如果在该链表中找到了"相等"的 key(== 或 equals)
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 此时 break，那么 e 为链表中[与要插入的新值的 key "相等"]的 node
                    break;
                p = e;
            }
        }
        // e!=null 说明存在旧值的key与要插入的key"相等"
        // 对于我们分析的put操作，下面这个 if 其实就是进行 "值覆盖"，然后返回旧值
        if (e != null) {
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount; // 每次修改+1，是一种fast-fail快速失败机制，是一种容错机制，会抛ConcurrentModification
    // 如果 HashMap 由于新插入这个值导致 size 已经超过了阈值，需要进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

```java
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                                       int h, K k, V v) {
            Class<?> kc = null;
            boolean searched = false;
            TreeNode<K,V> root = (parent != null) ? root() : this;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph; K pk;
                if ((ph = p.hash) > h)     // 小于当前结点，则往左边插入
                    dir = -1;
                else if (ph < h)  //大于当前结点，则往右边插入
                    dir = 1;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0) {
                    if (!searched) {
                        TreeNode<K,V> q, ch;
                        searched = true;
                        if (((ch = p.left) != null &&
                             (q = ch.find(h, k, kc)) != null) ||
                            ((ch = p.right) != null &&
                             (q = ch.find(h, k, kc)) != null))
                            return q;
                    }
                    dir = tieBreakOrder(k, pk);
                }

                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    Node<K,V> xpn = xp.next;
                    TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    xp.next = x;
                    x.parent = x.prev = xp;
                    if (xpn != null)
                        ((TreeNode<K,V>)xpn).prev = x;
                    moveRootToFront(tab, balanceInsertion(root, x));
                    return null;
                }
            }
        }
```

* #### 数组扩容：resize() 方法用于初始化数组或数组扩容，每次扩容后，容量为原来的 2 倍，并进行数据迁移。

  ```java
  final Node<K,V>[] resize() {
      Node<K,V>[] oldTab = table;
      int oldCap = (oldTab == null) ? 0 : oldTab.length;
      int oldThr = threshold;
      int newCap, newThr = 0;
      if (oldCap > 0) { // 对应数组扩容
          if (oldCap >= MAXIMUM_CAPACITY) {
              threshold = Integer.MAX_VALUE;
              return oldTab;
          }
          // 将数组大小扩大一倍
          else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                   oldCap >= DEFAULT_INITIAL_CAPACITY)
              // 将阈值扩大一倍
              newThr = oldThr << 1; // double threshold
      }
      else if (oldThr > 0) // 对应使用 new HashMap(int initialCapacity) 初始化后，第一次 put 的时候
          newCap = oldThr;
      else {// 对应使用 new HashMap() 初始化后，第一次 put 的时候
          newCap = DEFAULT_INITIAL_CAPACITY;
          newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
      }
  
      if (newThr == 0) {
          float ft = (float)newCap * loadFactor;
          newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
      }
      threshold = newThr;
  
      // 用新的数组大小初始化新的数组
      Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
      table = newTab; // 如果是初始化数组，到这里就结束了，返回 newTab 即可
  
      if (oldTab != null) {
          // 开始遍历原数组，进行数据迁移。
          for (int j = 0; j < oldCap; ++j) {
              Node<K,V> e;
              if ((e = oldTab[j]) != null) {
                  oldTab[j] = null;
                  // 如果该数组位置上只有单个元素，那就简单了，简单迁移这个元素就可以了
                  if (e.next == null)
                      newTab[e.hash & (newCap - 1)] = e;
                  // 如果是红黑树，具体我们就不展开了
                  else if (e instanceof TreeNode)
                      ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                  else { 
                      // 这块是处理链表的情况，
                      // 需要将此链表拆成两个链表，放到新的数组中，并且保留原来的先后顺序
                      // loHead、loTail 对应一条链表，hiHead、hiTail 对应另一条链表，代码还是比较简单的
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
                          // 第一条链表
                          newTab[j] = loHead;
                      }
                      if (hiTail != null) {
                          hiTail.next = null;
                          // 第二条链表的新的位置是 j + oldCap，这个很好理解
                          newTab[j + oldCap] = hiHead;
                      }
                  }
              }
          }
      }
      return newTab;
  }
  ```

### get分析过程

- 计算 key 的 hash 值，根据 hash 值找到对应数组下标: `hash & (length-1)`.

- 判断数组该位置处的元素是否刚好就是我们要找的，如果不是，走第三步.

- 判断该元素类型是否是 TreeNode，如果是，用红黑树的方法取数据，如果不是，走第四步.

- 遍历链表，直到找到相等(==或equals)的 key.

- ```
  public V get(Object key) {
      Node<K,V> e;
      return (e = getNode(hash(key), key)) == null ? null : e.value;
  }
  ```

  ```java
  final Node<K,V> getNode(int hash, Object key) {
      Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
      if ((tab = table) != null && (n = tab.length) > 0 &&
          (first = tab[(n - 1) & hash]) != null) {
          // 判断第一个节点是不是就是需要的
          if (first.hash == hash && // always check first node
              ((k = first.key) == key || (key != null && key.equals(k))))
              return first;
          if ((e = first.next) != null) {
              // 判断是否是红黑树
              if (first instanceof TreeNode)
                  return ((TreeNode<K,V>)first).getTreeNode(hash, key);
  
              // 链表遍历
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


### 红黑树退化为链表

resize的时候，对红黑树进行了拆分

resize的时候，判断节点类型，如果是链表，则将链表拆分，如果是TreeNode，则执行TreeNode的split方法分割红黑树，而split方法中将红黑树转换为链表的分支如下：

```java
//在这之前的逻辑是将红黑树每个节点的hash和一个bit进行&运算，
//根据运算结果将树划分为两棵红黑树，lc表示其中一棵树的节点数
if (lc <= UNTREEIFY_THRESHOLD)
    tab[index] = loHead.untreeify(map);
else {
    tab[index] = loHead;
    if (hiHead != null) // (else is already treeified)
        loHead.treeify(tab);
}
```

这里才用到了 UNTREEIFY_THRESHOLD 的判断，当红黑树节点元素小于等于6时，才调用untreeify方法转换回链表

# jdk8 concurentHashMap

##### ConcurrentHashMap取消了segment分段锁，而采用CAS+synchronized来保证并发安全，整个看起来就像是优化过且线程安全的HashMap。数据结构跟HashMap1.8的结构一样，**Node数组+链表+红黑树**。 synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍。桶中的结构可能是链表，也可能是红黑树，红黑树是为了提高查找效率。

spring ioc容器里保存的所有BeanFactory创建的单例bean 都放在一个 ConcurrentHashMap 中

TreeBin：红黑树节点      Node：链表节点 

![img](../../../../Software/Typora/Picture/2843224-59ca97f3ae25b043.png)

```java
  // node数组最大容量：2^30=1073741824  
  private  static  final  int  MAXIMUM_CAPACITY =  1  <<  30    ;  
  // 容量默认初始值，必须是2的幂数  
  private  static  final  int  DEFAULT_CAPACITY =  16    ;  
  //数组可能最大值，需要与toArray（）相关方法关联  
  static  final  int  MAX_ARRAY_SIZE = Integer.MAX_VALUE -  8    ;  
  //并发级别，遗留下来的，为兼容以前的版本  
  private  static  final  int  DEFAULT_CONCURRENCY_LEVEL =  16    ;  
  // 负载因子  
  private  static  final  float  LOAD_FACTOR =  0.75f;  
  // 链表转红黑树阀值,> 8 链表转换为红黑树  
  static  final  int  TREEIFY_THRESHOLD =  8    ;  
  //树转链表阀值，小于等于6（tranfer时，lc、hc=0两个计数器分别++记录原bin、新binTreeNode数量，<=UNTREEIFY_THRESHOLD 则untreeify(lo)）  
  static  final  int  UNTREEIFY_THRESHOLD =  6    ;  
  static  final  int  MIN_TREEIFY_CAPACITY =  64    ;  
  private  static  final  int  MIN_TRANSFER_STRIDE =  16    ;  
  private  static  int  RESIZE_STAMP_BITS =  16    ;  
  // 2^15-1，help resize的最大线程数  
  private  static  final  int  MAX_RESIZERS = (    1  << (    32  - RESIZE_STAMP_BITS)) -  1    ;  
  // 32-16=16，sizeCtl中记录size大小的偏移量  
  private  static  final  int  RESIZE_STAMP_SHIFT =  32  - RESIZE_STAMP_BITS;  
  // forwarding nodes的hash值  
  static  final  int  MOVED     = -    1    ;  
  // 树根节点的hash值  
  static  final  int  TREEBIN   = -    2    ;  
  // ReservationNode的hash值  
  static  final  int  RESERVED  = -    3    ;  
  // 可用处理器数量  
  static  final  int  NCPU = Runtime.getRuntime().availableProcessors();  
  //存放node的数组  
  transient  volatile  Node<K,V>[] table;  
  /*控制标识符，用来控制table的初始化和扩容的操作，不同的值有不同的含义  
  *当为负数时：-    1    代表正在初始化，-N代表有N-    1    个线程正在 进行扩容  
  *当为    0    时：代表当时的table还没有被初始化  
  *当为正数时：表示初始化或者下一次进行扩容的大小  
*/
  private  transient  volatile  int  sizeCtl;  
```



* HashMap没有的属性

  ```java
  /**
       hash表初始化或扩容时的一个控制位标识量。
       负数代表正在进行初始化或扩容操作
       -1代表正在初始化
       -N 表示有N-1个线程正在进行扩容操作
       正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小
       */
      private transient volatile int sizeCtl; 
      // 以下两个是用来控制扩容的时候 单线程进入的变量
      /**
       * The number of bits used for generation stamp in sizeCtl.
       * Must be at least 6 for 32bit arrays.
       */
      private static int RESIZE_STAMP_BITS = 16;
      /**
       * The bit shift for recording size stamp in sizeCtl.
       */
      private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
      
      /*
       * Encodings for Node hash fields. See above for explanation.
       */
      static final int MOVED     = -1; // hash值是-1，表示这是一个forwardNode节点
      static final int TREEBIN   = -2; // hash值是-2  表示这时一个TreeBin节点
  ```

### Node

Node是ConcurrentHashMap存储结构的基本单元，继承于HashMap中的Entry，用于存储数据。就是一个链表，但是只允许对数据进行查找，不允许进行修改

### TreeNode

TreeNode继承于Node，但是数据结构换成了二叉树结构，它是红黑树的数据的存储结构，用于红黑树中存储数据，当链表的节点数大于8时会转换成红黑树的结构，他就是通过TreeNode作为存储结构代替Node来转换成黑红树源代码如下

### TreeBin

TreeBin从字面含义中可以理解为存储树形结构的容器，而树形结构就是指TreeNode，所以TreeBin就是封装TreeNode的容器，它提供转换黑红树的一些条件和锁的控制

### **利用CAS和Synchronized进行高效的同步更新数据。**

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

    /** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    //ConcurrentHashMap 不允许插入null键，HashMap允许插入一个null键
    if (key == null || value == null) throw new NullPointerException();
    //计算key的hash值
    int hash = spread(key.hashCode());
    int binCount = 0;
    //for循环的作用：因为更新元素是使用CAS机制更新，需要不断的失败重试，直到成功为止。
    for (Node<K,V>[] tab = table;;) {
        // f：链表或红黑二叉树头结点，向链表中添加元素时，需要synchronized获取f的锁。
        Node<K,V> f; int n, i, fh;
        //判断Node[]数组是否初始化，没有则进行初始化操作
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        //通过hash定位Node[]数组的索引坐标，是否有Node节点，如果没有则使用CAS进行添加（链表的头结点），添加失败则进入下次循环。
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //检查到内部正在移动元素（Node[] 数组扩容）
        else if ((fh = f.hash) == MOVED)
            //帮助它扩容
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            //锁住链表或红黑二叉树的头结点
            synchronized (f) {
                //判断f是否是链表的头结点
                if (tabAt(tab, i) == f) {
                    //如果fh>=0 是链表节点
                    if (fh >= 0) {
                        binCount = 1;
                        //遍历链表所有节点
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //如果节点存在，则更新value
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            //不存在则在链表尾部添加新节点。
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    //TreeBin是红黑二叉树节点
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        //添加树节点
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                      value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            
            if (binCount != 0) {
                //如果链表长度已经达到临界值8 就需要把链表转换为树结构
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    //将当前ConcurrentHashMap的size数量+1
    addCount(1L, binCount);
    return null;
}
```

### 注意点

* 判断Node[]数组是否初始化，没有则进行初始化操作

* 通过hash定位Node[]数组的索引坐标，是否有Node节点，如果没有则使用CAS进行添加（链表的头结点），添加失败则进入下次循环。

* 检查到内部正在扩容，如果正在扩容，就帮助它一块扩容。

* 如果f != null，则使用synchronized锁住f元素（链表/红黑二叉树的头元素）

  如果是Node(链表结构)则执行链表的添加操作。
  如果是TreeNode(树型结果)则执行树添加操作。

* 判断链表长度已经达到临界值8 就需要把链表转换为树结构。

    JDK8中的concurrentHashMap 实现是锁分离的思想，它把锁分的比segment（JDK1.7）更细一些，只要hash不冲突，就不会出现并发获得锁的情况。它首先使用无锁操作CAS插入头结点，如果插入失败，说明已经有别的线程插入头结点了，再次循环进行操作。如果头结点已经存在，则通过synchronized获得头结点锁，进行后续的操作。性能比segment分段锁又再次提升。

### Synchronized在1.6之后锁升级的过程

一个对象一开始是无锁的状态，或者说是可偏向状态，会先去判断下，再进行升级过程。锁是支持偏向锁的，当前获取到锁的这个线程，会优先让它再去获取到这个锁，如果没有获取到这个锁，就升级成一个轻量级的CAS锁，即乐观锁，乐观锁是比较和交换的过程；如果CAS没有设置成功就会进行一个自旋，自旋到一定次数才会升级成一个Synchronized这个重量级锁，保证了性能问题

# 面试

### 为什么HashMap使用红黑树而不使用AVL树

* CurrentHashMap中加读写锁，如果写冲突就会等待，如果插入时间过长必然等待时间更长，而红黑树相对AVL树他的插入更快！

* 插入和删除方面，AVL树速度较慢：需要更高的旋转次数才能在修改时正确地重新平衡数据结构。
* 两个都给O（log n）查找，但平衡AVL树可能需要O（log n）旋转，而红黑树将需要最多**两次**旋转使其达到平衡
* AVL树的旋转比红黑树的旋转更加难以平衡和调试。















