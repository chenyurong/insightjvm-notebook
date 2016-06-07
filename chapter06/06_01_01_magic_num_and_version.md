<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [魔数与Class文件的版本](#%E9%AD%94%E6%95%B0%E4%B8%8Eclass%E6%96%87%E4%BB%B6%E7%9A%84%E7%89%88%E6%9C%AC)
  - [魔数](#%E9%AD%94%E6%95%B0)
  - [版本号](#%E7%89%88%E6%9C%AC%E5%8F%B7)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 魔数与Class文件的版本

## 魔数

每个 Class 文件的头 4 个字节称为魔数（Magic Number），它的唯一作用是确定这个文件是否为一个能被虚拟机接受的 Class 文件。Class 文件的魔数很有浪漫气息，其值是：**0xCAFEBABE**（咖啡宝贝）。

## 版本号

紧接着魔数的4个字节是 Class 文件的版本号：

- 第5、6个字节是：次版本号（Minor Version）
- 第7、8个字节是：主版本号（Major Version）

高版本的 JDK 能向下兼容以前版本的 Class 文件，但不能运行以后版本的 Class 文件。下图列出了JDK1.1 到 JDK1.7 主流 JDK 版本编译器输出的默认和可支持的 Class 文件版本号。

|编译器版本|-target参数|十六进制版本号|十进制版本号|
|:---:|:---:|:---:|:---:|
|JDK1.1.8  |不能带target参数| 00 03 00 2D  2D|  45.3 |
|JDK1.2.2  |不带（默认为-target1.1）  | 00 03 00 2D | 45.3 |
|JDK1.2.2  |-target1.2 | 00 00 00 2E | 46  |
|JDK1.3.1_19  |不带（默认为-target1.1）  | 00 03 00 2D | 45.3  |
|JDK1.3.1_19  |-target1.3| 00 00 00 2F | 47  |
|JDK1.4.2_10  |不带（默认为-target1.2）  | 00 00 00 2E |  46 |
|JDK1.4.2_10  |不带（默认为-target1.  | 00 00 00 30 | 48  |
|JDK1.5.0_11  |不带（默认为-target1.5）  | 00 00 00 31 |49|
|JDK1.5.0_11  |不带（默认为-target1.4 | 00 00 00 30 |48|
|JDK1.6.0_01  |不带（默认为-target1.6）  | 00 00 00 32 |50|
|JDK1.6.0_01  |-target 1.5| 00 00 00 31|49|
|JDK1.6.0_01  |-target 1.4 -source 1.4| 00 00 00 30|48|
|JDK1.7.0  |不带（默认为-target1.7）  | 00 00 00 33 |51|
|JDK1.7.0  |-target 1.6| 00 00 00 32 |50|
|JDK1.7.0  |-target 1.4 -source 1.4| 00 00 00 30 |48|

为了讲解方便，本章后面的内容都讲以下面这段小程序使用JDK1.7编译输出的 Class 文件作为基础来进行讲解。

```java
package com.jvm;

public class ClassDemo {

    private int m;

    public int inc() {
        return m + 1;
    }
}
```

下图是使用十六进制编辑器打开这个 Class 文件的结果，可以清楚地看到开头 4 个字节的十六进制是 0xCAFEBABE，代表次版本号的第5个和第6个字节为0x0000，而主版本号的值为0x0031，也即是十进制的50，该版本号说明这个文件是可以被JDK1.7或以上版本虚拟机执行的 Class 文件。

