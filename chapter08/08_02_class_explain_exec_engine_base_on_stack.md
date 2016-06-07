<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [基于栈的字节码解释执行引擎](#%E5%9F%BA%E4%BA%8E%E6%A0%88%E7%9A%84%E5%AD%97%E8%8A%82%E7%A0%81%E8%A7%A3%E9%87%8A%E6%89%A7%E8%A1%8C%E5%BC%95%E6%93%8E)
  - [基于栈的指令集](#%E5%9F%BA%E4%BA%8E%E6%A0%88%E7%9A%84%E6%8C%87%E4%BB%A4%E9%9B%86)
  - [基于栈的解释器执行过程](#%E5%9F%BA%E4%BA%8E%E6%A0%88%E7%9A%84%E8%A7%A3%E9%87%8A%E5%99%A8%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 基于栈的字节码解释执行引擎

无论是解释执行还是编译执行，其都会经历如下图所示的各个步骤。

```pic
程序源码 -> 词法分析 -> 单词流 -> 语法分析
								  ↓
解释执行	<- 解释器 <-	指令流	<- 抽象语法书	
								  ↓
目标代码 <- 生成器 <- 中间代码 <- 优化器
```

## 基于栈的指令集

- 优点：可移植、代码相对更加紧凑、编译器实现更加简单。
- 缺点：执行速度相对较慢。

## 基于栈的解释器执行过程

