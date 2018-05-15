---
title: java 知识点
date: 2018-05-04 10:37:54
tags: java
categories: java
---
### 整理一些 java 中的常备知识点
#### （1）原始数据类型相关知识
原始数据类型(封装类)：<u>boolean(Integer)，byte(Byte)，short(Short)，char(Character)，int(Integer)，float(Float)，long(Long)，double(Double)</u>
* ##### *boolean(封装类 Boolean)*
1. boolean 数据类型非 true 即 false。这个数据类型表示 1 bit，但是它的大小并没有精确定义
2. boolean 类型单独使用是4个字节，在数组中又是1个字节
3. boolean 不用byte或short，而是使用 int 的原因是：对于32位的CPU来说，一次进行32位的数据交换更加高效

综上，官方文档对boolean类型没有给出精确的定义，《Java虚拟机规范》给出了“单独时使用4个字节，boolean数组时1个字节”的定义，具体还要看虚拟机实现是否按照规范来，所以1个字节、4个字节都是有可能的。这其实是一种时空权衡。
* ##### *byte(封装类 Byte)*
byte 使用是 1 个字节, 1 byte = 8 bit
* ##### *short(封装类 Short)*
short 使用是 2 个字节
* ##### *char(封装类 Character)*
char 使用是 2 个字节
* ##### *int(封装类 Integer)*
int 使用是 4 个字节
* ##### *float(封装类 Float)*
float 单精度浮点数，使用是 4 个字节
* ##### *long(封装类 Long)*
long 长整型，使用是 8 个字节
* ##### *double(封装类 Double)*
double 双精度浮点数，使用是 8 个字节

#### （2）"==" 与 "equals()" 的区别
1. 《Thinking in Java》中说：“关系操作符生成的是一个boolean结果，它们计算的是操作数的值之间的关系”。
2. "=="判断的是两个对象的内存地址是否一样，适用于原始数据类型和枚举类型（它们的变量存储的是值本身，而引用类型变量存储的是引用）；
3. equals是Object类的方法，Object对它的实现是比较内存地址，<strong>我们可以重写这个方法来自定义“相等”这个概念。</strong>比如类库中的String、Date等类就对这个方法进行了重写。

<strong>综上，对于枚举类型和原始数据类型的相等性比较，应该使用"=="；对于引用类型的相等性比较，应该使用equals方法。</strong>

#### （3）Object中定义了哪些方法？
``` java
hashCode(),clone(),wait(),notify(),notifyAll(),equals(),toString(),finalize(),getClass()
```
#### （4）hashCode的作用？
在很多地方都会利用到hash表来提高查找效率。在Java的Object类中有一个方法:
``` java
public native int hashCode();
```
该方法返回一个int类型的数值，并且是本地方法，因此在Object类中并没有给出具体的实现。hashCode方法默认返回对象的内存地址

hashCode方法的主要作用是为了配合基于散列的集合一起正常运行，这样的散列集合包括HashSet、HashMap以及HashTable。

* 场景：当向集合中插入对象时，如何判别在集合中是否已经存在该对象了？

如果用equals方法，需要逐个比较，但是集合数据多的时候就比较耗性能了。

此时hashCode方法的作用就体现出来了，当集合要添加新的对象时，先调用这个对象的hashCode方法，得到对应的hashcode值，实际上在HashMap的具体实现中会用一个table保存已经存进去的对象的hashcode值，如果table中没有该hashcode值，它就可以直接存进去，不用再进行任何比较了；如果存在该hashcode值， 就调用它的equals方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址，所以这里存在一个冲突解决的问题，这样一来实际调用equals方法的次数就大大降低了，说通俗一点：Java中的hashCode方法就是根据一定的规则将与对象相关的信息（比如对象的存储地址，对象的字段等）映射成一个数值，这个数值称作为散列值。

