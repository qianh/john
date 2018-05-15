---
title: java源代码赏析 —— Object类
tags: java 源码
categories: java
---
###  Object类 —— 一切类的父类
#### (1) Object类的声明

``` java
package java.lang; //Object 类定义在java.lang 包下
public class Object
```
#### (2) registerNatives 方法
##### 通常情况下，为了使JVM发现您的本机功能，他们被一定的方式命名。例如，对于java.lang.Object.registerNatives，对应的C函数命名为Java_java_lang_Object_registerNatives。通过使用registerNatives（或者更确切地说，JNI函数RegisterNatives），您可以命名任何你想要你的C函数。
``` java
private static native void registerNatives();
static {
    registerNatives();
}
```
#### (3) hashCode 方法

``` java
public native int hashCode();
```
#### (4) <font color="#dd0000"> (重要) </font> equals() 方法 —— 比较对象的引用
##### <u>Object类的equals()方法比较的是对象的引用而不是内容</u>，这与实际生活的情况不符，实际生活中人们要求比较的是内容,比较引用没有意义，所以<u>其他类继承Object类之后,往往需要重写equals()方法</u>

``` java
public boolean equals(Object obj) {
  return (this == obj)
}
```
#### (5) <font color="#dd0000"> (重要) </font> toString() 方法 —— 打印对象的信息
##### <u>打印一个对象就是调用toString()方法</u>，该方法打印这个对象的信息，<u>其他类在继承Object类后，一般要重写此方法</u>，设置打印关于对象的信息的操作。
``` java
public String toString() {
  return getClass().getName()+"@"+Integer.toHexString(hashCode());
}
```
#### (6) 线程的等待
``` java
public final void wait() throws InterruptedException {
  wait(0);
}
public final native void wait(long timeout) throws InterruptedException;
```
##### 还可以指定等待的最长毫秒和纳秒
``` java
public final void wait(long timeout, int nanos) throws InterruptedException {
  if(timeout < 0) {
    throw new IllegalArgumentException("timeout value is negative");
  }
  if(nanos < 0 || nanos > 999999) {
    throw new IllegalArgumentException("nanosecond timeout value out of range");
  }
  if(nanos >= 500000 || (nanos != 0 && timeout == 0)) {
    timeout++;
  }
  wait(timeout);
}
```
#### (7) 唤醒等待的线程
##### 7.1 唤醒一个线程
``` java
public final native void notify();
```
##### 7.2 唤醒全部线程
###### 唤醒全部线程，哪个线程的优先级高，他就有可能先执行
``` java 
public final native void notifyAll();
```
#### (8) finalize() 方法 —— 对象的回收
##### 当确定一个对象不会被其他方法再使用时，该对象就没有存在的意义了，就只能等待JVM的垃圾回收线程来回收了。垃圾回收是以占用一定内存为代价的。System.gc();就是启动垃圾回收线程的语句。当用户认为需要回收时，可以使用Runtime.getRuntime().gc();或者System.gc();来回收内存。（System.gc();调用的就是Runtime类的 gc()方法）
##### 当一个对象在回收前想要执行一些操作，就要覆写Object类中的finalize()方法。
``` java
protected void finalize() throws Throwable {}
```
##### 注意到抛出的是Throwable，说明除了常规的异常Exception外，还有可能是JVM错误。说明调用该方法不一定只会在程序中产生异常，还有可能产生JVM错误。
#### （9）clone()方法 —— 用于对象克隆
##### Object类的直接或间接子类通过在重写该方法并直接调用本类的该方法完成对象克隆。
``` java
protected native Object clone() throws CloneNotSupportedException;
```
##### native 关键字表示调用本机操作系统的函数。