<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [类文件结构](#%E7%B1%BB%E6%96%87%E4%BB%B6%E7%BB%93%E6%9E%84)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 类文件结构

Class 类文件实现了系统无关性，只要这个 Class 文件可以被虚拟机解析，那么无论 Class 的来源是何种语言都可以。

Class 文件是一组以 8 位字节为基础的二进制流，各数据项目严格按照顺序紧凑地排列在 Class 文件之中，中间没有添加任何分隔符，这使得整个 Class 文件中存储的内容几乎全都是程序需要的数据，没有空隙存在。

Class 文件格式采用类似于 C 语言结构的伪结构来存储数据，这种伪结构中只有两种数据类型：无符号数和表。

**无符号数**

无符号数属于最基本的数据类型，以 u1、u2、u4、u8 六七分别代表 1 个字节、2 个字节、4 个字节、8 个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者按照 UTF-8 编码构成的字符串值。

**表**

表是由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性地以“_info”结尾。表用于描述有层次关系的复合结构的数据，整个 Class 文件本质上就是一张表，它由表下表所示的数据项构成。

|类型|名称|解释|数量|
|:---:|:---:|:---:|:---:|
|u4|magic|魔数|1|
|u2|minor_version|次版本号|1|
|u2|major_version|主版本号|1|
|u2|constant_pool_count|常量池常量个数|1|
|cp_info|constant_pool|常量池|constant_pool_count - 1|
|u2|access_flags|访问标记|1|
|u2|this_class|类索引|1|
|u2|super_class|父类索引|1|
|u2|interfaces_count|接口索引数量|1|
|u2|interfaces|接口内容|interfaces_count|
|u2|field_count|字段表字段数量|1|
|field_info|fields|字段表|field_count|
|u2|methods_count|方法表方法数量|1|
|method_info|methods|方法表|methods_count|
|u2|attributes_count|属性表属性数量|1|
|attribute_info|attributes|属性表|attributes_count|