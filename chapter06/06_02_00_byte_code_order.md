<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [字节码指令简介](#%E5%AD%97%E8%8A%82%E7%A0%81%E6%8C%87%E4%BB%A4%E7%AE%80%E4%BB%8B)
  - [字节码与数据类型](#%E5%AD%97%E8%8A%82%E7%A0%81%E4%B8%8E%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 字节码指令简介

Java虚拟机的指令是由一个字节长度的、代表着某种特定操作含义的数字（称为操作码，Opcode）以及跟随其后的零至多个代表此操作所需参数（称为操作数，Operands）而构成。但是因为Java虚拟机采用面向操作数栈而不是寄存器的架构，所以大多数的指令都不包含操作数，只有一个操作码。

Java虚拟机的操作码只能是一个字节大小，这意味着指令集的操作码总数不可能超过256条；又由于Class文件格式放弃了编译后代码的操作数长度对齐，这就意味着虚拟机处理那些超过一个字节数据的时候，不得不在运行时从字节中构建出数据的结构，因此牺牲了效率，但是却是的字节码变小了，适合网络传输。这就是为什么Java Web在网络方面如此流行的原因。

## 字节码与数据类型

在Java虚拟机的指令集中，大多数指令都包含了其操作所对应的数据类型信息。例如，iload指令用于从局部变量表中加载int型数据到操作数栈中，而fload指令则加载的是float类型的数据。这两条指令的操作在虚拟机内部可能是由同一段代码来实现，但在Class文件中它们必须拥有不同的操作码。

在Java虚拟机中，对于大部分与数据类型有关的字节码指令，指令中的数据类型会用特定的助记符来表示：

|数据类型|助记符|
|:---:|:---:|
|int|i|
|long|l|
|short|s|
|byte|b|
|char|c|
|float|f|
|double|d|
|reference|a|

对于没有知名操作类型的指令，如 arraylength 指令，它没有代表数据类型的特殊字符，但操作数只能是一个数组类型的对象。还有一些指令，如无条件跳转指令goto，其于数据类型无关。

前面说到了数据类型有8种之多，而操作又多大加减乘除等，如果每一种数据类型和操作都用一个操作符来表示，那么Java虚拟机的256个指令根本就不够存放。于是Java虚拟机的指令集对于特定的操作只提供了有限的类型相关指令去支持它。其中Java虚拟机采用的一条规则是：大部分的指令都没有支持整数类型byte、char、short、boolean，但编译器会将其转化成对于int类型指令的调用。所以大多数对于byte、char、short、boolean的操作实际上都是使用int类型作为运算类型。

