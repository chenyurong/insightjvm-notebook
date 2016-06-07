<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [属性表集合](#%E5%B1%9E%E6%80%A7%E8%A1%A8%E9%9B%86%E5%90%88)
  - [Code属性](#code%E5%B1%9E%E6%80%A7)
  - [00 00 异常表数量](#00-00-%E5%BC%82%E5%B8%B8%E8%A1%A8%E6%95%B0%E9%87%8F)
  - [Exceptions属性](#exceptions%E5%B1%9E%E6%80%A7)
  - [LineNumberTable属性](#linenumbertable%E5%B1%9E%E6%80%A7)
  - [LocalVariableTable属性](#localvariabletable%E5%B1%9E%E6%80%A7)
  - [SourceFile属性](#sourcefile%E5%B1%9E%E6%80%A7)
  - [ConstantValue属性](#constantvalue%E5%B1%9E%E6%80%A7)
  - [InnerClasses属性](#innerclasses%E5%B1%9E%E6%80%A7)
  - [Deprecated 及 Synthetic属性](#deprecated-%E5%8F%8A-synthetic%E5%B1%9E%E6%80%A7)
  - [StackMapTable 属性](#stackmaptable-%E5%B1%9E%E6%80%A7)
  - [Signature 属性](#signature-%E5%B1%9E%E6%80%A7)
  - [BootstrapMethods 属性](#bootstrapmethods-%E5%B1%9E%E6%80%A7)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 属性表集合

在Class文件中字段表、方法表都可以携带自己的属性表集合，用于描述某些场景专有的信息。

属性表有21项属性名称，每一项都代表不同的含义，一般属性表的前2个字节都会指向常量池一个CONSTANT_Utf8_info的字符串，这个字符串代表了属性名称。之后我们根据属性名称再去解析其属性数据结构。属性表的所有属性名称及其含义见下表。

|属性名称|使用位置|含义|
|:---:|:---:|:---:|
|||

虽然上面表中的每个属性都有其各自的数据结构，但其都需要通过一个u4的长度属性去说明属性值锁占用的位数。一个符合规则的属性表应该满足下表中所定义的结构。

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u1|info|attribute_length|

## Code属性

Java程序方法体中的代码经过javac编译器处理后，最终变为字节码指令存储在Code属性内。Code属性出现在方法表的属性集合之中，但并非所有的方法表都必须存在这个属性，譬如接口或抽象类中的方法就不存在Code属性。如果方法表有Code属性存在，那么它的结构将如下表所示。

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|max_stack|1|
|u2|max_locals|1|
|u4|code_length|1|
|u1|code|code_length|
|u2|exception_table_length|1|
|exception_info|exception_table|exception_table_length|
|u2|attributes_count|1|
|attribute_info|attributes|attributes_count|

- `attribute_name_index` 是一项指向CONSTANT_Utf8_info型常量的索引，常量值固定为“Code”，它代表了该属性的属性名称。
- `attribute_length` 指示了属性值的长度。
- `max_stack` 代表了操作数栈深度的最大值。在方法执行的任意时刻，操作数栈都不会超过这个深度。
- `max_locals` 代表了局部变量表所需的存储空间。在这里，max_locals的单位是Slot，Slot是虚拟机为局部变量分配内存所使用的最小单位。
- `code_length`和`code` 用于存储Java源程序编译后生成的字节码指令。
- `exception_table_length`和`exception_table`是这个方法的显式异常处理表集合，异常表对于Code属性来说并不是必须存在的。异常表的格式如下表所示，这些字段的含义为：如果字节码在第start_pc行到第end_pc行之间（不含end_pc行）出现了类型为catch_type或者其子类的异常（catch_type为指向一个CONSTANT_Class_info型常量的索引），则转到第handler_pc行继续处理。当catch_type的值为0时，代表任意异常情况都需要转向到handler_pc处进行处理。

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|start_pc|1|
|u2|end_pc|1|
|u2|handler_pc|1|
|u2|catch_pc|1|

在ClassDemo类的字节码文件中，方法表`<init>`后有一个属性表，其名称索引为0x0009，即指向了常量池中第9个常量，其值为Code。之后的0x00001D代表其属性长度为19。之后的0x0001代表其最大栈深度为1，0x0001代表其局部变量表所需要的存储空间为1Slot。

之后的4个字节代表字节码指令长度0x00000005，表示后面紧跟着的5歌字节是字节码指令。也就是说`2A B7 00 01 B1`是字节码指令，通过查询字节码指令表，可知其对应的字节码指令：

- 读入2A，查表得0x2A对应的指令为aload_0，这个指令的含义是将第0个Slot中为reference类型的本地变量推送到操作数栈顶。
- 读入B7，查表得0xB7对应的指令为invokespecial，这条指令的作用是以栈顶的reference类型的数据所指向的对象作为方法接收者，调用此对象的实例构造器方法、private方法或者它的父类的方法。这个方法有一个u2类型的参数说明具体调用哪一个方法，它指向常量池中的一个CONSTANT_Methodref_info类型常量，即此方法的方法符号引用。
- 读入00 01，这是invokespecial的参数，查常量池得0x0001对应的常量为实例构造器“<init>”方法的符号引用。
- 读入B1，查表得0xB1对应的指令为return，含义是返回此方法，并且返回值为void。这条指令执行后，当前方法结束。

到这里对于init方法的字节码分析结束，下面是对于字节码的分析，以及字节码对于的含义。

```bash
00 01 属性表计数器
00 09 第1个属性索引项
00 00 00 1D 属性长度
00 01 max_stack
00 01 max_locals
00 00 00 05 code_length 字节码指令长度
2A B7 00 01 B1 字节码指令
00 00 异常表数量
-------
00 01 属性数量
00 0A 属性名称索引项，LineNumberTable（Java源码的行号与字节码指令的对应关系）
00 00 00 06 属性长度，6个字节
00 01 行号表长度，1个字节
00 00 line_number——start_pc 字节码行号  第0行
00 07 line_number——line_number Java源码行号 第7行
```

下面我们再进行第二个方法的分析。

```
00 01 方法访问标识符（public）
00 0B 方法名称索引项，第11个常量（inc）
00 0C 方法描述符索引项，第12个常量（()I）
00 01 属性表计数器
00 09 第1个属性索引项，代表属性名称，查值得Code
00 00 00 1F 属性长度
00 02 max_stack
00 01 max_locals
00 00 00 07 code_length 字节码长度
2A B4 00 02 04 60 AC 字节码指令
00 00 异常表数量
```

```
00 01 00 0A 00 00 00 06 00 01 00 00 00 0C 00 01 00 0D 00 00 00 02 00 0E
```

## Exceptions属性

这里的Exceptions属性是在方法表中与Code属性平级的一项属性，它的作用是列举出方法中可能抛出的受查异常，也就是方法描述时在throws关键字后面列举的异常。

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|number_of_exceptions|1|
|u2|exception_index_table|number_of_exceptions|

number_of_exceptions表示方法可能抛出number_of_exceptions种受检异常，每一种受检异常使用一个exception_index_table项表示，exception_index_table是一个指向常量池中CONSTANT_Class_info型常量的索引，代表了该受查异常的类型。

## LineNumberTable属性

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|line_number_table_length|1|
|line_number_info|line_number_table|line_number_table_length|

## LocalVariableTable属性

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|line_variable_table_length|1|
|line_variable_info|line_variable_table|line_variable_table_length|

local_variable_info结构：

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|start_pc|1|
|u2|length|1|
|u2|name_index|1|
|u2|descriptor_index|1|
|u2|index|1|

## SourceFile属性

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|sourcefile_index|1|

## ConstantValue属性

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|constantvalue_index|1|

## InnerClasses属性

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|number_of_classes|1|
|inner_classes_info|inner_classes|number_of_classes|

inner_classes_info表结构：

|类型|名称|数量|
|:---:|:---:|:---:|
|u2|inner_class_info_index|1|
|u2|outer_class_info_index|1|
|u2|inner_name_index|1|
|u2|inner_class_access_flags|1|

inner_class_access_flags标志：

|标志名称|标志值|含义|
|:---:|:---:|:---:|
|ACC_PUBLIC|0x0001|内部类是否public|
|ACC_PRIVATE|0x0002|内部类是否private|
|ACC_PROTECTED|0x0004|内部类是否protected|
|ACC_STATIC|0x0008|内部类是否static|
|ACC_FINAL|0x0010|内部类是否final|
|ACC_INTERFACE|0x0020|内部类是否为借口|
|ACC_ABSTRACT|0x0400|内部类是否abstract|
|ACC_SYNTHETIC|0x1000|内部类是否并非由用户代码产生的|
|ACC_ANNOTATION|0x2000|内部类是否是一个注解|
|ACC_ENUM|0x4000|内部类是否是一个枚举|

## Deprecated 及 Synthetic属性

Deprecated 及 Synthetic 两个属性都属于标志类型的布尔属性，只存在有和没有的区别，没有属性值的概念。

Deprecated 属性用于表示某个类、字段或者方法，已经被程序作者定义为不再推荐使用，它可以在代码中通过注释`@Deprecated`进行设置。

Synthetic 属性代表此字段或方法不是由Java源码直接产生的。

|标志名称|标志值|含义|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|

其中 attribute_length 数据项的值必须为`0x00000000`，因为没有属性值需要设置。

## StackMapTable 属性

|标志名称|标志值|含义|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|number_of_entries|1|
|stack_map_frame|stack_map_frame entries|number_of_entries|

## Signature 属性

Signature 属性是一个可选的定长属性，用于记录泛型签名信息。之所以要设置这样一个属性去记录泛型类型，是因为Java语言的泛型采用的是擦除法实现的伪泛型，在字节码中，泛型信息编译之后都通通被擦除掉。下表是 Signature 属性的结构。

|标志名称|标志值|含义|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|signature_index|1|

其中 signature_indx 项的值必须是一个队常量池的有效索引，常量池在该索引处的项必须是 CONSTANT_Utf8_info 结构，表示签名、方法类型签名或字段类型签名。

如果当前 Signature 属性是**类文件的属性**，则这个结构表示类签名；如果当前的 Signature 属性是**方法表的属性**，则这个结构表示方法类型的签名；如果当前 Signature 属性是**字段表的属性**，则这个结构表示字段类型签名。

## BootstrapMethods 属性

BootstrapMethods 属性的结构：

|标志名称|标志值|含义|
|:---:|:---:|:---:|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|num_bootstrap_methods|1|
|bootstrap_method|bootstrap_methods|num_bootstrap_methods|

其中引用到的 bootstrap_method 结构见下表：

|标志名称|标志值|含义|
|:---:|:---:|:---:|
|u2|bootstrap_method_ref|1|
|u2|num_bootstrap_arguments|1|
|u2|bootstrap_arguments|num_bootstrap_arguments|
