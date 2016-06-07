<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [常见的类加载构造器](#%E5%B8%B8%E8%A7%81%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9E%84%E9%80%A0%E5%99%A8)
  - [Tomcat：正统的类加载器构造](#tomcat%EF%BC%9A%E6%AD%A3%E7%BB%9F%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8%E6%9E%84%E9%80%A0)
  - [OSGi：灵活的类加载器架构](#osgi%EF%BC%9A%E7%81%B5%E6%B4%BB%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8%E6%9E%B6%E6%9E%84)
  - [字节码生成技术与动态代理的实现](#%E5%AD%97%E8%8A%82%E7%A0%81%E7%94%9F%E6%88%90%E6%8A%80%E6%9C%AF%E4%B8%8E%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%E7%9A%84%E5%AE%9E%E7%8E%B0)
  - [Retrotranslator：跨越JDK版本](#retrotranslator%EF%BC%9A%E8%B7%A8%E8%B6%8Ajdk%E7%89%88%E6%9C%AC)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 常见的类加载构造器

## Tomcat：正统的类加载器构造

- `CommonClassLoader`：对应tomcat的/common目录，对于Tomcat和所有Web应用可见
- `CatalinaClassLoader`：对应tomcat的/server目录，对于Tomcat可见，但对所有Web不可见
- `SharedClassLoader`：对应tomcat的/shared目录，对于Tomcat不可见，对于所有Web可见
- `WebAppClassLoader`：对应于每个应用的/webapp/WEB-INF目录，对于该Web应用可见。
- `JsperLoader`：对于单个JSP文件可见，其出现的目的是为了实现热替换。当有新的JSP文件时，会产生新的JSP实例来替换到原来的。

但是在Tomcat6.X版本中，其将/common、/server、/shared三个目录合并起来变成了一个/lib目录。因为默认情况下CatalinaClassLoader和SharedClassLoader是不开启的，所以/server、/shared目录也就没什么意义。

## OSGi：灵活的类加载器架构

与双亲委派模型树型结构不同的是，OSGi采用了图这种更加灵活的加载模型。假设存在 BundleA、BundleB、BundleC三个模块，并且这三个Bundle定义的依赖关系如下。

- BundleA：声明发布了packageA，依赖了java.*的包。
- BundleB：声明依赖了packageA和packageC，同时也依赖了java.*的包。
- BundleC：声明发布了packageC，依赖了packageA。

那么，这三个Bundle之间的类加载器及父类加载器之间的关系如下图所示：

![](../img/09/09_01_01_osgi_artitecture.png)

在这种加载器架构下，所有依赖BundleA的包，其依赖的BundleA包将由发布该Bundle的包进行加载管理。

## 字节码生成技术与动态代理的实现


## Retrotranslator：跨越JDK版本

Retrotranslator是一个“Java逆向移植”工具，其可以将JDK1.5中编写的代码放到JDK1.4或1.3的环境中去部署使用。


