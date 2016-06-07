<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [方法调用](#%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8)
  - [解析调用](#%E8%A7%A3%E6%9E%90%E8%B0%83%E7%94%A8)
  - [分派调用](#%E5%88%86%E6%B4%BE%E8%B0%83%E7%94%A8)
    - [静态分派](#%E9%9D%99%E6%80%81%E5%88%86%E6%B4%BE)
    - [动态分派](#%E5%8A%A8%E6%80%81%E5%88%86%E6%B4%BE)
    - [虚拟机动态分派的实现](#%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%8A%A8%E6%80%81%E5%88%86%E6%B4%BE%E7%9A%84%E5%AE%9E%E7%8E%B0)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 方法调用

方法调用并不等同于方法执行，方法调用阶段的唯一任务就是确定被调用方法的版本（即调用哪一个方法），暂时还不涉及方法内部的具体执行过程。

## 解析调用

解析指的是调用目标在程序代码写好、编译器进行编译时就必须确定下来执行哪一个方法。而要在编译时就确定执行的是哪一个方法，其实就是要符合“编译器可知，运行期不可变”这个规则。在Java语法中，符合这个条件的有**静态方法、私有方法、实例构造器、父类方法**。下面代码中静态方法sayHello()只可能属于类型StaticResolution，没有任何手段可以覆盖或隐藏这个类。

```java
public class StaticResolution{
	public static void sayHello(){
		System.out.println("hello world.");
	}
	public static void main(String args[]){
		StaticResolution.sayHello();
	}
}
```

解析调用一定是个静态的过程，在编译期间就完全确定，在类装载的解析阶段就会把涉及的符号引用全部转变为可确定的直接引用，不会延迟到运行期再去完成。

## 分派调用

分派调用可能是静态的也可能是动态的，它要根据分派依据的宗量数确定。单双分派和静动态分派双双组合，就有了四种组合：静态单分派、静态多分派、动态单分派、动态多分派。

### 静态分派

```java
Human man = new Man();
man = new Woman();  //实际类型改变，但静态类型不变
```

上面代码中`Human`称为变量的**静态类型**，后面的`Man`称为变量的实际类型。变量最终的静态类型在编译器是可知的，而实际类型变化需要等到运行期才可确定。

观察下面的代码将输出什么？

当程序执行到`sr.sayHello(man);`的时候，JVM判断man的静态类型为Human，于是选择`sayHello(Human)`作为调用目标。而当程序执行到`sr.sayHello(woman);`的时候，JVM判断man的静态类型也为Human，于是选择`sayHello(Human)`作为调用目标。这时候，你如果将程序中的`sayHello(Human human)`方法去掉，那么就会报错。

根据上面代码的分析结果，我们可以明确一个原则：**重载时调用哪个方法，是由参数的静态类型决定的**。

但很多时候重载版本并不是唯一的，往往只能确定一个比较适合的版本。比如我们调用同一个方法不同参数的时候，参数就会根据：`char->int->long->float->double->Character->Serializable->Object->边长参数`进行选择。

### 动态分派

动态分派一般与重写有很大的关联，如果一个方法在子类中被重写了，那么JVM在动态连接的时候进行动态分派时，会根据对象实例的实际类型来分派方法执行版本。

```java
360 qq
```

### 虚拟机动态分派的实现

虚拟机动态分派其实是通过虚方法表来实现的。