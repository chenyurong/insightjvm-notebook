<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [早期（编译期）优化](#%E6%97%A9%E6%9C%9F%EF%BC%88%E7%BC%96%E8%AF%91%E6%9C%9F%EF%BC%89%E4%BC%98%E5%8C%96)
  - [Javac编译器](#javac%E7%BC%96%E8%AF%91%E5%99%A8)
    - [Javac的源码与调试](#javac%E7%9A%84%E6%BA%90%E7%A0%81%E4%B8%8E%E8%B0%83%E8%AF%95)
    - [Javac的编译过程](#javac%E7%9A%84%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B)
  - [Java中的语法糖](#java%E4%B8%AD%E7%9A%84%E8%AF%AD%E6%B3%95%E7%B3%96)
    - [泛型与泛型擦除](#%E6%B3%9B%E5%9E%8B%E4%B8%8E%E6%B3%9B%E5%9E%8B%E6%93%A6%E9%99%A4)
    - [自动装箱、拆箱与遍历循环](#%E8%87%AA%E5%8A%A8%E8%A3%85%E7%AE%B1%E3%80%81%E6%8B%86%E7%AE%B1%E4%B8%8E%E9%81%8D%E5%8E%86%E5%BE%AA%E7%8E%AF)
    - [条件编译](#%E6%9D%A1%E4%BB%B6%E7%BC%96%E8%AF%91)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 早期（编译期）优化

## Javac编译器

Javac编译期能把`*.java`文件转变成`*.class`文件，但javac对代码的运行效率基本没有任何优化措施，但是其对程序员编码有提升作用。

### Javac的源码与调试

待补充

### Javac的编译过程

Javac的编译过程大致可以分为以下3个过程，分别是：

- 解析与填充符号表过程
- 插入式注解处理器的注解处理过程
- 分析与字节码生成过程

**解析与填充符号表**

在这个过程，Javac主要进行语法分析、语法分析以及填充符号表操作。

**注解处理器**

在这个过程中，Javac对JDK中支持的注解进行解析。

**语义分析与字节码生成**

在这个过程中Javac会进行标注检查、数据及控制流分析、解语法糖等操作，最后才生成字节码。

## Java中的语法糖

### 泛型与泛型擦除


### 自动装箱、拆箱与遍历循环


### 条件编译

