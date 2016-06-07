<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [字段表集合](#%E5%AD%97%E6%AE%B5%E8%A1%A8%E9%9B%86%E5%90%88)
  - [字段计数器](#%E5%AD%97%E6%AE%B5%E8%AE%A1%E6%95%B0%E5%99%A8)
  - [字段访问标识](#%E5%AD%97%E6%AE%B5%E8%AE%BF%E9%97%AE%E6%A0%87%E8%AF%86)
  - [字段名称索引项](#%E5%AD%97%E6%AE%B5%E5%90%8D%E7%A7%B0%E7%B4%A2%E5%BC%95%E9%A1%B9)
  - [字段描述符索引项](#%E5%AD%97%E6%AE%B5%E6%8F%8F%E8%BF%B0%E7%AC%A6%E7%B4%A2%E5%BC%95%E9%A1%B9)
  - [属性表计数器](#%E5%B1%9E%E6%80%A7%E8%A1%A8%E8%AE%A1%E6%95%B0%E5%99%A8)
  - [属性表](#%E5%B1%9E%E6%80%A7%E8%A1%A8)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 字段表集合

字段表（field_info）用于描述接口或者类中声明的变量。字段包括类级变量和实例级变量，但不包括在方法内部声明的局部变量。

类中的变量字段有许多需要描述的信息：字段的作用域（public、private、protected修饰符）、是实例变量还是类变量（static修饰符）、可变性（final）等，这些适合用布尔值表示。而字段名、字段类型这些无法固定，只能引用常量池中的常量来描述。

在类接口集合后的2个字节是一个字段计数器，表示总有有几个属性字段。在字段计数器后，才是具体的属性数据。其数据项如下图所示：

|类型|名称|数量|含义|
|:---:|:---:|:---:|:---:|
|u2|field_count|1|字段计数器|
|field_info|fields|field_count|字段数据|

其中field_info的表结构是：

|类型|名称|数量|含义|
|:---:|:---:|:---:|:---:|
|u2|access_flags|1|字段访问标识|
|u2|name_index|1|字段名称索引项|
|u2|descriptor_index|1|字段描述符索引项|
|u2|attributes_count|1|属性表计数器|
|attribute_info|attributes|attribute_count|属性表|

## 字段计数器

在类接口集合后的2个字节表示字段计数器。在ClassDemo类字节码文件中是0x0001，标识类总有有1个字段属性。

## 字段访问标识

在字段计数器后的2个字节表示字段访问标识，它与类中的access_flags项目是非常相似的，其中可以设置的标志位和含义见下表：

|标志名称|标志值|含义|
|:---:|:---:|:---:|
|ACC_PUBLIC|0x0001|是否public|
|ACC_PRIVATE|0x0002|是否private|
|ACC_PROTECTED|0x0004|字段是否protected|
|ACC_STATIC|0x0008|字段是否static|
|ACC_FINAL|0x0010|字段是否final|
|ACC_VOLATILE|0x0040|字段是否volatile|
|ACC_TRANSIENT|0x0080|字段是否transient|
|ACC_SYNTHETIC|0x1000|字段是否由编译器自动产生的|
|ACC_ENUM|0x4000|字段是否enum|

在ClassDemo类字节码文件中是0x0002，这表示其字段访问标识是private。

## 字段名称索引项

在字段访问名称后的2个字节表示字段名称索引项，其是一个指向了字符串常量（CONSTANT_Utf8_info）的索引。在ClassDemo类字节码文件中是0x0005，这表示其指向了常量池中的第5个常量，通过查询可知其字符串常量值是m。

## 字段描述符索引项

在字段名称索引项后的2个字节表示字段描述符索引项，与字段名称索引项一样，其也是一个指向了字符串常量（CONSTANT_Utf8_info）的索引。在ClassDemo类字节码文件中是0x0006，这表示其指向了常量池中的第6个常量，通过查询可知其字符串常量值是I。

解析到这里，我们已经得到了字段访问标识、字段名称值以及字段描述符值，那么这些值还原我们的字段声明呢？这就是我们下面要说的描述符的描述规则。

**描述符描述规则**

描述符描述规则由两部分组成：

- 描述符
- 描述规则

描述符描述了字段的数据类型，所有的描述符以及其含义如下表所示。

|标识字符|含义|
|:---:|:---:|
|B|基本类型byte|
|C|基本类型char|
|D|基本类型double|
|F|基本类型float|
|I|基本类型int|
|J|基本类型long|
|S|基本类型short|
|Z|基本类型boolean|
|V|特殊类型void|
|L|对象类型，如Ljava/lang/Object|

上面ClassDemo中的字段描述符为I，也即使基本数据类型int。

而对于不同类型数据，其描述规则也略有不同。

**描述数组类型时**，每一维度使用一个前置的“[”字符来描述，如一个定义为“java.lang.String[][]”类型的二位数组，将被定义为“[[Ljava/lang/String;”，一个整型数组“int[]”将被记录为“[I”。

**描述方法时**，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号“（）”之内。如方法void inc()的描述符为“()V”，方法java.lang.String toString()的描述符为“()Ljava/lang/String;”，方法int indexOf(char[] source, int sourceOffset,int sourceCount, char[] target, int targetOffset, int targetCount, int fromIndex)的描述符为“([CII[CIII)I”。

根据上述描述规则，我们把我们得出的访问标识凑起来就可以得到字段的整体描述——`private int m`。

## 属性表计数器

字段描述符索引项后2个字节标识属性表计数器，其中属性表可以用来描述存储一些额外的信息，字段可以在属性表中描述零项至多项的额外信息。对于本文汇总的字段m，它的属性表计数器为0，也就是没有额外的描述信息。

但是如果将字段m的声明改为“final static int m = 123;”，那就可能会存在一项名称为ConstantValue的属性，其值指向常量123。

## 属性表

属性表是具体存储字段额外属性的地方，这个我们将在介绍属性表的时候再进一步介绍。

```bash
00 01 属性字段数量
00 02 字段访问标识
00 05 字段名称访问索引项
00 06 字段描述符索引项
00 00 属性表计数器
```
