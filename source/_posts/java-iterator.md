---
title: java源代码赏析 —— 迭代器（Iterator）
date: 2018-05-14 21:25:54
tags: java 源码
categories: java
---
#### (1) java集合类都在java.util包下，都可以追溯到一个接口——iterator(迭代器)。
##### iterator是对原java Enumeration的替代。下图展示了java集合家族的关系：
![java集合家谱](iterator.png)
#### (2) iterator 接口方法：
```java
public interface Iterator<E> {
    //如果迭代器中还有元素，则返回true
    boolean hasNext();
    //返回迭代器中的下一个元素
    E next();
    //删除迭代器新返回的元素
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
    //为每个剩余元素执行给定的操作,直到所有的元素都已经被处理或行动将抛出一个异常
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```
##### 对于 hasNext(),next(),remove(),用List来实际操作一下用法：
```java
public static void main(String[] args) {
    List<String> list = new ArrayList<String>();
    list.add("a");
    list.add("b");
    list.add("c");
    Iterator i = l.iterator();
    while (i.hasNext()) {
        System.out.println(i.next());
        i.remove();
    }
    System.out.println("list size:"+list.size());
}

/** 输出结果：
* a
* b
* c
* list size:0
*/
```
#### (3) 与之相近的 iterable (java.lang包下)
##### Collection实现了iterable接口， iterable接口提供方法：
```java
public interface Iterable<T> {
    Iterator<T> iterator();
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```
##### iterable 提供了获取Iterator的方法，可以获得Iterator迭代器的相应功能。
#### (4) Iterator和Iterable 区别：
##### 4.1 所属包不同，Iterator在java.util包下，Iterable在java.lang包下
##### 4.2 Iterator是迭代器类，而Iterable是接口。 
##### 4.3 Iterator接口的核心方法next()或者hasNext() 是依赖于迭代器的当前迭代位置的。 如果Collection直接实现Iterator接口，势必导致集合对象中包含当前迭代位置的数据(指针)。  当集合在不同方法间被传递时，由于当前迭代位置不可预置，那么next()方法的结果会变成不可预知。  除非再为Iterator接口添加一个reset()方法，用来重置当前迭代位置。 但即时这样，Collection也只能同时存在一个当前迭代位置。而Iterable则不然，每次调用都会返回一个从头开始计数的迭代器。 多个迭代器是互不干扰的。 