java.util.HashMap的中put方法的具体实现：
``` java
public V put(K key, V value) {
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```
put方法是用来向HashMap中添加新的元素，从put方法的具体实现可知，会先调用hashCode方法得到该元素的hashCode值，然后查看table中是否存在该hashCode值，如果存在则调用equals方法重新确定是否存在该元素，如果存在，则更新value值，否则将新的元素添加到HashMap中。从这里可以看出，hashCode方法的存在是为了减少equals方法的调用次数，从而提高程序效率。

* equals方法和hashCode方法
1. 在程序执行期间，<font color="#dd0000">只要equals方法的比较操作用到的信息没有被修改，那么对这同一个对象调用多次，hashCode方法必须始终如一地返回同一个整数。</font>
2. 如果两个对象根据equals方法比较是相等的，那么调用两个对象的hashCode方法必须返回相同的整数结果。
3. 如果两个对象根据equals方法比较是不等的，则hashCode方法不一定得返回不同的整数。

在《Java编程思想》一书中的有同第一条类似的一段话：
　　“设计hashCode()时最重要的因素就是：无论何时，对同一个对象调用hashCode()都应该产生同样的值。如果在讲一个对象用put()添加进HashMap时产生一个hashCdoe值，而用get()取出时却产生了另一个hashCode值，那么就无法获取该对象了。所以如果你的hashCode方法依赖于对象中易变的数据，用户就要当心了，因为此数据发生变化时，hashCode()方法就会生成一个不同的散列码”。

#### （5）ArrayList, LinkedList, Vector的区别是什么？
* ArrayList: 内部采用数组存储元素，支持高效随机访问，支持动态调整大小
* LinkedList: 内部采用链表来存储元素，支持快速插入/删除元素，但不支持高效地随机访问
* Vector: 可以看作线程安全版的ArrayList

#### （6）String, StringBuilder, StringBuffer的区别是什么？
* String: 不可变的字符序列，若要向其中添加新字符需要创建一个新的String对象
* StringBuilder: 可变字符序列，支持向其中添加新字符（无需创建新对象）
* StringBuffer: 可以看作线程安全版的StringBuilder

#### （7）Map, Set, List, Queue, Stack的特点及用法。
* Map<K, V>: Java中存储键值对的数据类型都实现了这个接口，表示“映射表”。支持的两个核心操作是get(Object key)以及put(K key, V value)，分别用来获取键对应的值以及向映射表中插入键值对。
* Set<E>: 实现了这个接口的集合类型中不允许存在重复的元素，代表数学意义上的“集合”。它所支持的核心操作有add(E e),* remove(Object o)*, contains(Object o)，分别用于添加元素，删除元素以及判断给定元素是否存在于集中。
* List<E>: Java中集合框架中的列表类型都实现了这个接口，表示一种有序序列。支持get(int index), add(E e)等操作。
* Queue<E>: Java集合框架中的队列接口，代表了“先进先出”队列。支持add(E element), remove()等操作。
* Stack<E>: Java集合框架中表示堆栈的数据类型，堆栈是一种“后进先出”的数据结构。支持push(E item), pop()等操作。

#### （8）HashMap和HashTable的区别
* HashTable是线程安全的，而HashMap不是
* HashMap中允许存在null键和null值，而HashTable中不允许

#### （9）HashMap的实现原理
Java中数据存储方式最底层的两种结构，一种是数组，另一种就是链表
* 数组的特点：连续空间，寻址迅速，但是在删除或者添加元素的时候需要有较大幅度的移动，所以查询速度快，增删较慢。
* 链表正好相反，由于空间不连续，寻址困难，增删元素只需修改指针，所以查询慢、增删快。

