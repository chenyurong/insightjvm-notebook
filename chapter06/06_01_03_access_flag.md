# 访问标志

在常量池结束之后，紧接着的两个字节代表访问标记（access_flags），这个标志用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型等。具体的标志位以及标志的含义见下表。

|标志名称|标志值|含义|
|:---:|:---:|:---:|
|ACC_PUBLIC|0x0001|是否为public类型|
|ACC_FINAL|0x0010|是否被声明为final，只有类可设置|
|ACC_SUPER|0x0020|是否允许使用invokespecial字节码指令的新语义，invokespecial指令的语义在JDK1.2发生过改变，为了区别这条指令使用哪种语义，JDK1.2之后编译出来的类的这个标志都必须为真|
|ACC_INTERFACE|0x0200|标识这时一个接口|
|ACC_ABSTRACT|0x0400|是否为abstract类型，对于接口或抽象类来说，此标志值为真，其他类值为假|
|ACC_SYNTHETIC|0x1000|标识这个类并非由用户代码产生的|
|ACC_ANNOTATION|0x2000|标识这时一个注解|
|ACC_ENUM|0x4000|标识这时一个枚举|

access_flags中一共有16个标志位可以使用，当前只定义了其中8个，没有使用到的标志位要求一律为0。以前面的例子为例，ClassDemo是一个普通的Java类，不是接口、枚举或注解，被public关键字修饰但没有被声明为final何abstract，并且它使用了JDK1.2之后的编译器进行编译，因此其除了ACC_PUBLIC、ACC_SUPER标志应当为真，而其他的都为假，因此它的access_flags的值应为：0x0001|0x0020=0x0021。这与字节码文件的access_flags标志的确为0x0021。