# 方法表集合

在字段表后的2个字节是一个方法计数器，表示类中总有有几个方法。在字段计数器后，才是具体的方法数据。其数据项如下图所示：

|类型|名称|数量|含义|
|:---:|:---:|:---:|:---:|
|u2|method_count|1|方法计数器|
|method_info|methods|method_count|方法数据|

Class文件存储格式中对方法的描述与对字段的描述完全一致，方法表的结构如同字段表一样，依次包括了访问标识（access_flags）、名称索引（name_index）、描述符索引（descriptor_index）、属性表集合（attributes）。其中method_info的数据结构如下：

|类型|名称|数量|含义|
|:---:|:---:|:---:|:---:|
|u2|access_flags|1|方法访问标识|
|u2|name_index|1|方法名称索引项|
|u2|descriptor_index|1|方法描述符索引项|
|u2|attributes_count|1|属性表计数器|
|attribute_info|attributes|attribute_count|属性表|

方法表与字段表的不同点就是方法表的方法访问标识。因为volatile关键字和transient关键字不能修饰方法，所以方法表的访问标志中没有ACC_VOLATILE标志和ACC_TRANSIENT标志。对于方法表，所有标志位及其取值可参考下表。

|标志名称|标志值|含义|
|:---:|:---:|:---:|
|ACC_PUBLIC|0x0001|方法是否public|
|ACC_PRIVATE|0x0002|方法是否private|
|ACC_PROTECTED|0x0004|方法是否protected|
|ACC_STATIC|0x0008|方法是否static|
|ACC_FINAL|0x0010|方法是否final|
|ACC_SYNCHRONIZED|0x0020|方法是否为synchronized|
|ACC_BRIDGE|0x0040|方法是否是由编译器产生的桥接方法|
|ACC_VARARGS|0x0080|方法是否接受不定参数|
|ACC_NATIVE|0x0100|方法是否为native|
|ACC_ABSTRACT|0x0400|方法是否为abstract|
|ACC_STRICTFP|0x0800|方法是否为strictfp|
|ACC_SYNTHETIC|0x1000|方法是否是由编译器自动产生|

在ClassDemo字节码文件中，字段表后的两个字节是0x0002，这表示该类有两个方法。我们先来看第一个方法，紧跟着的两个字节是0x0001，表示其方法访问标识是public。之后两个字节是0x0007，表示其方法名称指向了常量池中第7个常量，查询可知其值为`<init>`。之后两个字节是0x0008，表示其方法描述符指向了常量池第8个常量，查询可知其值为`()V`。按照上节对于字段描述符的描述规则，我们可以还原出其方法的语法描述：`public void <init>();`。

紧跟其后的2个字节0x0001，表示此方法的属性表集合有一项属性，属性名称索引为0x0009，指向了常量池第9个常量，查询可知其值为`Code`，说明此属性是方法的字节码描述。

```bash
00 02 方法计数器
00 01 方法访问标识（public）
00 07 方法名称索引项（<init>）
00 08 方法描述符索引项（）
00 01 属性表计数器
00 09 第1个属性索引项，代表属性名称，查值得Code
```
