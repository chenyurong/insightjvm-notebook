<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [常量池](#%E5%B8%B8%E9%87%8F%E6%B1%A0)
  - [字面量](#%E5%AD%97%E9%9D%A2%E9%87%8F)
  - [符号引用](#%E7%AC%A6%E5%8F%B7%E5%BC%95%E7%94%A8)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 常量池

紧接着主次版本号之后是常量池入口，常量池可以理解为 Class 文件之中的资源仓库，它是 Class 文件结构中与其他项目关联最多的数据类型。由于常量池数量不是固定的，所以在常量池的入口需要放置一个 u2 类型的数据，代表常量池容量计数值，并且这个计数器是从 1 而不是 0 开始的。在常量池容量计数值后才是对应的常量值。

|类型|名称|解释|数量|
|:---:|:---:|:---:|:---:|
|u2|constant_pool_count|常量池常量个数|1|
|cp_info|constant_pool|常量池|constant_pool_count - 1|

如下图所示，常量池容量为十六进制数`0x0013`，即十进制数的19，这就代表常量池中有18项常量，索引值范围为1-18。在设计之中将0项常量空出来，是有特殊考虑的，它用来表示“不引用任何一个常量池项目”。

在常量池中主要存放两大类常量：字面量和符号引用。

## 字面量

字面量比较接近于Java语言层面的常量概念，如文本字符串、声明为fina的常量池等。

## 符号引用

而符号引用则属于编译原理方面的概念，包括了下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

其中全限定名指的是包括了包路径的类名。

无论是字面量或符号类型的常量，它们本质上都是一个表。这些表都有一些共同的特点，就是表开始的第一位是一个 u1 类型的标志位（即下表的标志列），代表当前常量属于哪种常量类型。之后携带这种常量类型对应数据结构的数据。在JDK1.7中共有14中常量类型，见下表：

|类型|标志|描述|
|:---:|:---:|:---:|
|CONSTANT_Utf_info|1|UTF-8编码的字符串|
|CONSTANT_Integer_info|3|整型字面量|
|CONSTANT_Float_info|4|浮点型字面量|
|CONSTANT_Long_info|5|长整型字面量|
|CONSTANT_Double_info|6|双精度浮点型字面量|
|CONSTANT_Class_info|7|累活接口的符号引用|
|CONSTANT_String_info|8|字符串类型字面量|
|CONSTANT_Fieldref_info|9|字段的符号引用|
|CONSTANT_Methodref_info|10|类中方法的符号引用|
|CONSTANT_InterfaceMethodref_info|11|接口中方法的符号引用|
|CONSTANT_NameAndType_info|12|字段或方法的部分符号引用|
|CONSTANT_MethodHandle_info|15|表示方法句柄|
|CONSTANT_MethodType_info|16|标识方法类型|
|CONSTANT_InvokeDynamic_info|18|表示一个动态方法调用点|

```bash
CA FE BA BE 00 00 00 33 00 13 0A 00 04 00 0F 09 00 03 00 10 07 00 11 07 00 12 01 00 01 6D 01 00 01 49 01 00 06 3C 69 6E 69 74 3E 01 00 03 28 29 56 01 00 04 43 6F 64 65 01 00 0F 4C 69 6E 65 4E 75 6D 62 65 72 54 61 62 6C 65 01 00 03 69 6E 63 01 00 03 28 29 49 01 00 0A 53 6F 75 72 63 65 46 69 6C 65 01 00 0E 43 6C 61 73 73 44 65 6D 6F 2E 6A 61 76 61 0C 00 07 00 08 0C 00 05 00 06 01 00 11 63 6F 6D 2F 6A 76 6D 2F 43 6C 61 73 73 44 65 6D 6F 01 00 10 6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 65 63 74 00 21 00 03 00 04 00 00 00 01 00 02 00 05 00 06 00 00 00 02 00 01 00 07 00 08 00 01 00 09 00 00 00 1D 00 01 00 01 00 00 00 05 2A B7 00 01 B1 00 00 00 01 00 0A 00 00 00 06 00 01 00 00 00 07 00 01 00 0B 00 0C 00 01 00 09 00 00 00 1F 00 02 00 01 00 00 00 07 2A B4 00 02 04 60 AC 00 00 00 01 00 0A 00 00 00 06 00 01 00 00 00 0C 00 01 00 0D 00 00 00 02 00 0E
```

回头来看下图中常量池的第一项常量，它的标志位是`0x0A`即十进制的10，查上表可得知其属于`CONSTANT_Methodref_info`类型的常量，即其实一个类中方法的符号引用。`CONSTANT_Methodref_info`的数据结构比较简单：

|类型|名称|数量|
|:---:|:---:|:---:|
|u1|tag|值为10|
|u2|index|指向声明方法的描述符CONSTANT_Class_info的索引项|
|u2|index|指向名称及类型描述符CONSTANT_NameAndType的索引项|

其中tag是标志位，上面已经说过。第一个index，其值为`0x0004`，指向声明方法的描述符CONSTANT_Class_info的索引项，其指向了常量池中的第4个常量，通过后文`javap`计算出来的常量池信息，我们可以知道其指向`java/lang/Object`。第二个index，其值为`0x000F`，指向名称及类型描述符CONSTANT_NameAndType的索引项，其指向了第15个常量池，通过后文`javap`计算出来的常量池信息，我们可以得知其指向了`"<init>":()V`，也即是一个无返回值的初始化方法，也即`void TestClass()`。

我们继续分析第二项常量，其标志位为`0x09`，即十进制的9，即`CONSTANT_Fieldref_info`类型的常量，它表示字段的符号引用。这种类型的符号引用的数据结构如下：

|类型|名称|数量|
|:---:|:---:|:---:|
|u1|tag|值为9|
|u2|index|指向声明字段的类或接口描述符CONSTANT_Class_info的索引项|
|u2|index|指向字段描述符CONSTANT_NameAndType的索引项|

紧跟`0x09`的4个字节分别是：`00 03 00 10`。第一个index，其值为`0x0003`，表示字段的类或接口描述符索引项，指向了第三个常量，通过后文`javap`计算出来的常量池信息可知第三个常量是`com/jvm/ClassDemo`，表示该字段是类ClassDemo中的字段。第二个index、其值为`0x0010`，表示字段描述符索引项，指向了第16个常量，通过后文`javap`计算出来的常量池信息可知第16个常量，其值是`m:I`，表示字段名称为m，字段类型是Integer类型。

到这里为止，我们只分析了18个常量中的两个，其余的16个常量可以通过Oracle公司提供的`javap`工具计算出来。使用`javap -verbose [java_file_name]`即可输出对应Java类文件的字节码内容。

```bash
$ javap -verbose com/jvm/ClassDemo
Classfile /Users/yurongchan/workspace_intelJ/ThreadDemo/src/com/jvm/ClassDemo.class
  Last modified May 18, 2016; size 283 bytes
  MD5 checksum afc6af8a5d0757b73b28f824b10e1bdb
  Compiled from "ClassDemo.java"
public class com.jvm.ClassDemo
  SourceFile: "ClassDemo.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#15         //  java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         //  com/jvm/ClassDemo.m:I
   #3 = Class              #17            //  com/jvm/ClassDemo
   #4 = Class              #18            //  java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               ClassDemo.java
  #15 = NameAndType        #7:#8          //  "<init>":()V
  #16 = NameAndType        #5:#6          //  m:I
  #17 = Utf8               com/jvm/ClassDemo
  #18 = Utf8               java/lang/Object
{
  public com.jvm.ClassDemo();
    flags: ACC_PUBLIC
    Code:
……省略其他内容
```

可以看出，计算机已经自动帮我们生成了18项常量内容，并且第1、2项的计算结果与手工计算的结果一致。

[补充常量池14种常量项的结构总表]

<table>
    <tr align="center">
        <td>常量</td><td>项目</td><td>类型</td><td>描述</td>
    </tr>
    <!-- 1 -->
    <tr align="center">
    	<td  rowspan="3">CONSTANT_Utf8_info</td><td>tag</td><td>u1</td><td>值为1</td>
    </tr>
    <tr align="center">
    	<td>length</td><td>u2</td><td>UTF-8编码的字符串占用的字节数</td>
    </tr>
    <tr align="center">
    	<td>bytes</td><td>u1</td><td>长度为length的UTF-8编码的字符串</td>
    </tr>
    <!-- 2 -->
    <tr align="center">
    	<td  rowspan="2">CONSTANT_Integer_info</td><td>tag</td><td>u1</td><td>值为3</td>
    </tr>
    <tr align="center">
    	<td>bytes</td><td>u4</td><td>按照高位在前存储的int值</td>
    </tr>
    <!-- 3 -->
    <tr align="center">
    	<td  rowspan="2">CONSTANT_Float_info</td><td>tag</td><td>u1</td><td>值为4</td>
    </tr>
    <tr align="center">
    	<td>bytes</td><td>u4</td><td>按照高位在前存储的float值</td>
    </tr>
    <!-- 4 -->
    <tr align="center">
    	<td  rowspan="2">CONSTANT_Long_info</td><td>tag</td><td>u1</td><td>值为5</td>
    </tr>
    <tr align="center">
    	<td>bytes</td><td>u8</td><td>按照高位在前存储的long值</td>
    </tr>
    <!-- 5 -->
    <tr align="center">
    	<td  rowspan="2">CONSTANT_Double_info</td><td>tag</td><td>u1</td><td>值为6</td>
    </tr>
    <tr align="center">
    	<td>bytes</td><td>u8</td><td>按照高位在前存储的double值</td>
    </tr>
    <!-- 6 -->
    <tr align="center">
    	<td  rowspan="2">CONSTANT_Class_info</td><td>tag</td><td>u1</td><td>值为7</td>
    </tr>
    <tr align="center">
    	<td>index</td><td>u2</td><td>指向全限定名常量项的索引</td>
    </tr>
    <!-- 7 -->
    <tr align="center">
    	<td  rowspan="2">CONSTANT_String_info</td><td>tag</td><td>u1</td><td>值为8</td>
    </tr>
    <tr align="center">
    	<td>index</td><td>u2</td><td>指向字符串字面量的索引</td>
    </tr>
    <!-- 8 -->
    <tr align="center">
    	<td  rowspan="3">CONSTANT_Fieldref_info</td><td>tag</td><td>u1</td><td>值为9</td>
    </tr>
    <tr align="center">
    	<td>index</td><td>u2</td><td>指向声明字段的类或者接口描述符CONSTANT_Class_info的索引项</td>	
    </tr>
    <tr align="center">
    	<td>index</td><td>u2</td><td>指向字段描述符CONSTANT_NameAndType的索引项</td>
    </tr>
    <!-- 9 -->
    <tr align="center">
    	<td  rowspan="3">CONSTANT_Methodref_info</td><td>tag</td><td>u1</td><td>值为10</td>
    </tr>
    <tr align="center">
    	<td>index</td><td>u2</td><td>指向声明方法的类描述符CONSTANT_Class_info的索引项</td>	
    </tr>
    <tr align="center">
    	<td>index</td><td>u2</td><td>指向名称及类型描述符CONSTANT_NameAndType的索引项</td>
    </tr>
    <!-- 10 -->
    <tr align="center">
    	<td  rowspan="3">CONSTANT_InterfaceMethodref_info</td><td>tag</td><td>u1</td><td>值为11</td>
    </tr>
    <tr align="center">
    	<td>index</td><td>u2</td><td>指向声明方法的接口描述符CONSTANT_Class_info的索引项</td>	
    </tr>
    <tr align="center">
    	<td>index</td><td>u2</td><td>指向名称及类型描述符CONSTANT_NameAndType的索引项</td>
    </tr>
    <!-- 11 -->
    <tr align="center">
    	<td  rowspan="3">CONSTANT_NameAndType_info</td><td>tag</td><td>u1</td><td>值为12</td>
    </tr>
    <tr align="center">
    	<td>index</td><td>u2</td><td>指向该字段或方法名称常量项的索引</td>	
    </tr>
    <tr align="center">
    	<td>index</td><td>u2</td><td>指向该字段或方法描述符常量项的索引</td>
    </tr>
    <!-- 12 -->
    <tr align="center">
    	<td  rowspan="3">CONSTANT_MethodHandle_info</td><td>tag</td><td>u1</td><td>值为15</td>
    </tr>
    <tr align="center">
    	<td>reference_kind</td><td>u1</td><td>值必须在1~9之间（包括1和9），它决定了方法句柄的类型。方法句柄类型的值表示方法句柄的字节码行为</td>	
    </tr>
    <tr align="center">
    	<td>reference_index</td><td>u2</td><td>值必须是对常量池的有效索引</td>
    </tr>
    <!-- 13 -->
    <tr align="center">
    	<td  rowspan="2">CONSTANT_MethodType_info</td><td>tag</td><td>u1</td><td>值为16</td>
    </tr>
    <tr align="center">
    	<td>descriptor_index</td><td>u2</td><td>值必须是对常量池的有效索引，常量池在该索引处的项必须是CONSTANT_Utf8_info结构，表示方法的描述符</td>	
    </tr>  
    <!-- 14 -->
    <tr align="center">
    	<td  rowspan="3">CONSTANT_InvokeDynamic_info</td><td>tag</td><td>u1</td><td>值为18</td>
    </tr>
    <tr align="center">
    	<td>bootstrap_method_attr_index</td><td>u2</td><td>值必须是对当前Class文件中引导方法表的bootstrap_methods[]数组的有效索引</td>	
    </tr>
    <tr align="center">
    	<td>name_and_type_index</td><td>u2</td><td>值必须是对当前常量池的有效索引，常量池在该索引处的项必须是CONSTANT_NameAndType_info结构，表示方法名和方法描述符</td>
    </tr>
</table>

有兴趣的朋友可以分析一下剩余的16个常量，下面是我分析后的结果。

```bash
CA FE BA BE    魔数  
00 00  次版本号
00 33  主版本号（JDK1.7.0）
00 13  常量池18个常量
0A 第1个常量，10，方法引用
00 04 参数1，类描述符，指向第4个常量
00 0F 参数2，名称、类型描述符，指向第10个常量
09 第2个常量，9，字段引用
00 03 参数1，类或接口描述符，指向第3个常量
00 10 参数2，名称、类型描述符，指向第16个常量
07 第3个常量，7，类信息
00 11 参数1，类全限定名常量项索引值，指向第17个常量
07 第4个常量，7，类信息
00 12 参数1，类全限定名常量项索引值，指向第18个常量
01 第5个常量，UTF-8常量
00 01 参数1，常量字符串占用字节数，这里占用了1个字节
6D 参数2，常量值，6D表示字符串“m”
01 第6个常量，UTF-8常量
00 01 参数1，常量字符串占用字节数，这里占用了1个字节
49 参数2，常量值，49表示字符串“I”
01 第7个常量，UTF-8常量
00 06 参数1，常量字符串占用字节数，这里占用了6个字节
3C 69 6E 69 74 3E 参数2，常量值，代表字符串“<init>”
01 第8个常量，UTF-8常量
00 03 参数1，常量字符串占用字节数，这里占用了3个字节
28 29 56 参数2，常量值，代表字符串“()V”
01 第9个常量，UTF-8常量
00 04 参数1，常量字符串占用字节数，这里占用了4个字节
43 6F 64 65 参数2，常量值，代表字符串“Code”
01 第10个常量，UTF-8常量
00 0F 参数1，常量字符串占用字节数，这里占用了15个字节
4C 69 6E 65 4E 75 6D 62 65 72 54 61 62 6C 65 参数2，常量值，代表字符串“LineNumberTable”
01 第11个常量，UTF-8常量
00 03 参数1，常量字符串占用字节数，这里占用了3个字节
69 6E 63 参数2，常量值，代表字符串“inc”
01 第12个常量，UTF-8常量
00 03 参数1，常量字符串占用字节数，这里占用了3个字节
28 29 49 参数2，常量值，代表字符串“()I”
01 第13个常量，UTF-8常量
00 0A 参数1，常量字符串占用字节数，这里占用了10个字节
53 6F 75 72 63 65 46 69 6C 65 参数2，常量值，代表字符串“SourceFile”
01 第14个常量，UTF-8常量
00 0E 参数1，常量字符串占用字节数，这里占用了16个字节
43 6C 61 73 73 44 65 6D 6F 2E 6A 61 76 61 0C 00 参数2，常量值，代表字符串“ClassDemo.java”
07 第15个常量，7，类信息
00 08 参数1，指向全限定名常量项的索引，指向常量8，即“()V”
0C 第16个常量，12，名字类型信息
00 05 参数1，指向字段或方法名称常量项索引，指向常量5，即“m”
00 06 参数2，指向字段或方法描述符常量项索引，指向常量6，即“I”
01 第17个常量，UTF-8常量
00 11 参数1，常量字符串占用字节数，这里占用了17个字节
63 6F 6D 2F 6A 76 6D 2F 43 6C 61 73 73 44 65 6D 6F 参数2，常量值，代表字符串“com/jvm/ClassDemo”
01 第18个常量，UTF-8常量
00 10 参数1，常量字符串占用字节数，这里占用了16个字节
6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 65 63 74 参数2，常量值，代表字符串“java/lang/Object”
00 21 00 03 00 04 00 00 00 01 00 02 00 05 00 06 00 00 00 02 00 01 00 07 00 08 00 01 00 09 00 00 00 1D 00 01 00 01 00 00 00 05 2A B7 00 01 B1 00 00 00 01 00 0A 00 00 00 06 00 01 00 00 00 07 00 01 00 0B 00 0C 00 01 00 09 00 00 00 1F 00 02 00 01 00 00 00 07 2A B4 00 02 04 60 AC 00 00 00 01 00 0A 00 00 00 06 00 01 00 00 00 0C 00 01 00 0D 00 00 00 02 00 0E
```
