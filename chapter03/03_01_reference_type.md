<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Java中的引用类型](#java%E4%B8%AD%E7%9A%84%E5%BC%95%E7%94%A8%E7%B1%BB%E5%9E%8B)
  - [强引用](#%E5%BC%BA%E5%BC%95%E7%94%A8)
  - [软引用](#%E8%BD%AF%E5%BC%95%E7%94%A8)
  - [弱引用](#%E5%BC%B1%E5%BC%95%E7%94%A8)
  - [虚引用](#%E8%99%9A%E5%BC%95%E7%94%A8)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Java中的引用类型 

在C/C++中有指针这个概念，指针直接指向了对象所在的内存地址。而Java中则没有指针的概念，如果我们要找到一个对象，我们一般是通过引用的方式找到。

在Java中一共有4种引用方式，分别是：强引用、软引用、弱引用、虚引用，它们的引用强度依次递减。引用类型与垃圾回收也有一定的关联，不同的引用强度对于GC的意义是不一样的。

## 强引用

强引用是Java引用类型中最强的一种引用。强引用传递给GC的信息是：只要一个对象有对应的强引用存在，那么这个对象就永远不可以被回收。

```java
String str = new String();
```

上面代码中的`str`变量就指向了一个String实例，这个实例引用有了一个引用，那么这个实例就永远不会被回收。

## 软引用

软引用传递给GC的信息是：软引用所指向的对象是可以被回收的。垃圾回收器会保证在抛出OutOfMemoryError 错误之前，回收掉所有软引用可达的对象。创建软引用如下：

```java
SoftReference<Widget> softWidget = new SoftReference<Widget>(widget);
```

由于软引用可到达的对象比弱引用可达到的对象滞留内存时间会长一些，我们可以利用这个特性来做上面说到的图片缓存。当内存充足的时候，所有的已经加载过的图片都会在缓存中，这时候加载图片的速度就很快。而如果内存不充足，那么垃圾回收器就会回收掉部分已经加载的图片。

## 弱引用

弱引用传递给垃圾回收器的信息是：在判断一个对象是否存活时，可以不考虑弱引用的存在。比如，指向一个对象的既有一个强引用，又有多个弱引用，当该强引用被移除之后，这个对象就可以被垃圾回收器回收，可以忽略指向该对象的弱引用。创建弱引用如下：

```java
WeakReference<Widget> weakWidget = new WeakReference<Widget>(widget);
```

使用weakWidget.get()就可以得到真实的Widget对象，因为弱引用不能阻挡垃圾回收器对其回收，你会发现（当没有任何强引用到widget对象时）使用get时突然返回null。

## 虚引用

与软引用，弱引用不同，虚引用指向的对象十分脆弱，我们不可以通过get方法来得到其指向的对象。它的唯一作用就是当其指向的对象被回收之后，自己被加入到引用队列，用作记录该引用指向的对象已被销毁。
