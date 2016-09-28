<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [类加载时机](#%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%97%B6%E6%9C%BA)
  - [主动引用](#%E4%B8%BB%E5%8A%A8%E5%BC%95%E7%94%A8)
  - [被动引用](#%E8%A2%AB%E5%8A%A8%E5%BC%95%E7%94%A8)
    - [通过子类引用父类的静态字段](#%E9%80%9A%E8%BF%87%E5%AD%90%E7%B1%BB%E5%BC%95%E7%94%A8%E7%88%B6%E7%B1%BB%E7%9A%84%E9%9D%99%E6%80%81%E5%AD%97%E6%AE%B5)
    - [通过定义数组类引用类](#%E9%80%9A%E8%BF%87%E5%AE%9A%E4%B9%89%E6%95%B0%E7%BB%84%E7%B1%BB%E5%BC%95%E7%94%A8%E7%B1%BB)
    - [常量在编译阶段存入调用类的常量池中](#%E5%B8%B8%E9%87%8F%E5%9C%A8%E7%BC%96%E8%AF%91%E9%98%B6%E6%AE%B5%E5%AD%98%E5%85%A5%E8%B0%83%E7%94%A8%E7%B1%BB%E7%9A%84%E5%B8%B8%E9%87%8F%E6%B1%A0%E4%B8%AD)
    - [接口主动初始化的时机](#%E6%8E%A5%E5%8F%A3%E4%B8%BB%E5%8A%A8%E5%88%9D%E5%A7%8B%E5%8C%96%E7%9A%84%E6%97%B6%E6%9C%BA)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 类加载时机

类从被加载到虚拟机内存开始，到卸载出内存为止，它的整个生命周期包括：**加载、验证、准备、解析、初始化、使用和卸载**，其中验证、准备、解析3个部分统称为连接。

![](../img/07/07_01_class_life_cycle.png)

在上面的类生命周期中，加载、验证、准备、初始化和卸载这5个过程的顺序是确定的，类的加载过程会按照这个过程按部就班地进行，但解析阶段则不一定：它在某些情况下可以在初始化阶段之后再开始，这是为了支持Java语言的运行时绑定。

## 类的主动引用

在Java虚拟机规范中规定了以下5种情形必须立即对类进行初始化，这5种场景中的行为称为对一个类进行主动引用。

- 遇到new、getstatic、putstatic 或 invokestatic 这4条字节码指令时，如果类没有进行初始化，则需要先触发其初始化。生成这4条字节码指令最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个雷的静态字段（被final修饰、已在编译器把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
- 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
- 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
- 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
- 当使用JDK1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果是REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

对于上面5种主动引用，虚拟机规范中使用了一个很强烈的限定于：“有且只有”，即有且只有上述5种情况会引发虚拟机主动去初始化。

## 类的被动引用

与主动引用相对的是被动引用，被动引用指的是所有引用类的方式都不会触发初始化。

### 通过子类引用父类的静态字段

当通过子类引用父类的静态字段时，不会导致子类初始化。

```java
public class PassiveReference {
    public static void main(String args[]) {
        System.out.println(SubClass.value);
    }
}

class SuperClass{
    static {
        System.out.println("SuperClass init!");
    }

    public static int value = 123;
}

class SubClass extends SuperClass {
    static {
        System.out.println("SubClass init!");
    }
}
```

运行上面的程序，输出结果：

```
SuperClass init!
123
```

从结果我们可以看出只有父类被初始化了而子类没有被初始化。这是因为对于静态字段，只有直接定义这个字段的类才会被初始化，因此通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。

### 通过定义数组类引用类

为了节省版面，我们复用了上面的SuperClass和SubClass，但修改main()方法的内容：

```java
public class PassiveReference {
    public static void main(String args[]) {
        SuperClass[] sca = new SuperClass[10];
    }
}
```

运行之后发现其根本就没有输出“SuperClass init”，这说明其没有触发SuperClass的初始化。

### 常量在编译阶段存入调用类的常量池中

如果一个属性被`static final`修改，那它就是一个常量，它会在编译阶段就写入调用类的常量池。这时候即使直接引用到定义常量的类，也不会触发定义常量的类的初始化。

```java
public class NotInitialization {

    public static void main(String args[]) {
        System.out.println(ConstClass.HELLO_WORLD);
    }
}

class ConstClass{
    static {
        System.out.println("ConstClass init!");
    }

    public static final String HELLO_WORLD = "hello world!";
}
```

输出结果：

```bash
hello world!
```

### 接口主动初始化的时机

接口与类真正有所区别的是前面讲述的5种“有且只有”需要开始初始化场景中的第3种：当一个类在初始化时，要求其父类全部都已经初始化过了，但是一个接口在初始化时，并不要求其父接口全部都完成了初始化，只有在真正使用到了父接口的时候（如引用接口中定义的常量）才会初始化。

