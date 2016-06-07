<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Java虚拟机发展史](#java%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%8F%91%E5%B1%95%E5%8F%B2)
  - [Sun Classic VM](#sun-classic-vm)
  - [Exact VM](#exact-vm)
  - [Sun HotSpot VM](#sun-hotspot-vm)
  - [BEA JRockit](#bea-jrockit)
  - [IBM J9 VM](#ibm-j9-vm)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Java虚拟机发展史

## Sun Classic VM

作为“世界上第一款商用 Java 虚拟机”，从 JDK1.0 到 JDK1.2 都讲 Sun Classic VM 作为其默认的 Java 虚拟机。

## Exact VM

Exact VM 是在 JDK1.2 时发布的一款运行在 Solaris 平台的虚拟机，但是到了 JDK1.3 其就不存在了。因为在 JDK1.3 之后，Sun 公司将 HotSpot 虚拟机作为 Java 语言默认的虚拟机。

## Sun HotSpot VM

其实一开始是由一家名为“Longview Technologies”的小公司开发，1999年的时候被 Sun 公司收购。在JDK1.3 中其作为附加额虚拟机中提供，在 JDK1.3 之后其作为默认的 Java 虚拟机，并且一直将其作为默认虚拟机沿用至今。

## BEA JRockit

BEA JRockit 是由 BEA 公司开发的，专门为服务器硬件和服务器端应用场景高度优化的虚拟机。

## IBM J9 VM

多用途虚拟机，其定位与 Sun HotSpot VM，IBM 的目标是将其应用于 IBM 产品上。