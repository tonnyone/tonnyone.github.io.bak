---
title: 关于java常量池的理解
category: java 
tag: java
---
## 概要

最近刚学习了jvm的知识,关于java常量池书中只说到了jdk1.6的常量池模型,而且关于jdk1.7，1.8 网上也众说纷纭,之前面试也屡屡被问起,屡屡不懂,我们就来研究一下,闲话少说:

### jdk6

[引用自jdk6 jvm 说明书](https://docs.oracle.com/javase/specs/jvms/se6/html/Overview.doc.html#1732)
> A runtime constant pool is a per-class or per-interface runtime representation of the constant_pool table in a class file (§4.4). It contains several kinds of constants, ranging from numeric literals known at compile time to method and field references that must be resolved at run time. The runtime constant pool serves a function similar to that of a symbol table for a conventional programming language, although it contains a wider range of data than a typical symbol table.
Each runtime constant pool is allocated from the Java virtual machine's method area (§3.5.4). The runtime constant pool for a class or interface is constructed when the class or interface is created (§5.3) by the Java virtual machine.

> 运行时常量池是java**每一个**类(或接口)的类文件的一个常量表,包含多种类型,编译时的基本数据类型, 运行时的引用类型.运行时常量池提供的功能类似于传统编程语言的符号表，虽然它的数据范围比符号表更广。
> **每一个**运行时常量池都是从**jvm的方法区**分配的,当类(或接口)创建的时候,那么这个类(或接口)的常量池就会被创建.

### jdk7
> A run-time constant pool is a per-class or per-interface run-time representation of the constant_pool table in a class file (§4.4). It contains several kinds of constants, ranging from numeric literals known at compile-time to method and field references that must be resolved at run-time. The run-time constant pool serves a function similar to that of a symbol table for a conventional programming language, although it contains a wider range of data than a typical symbol table.
Each run-time constant pool is allocated from the Java Virtual Machine's method area (§2.5.4). The run-time constant pool for a class or interface is constructed when the class or interface is created (§5.3) by the Java Virtual Machine.

### 回顾一个jvm 类加载过程


#### load

找到具有特定名称的类或接口类型的二进制表示并从该二进制表示中创建类或接口

#### link

获取类或接口并将其组合到Java虚拟机的运行时的状态.

#### initializes

执行类或接口初始化方法组成

```java
```

## 参考资料

- [Java Language and Virtual Machine Specifications](https://docs.oracle.com/javase/specs/index.html)
- [stackoverflow purpose of constant pool](https://stackoverflow.com/questions/10209952/what-is-the-purpose-of-the-java-constant-pool)
- [java-string-pool](https://www.journaldev.com/797/what-is-java-string-pool)