为了查询和增删都快的目的，出现了哈希表。
* 哈希表具有较快（常量级）的查询速度，及相对较快的增删速度，所以很适合在海量数据的环境中使用。一般实现哈希表的方法采用“拉链法”，我们可以理解为“链表的数组”。HashMap的底层实现是“基于拉链法的散列表”。
* [详细分析请参考深入解析HashMap、HashTable](https://blog.csdn.net/tgxblue/article/details/8479147/)

#### （10）ConcurrentHashMap的实现原理
* ConcurrentHashMap是支持并发读写的HashMap，它的特点是读取数据时无需加锁，写数据时可以保证加锁粒度尽可能的小。由于其内部采用“分段存储”，只需对要进行写操作的数据所在的“段”进行加锁。
* [Java并发编程：并发容器之ConcurrentHashMap](http://www.cnblogs.com/dolphin0520/p/3932905.html)

#### （11）TreeMap, LinkedHashMap, HashMap的区别是什么？
* HashMap的底层实现是散列表，因此它内部存储的元素是无序的；
* TreeMap的底层实现是红黑树，所以它内部的元素的有序的。排序的依据是自然序或者是创建TreeMap时所提供的比较器（Comparator）对象。
* LinkedHashMap可以看作能够记住插入元素的顺序的HashMap。

#### （12）Collection与Collections的区别是什么？
* Collection<E>是Java集合框架中的基本接口；
* Collections是Java集合框架提供的一个工具类，其中包含了大量用于操作或返回集合的静态方法。
* [Collections类常用方法总结](https://www.cnblogs.com/Eason-S/p/5786066.html)

#### （13）Java中的异常层次结构
* Throwable类是异常层级中的基类。
* Error类表示内部错误，这类错误使我们无法控制的；
* Exception表示异常，RuntimeException及其子类属于未检查异常，这类异常包括ArrayIndexOutOfBoundsException、NullPointerException等，我们应该通过条件判断等方式语句避免未检查异常的发生。
* IOException及其子类属于已检查异常，编译器会检查我们是否为所有可能抛出的已检查异常提供了异常处理器，若没有则会报错。对于未检查异常，我们无需捕获（当然Java也允许我们捕获，但我们应该做的事避免未检查异常的发生）。

#### （14）Java面向对象的三个特征与含义
*  三大特征：封装、继承、多态。[详细介绍请戳Java面向对象三大特性](http://javabc.baike.com/article-692235.html)

#### （15）Override, Overload的含义与区别
* Override表示“重写”，是子类对父类中同一方法的重新定义
* Overload表示“重载”，也就是定义一个与已定义方法名称相同但签名不同的新方法

#### （16）接口与抽象类的区别
* 接口是一种约定，实现接口的类要遵循这个约定；
* 抽象类本质上是一个类，使用抽象类的代价要比接口大。
接口与抽象类的对比如下：
1. 抽象类中可以包含属性，方法（包含抽象方法与有着具体实现的方法），常量；接口只能包含常量和方法声明。
2. 抽象类中的方法和成员变量可以定义可见性（比如public、private等）；而接口中的方法只能为public（缺省为public）。
3. 一个子类只能有一个父类（具体类或抽象类）；而一个接口可以继承一个或多个接口，一个类也可以实现多个接口。
4. 子类中实现父类中的抽象方法时，可见性可以大于等于父类中的；而接口实现类中的接口 方法的可见性只能与接口中相同（public）。

#### （17）Java中多态的实现原理
* 所谓多态，指的就是父类引用指向子类对象，调用方法时会调用子类的实现而不是父类的实现。多态的实现的关键在于“动态绑定”。
* [Java动态绑定的内部实现机制](https://blog.csdn.net/sureyonder/article/details/5569617)

#### （18）简述Java中创建新线程的两种方法
1. ##### 继承Thread类。
###### 1）定义Thread类的子类，并重写该类的run()方法，该方法的方法体就是线程需要完成的任务，run()方法也称为线程执行体。
###### 2）创建Thread子类的实例，也就是创建了线程对象
###### 3）启动线程，即调用线程的start()方法
``` java 
public class MyThread extends Thread{//继承Thread类
　　public void run(){
　　//重写run方法
　　}
}
public class Main {
　　public static void main(String[] args){
　　　　new MyThread().start();//创建并启动线程
　　}
}
```
2. ##### 实现Runnable接口。
###### 1）定义Runnable接口的实现类，一样要重写run()方法，这个run（）方法和Thread中的run()方法一样是线程的执行体。
###### 2）创建Runnable实现类的实例，并用这个实例作为Thread的target来创建Thread对象，这个Thread对象才是真正的线程对象。
###### 3）第三部依然是通过调用线程对象的start()方法来启动线程。
``` java 
public class MyThread2 implements Runnable {//实现Runnable接口
　　public void run(){
　　//重写run方法
　　}
}
public class Main {
　　public static void main(String[] args){
　　　　//创建并启动线程
　　　　MyThread2 myThread=new MyThread2();
　　　　Thread thread=new Thread(myThread);
　　　　thread().start();
　　　　//或者    new Thread(new MyThread2()).start();
　　}
}
```
3. ##### 使用Callable和Future创建线程。

和Runnable接口不一样，Callable接口提供了一个call()方法作为线程执行体，call()方法比run()方法功能要强大。
###### >>call()方法可以有返回值
###### >>call()方法可以声明抛出异常

在Future接口里定义了几个公共方法来控制它关联的Callable任务。
###### >>boolean cancel(boolean mayInterruptIfRunning)：视图取消该Future里面关联的Callable任务
###### >>V get()：返回Callable里call()方法的返回值，调用这个方法会导致程序阻塞，必须等到子线程结束后才会得到返回值
###### >>V get(long timeout,TimeUnit unit)：返回Callable里call()方法的返回值，最多阻塞timeout时间，经过指定时间没有返回抛出TimeoutException
###### >>boolean isDone()：若Callable任务完成，返回True
###### >>boolean isCancelled()：如果在Callable任务正常完成前被取消，返回True

具体操作步骤：[Callable和Future](https://blog.csdn.net/ghsau/article/details/7451464)

##### 一般推荐采用实现接口的方式来创建多线程
#### [Java线程专栏文章汇总](https://blog.csdn.net/ghsau/article/details/17609747)

#### （19）单例
##### 1，懒汉式
``` java
public class Single {
    private static volatile Single instance;
    private Single() {}
    public static Single getInstance() {
        if(instance == null) {
            synchronized (Single.class) {
                if(instance == null) {
                    instance = new Single();
                }
            }
        }
        return instance;
    }
}
```
##### 1，饿汉式
``` java
public class Single2 {
    private static class SingleHandle() {
        private static final Single2 instance = new Single2();
    }
    private Single2() {}
    public static final Single2 getInstance() {
        return SingleHandle.instance;
    }
}
```

#### （20）排序算法
##### 1 冒泡
``` java 
public void bubbleSort(int[] array) {
    int temp = 0;
    for(int i=0;i<array.length-1;i++) {
        for(int j=0;j<array.length-1-i;j++) {
            if(array[j] > array[j+1]) {
                temp = array[j];
                array[j] = array[j+1];
                array[j+1] = temp;
            }
        }
    }
    System.out.println(Arrays.toString(array));
}
```
##### 2 选择
``` java 
public void selectSort(int array[]) {
    int t = 0;
    for (int i = 0; i < array.length - 1; i++){
        int index=i;
        for (int j = i + 1; j < array.length; j++) {
            if (array[index] > array[j]) {
                index = j;
            }
        }

        if(index!=i){ //找到了比array[i]小的则与array[i]交换位置
            t = array[i];
            array[i] = array[index];
            array[index] = t;
        }
    }
    System.out.println(Arrays.toString(array));
}
```
##### 3 插入
``` java 
public void insertionSort(int array[]) {
    int i, j, t = 0;
    for (i = 1; i < array.length; i++) {
        t = array[i];
        for (j = i - 1; j >= 0 && t < array[j]; j--) {
            array[j + 1] = array[j];
        }
        array[j + 1] = t;
    }
    System.out.println(Arrays.toString(array));
}
```