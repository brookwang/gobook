Go语言(Golang)最初是由Robert Griesemer, Rob Pike, 和 Ken Thompson在谷歌于2007年开发出来的编程语言。

Go编程语言是静态类型语言，语法类似于C语言，它提供了垃圾收集，类型安全，动态的输入能力，还有很多先进的内置类型，例如，可变长度数组和映射（键-值对）。它还提供了丰富的标准库。

Go编程语言是在2009年11月正式对外发布，主要是应用谷歌的于一些生产系统链中。

设计原理
* 支持环境采取的模式类似于动态语言。例如：类型推断（x := 0是有效的int类型变量x的声明）
* 编译时快
* 内置的并发支持：轻量进程（通过goroutines），通道，select语句。
* 简炼，简单和安全
* 支持的接口类型和嵌入
* 产生没有外部的依赖静态链接的本机二进制文件

特点
* 为了保持语言的简洁和简单，按照类似的语言省略常用的功能。
* 不支持类型继承
* 不支持任何方法或运算符重载
* 不支付包之间循环依赖
* 不支持对指针运算
* 不支持断言
* 不支持泛型编程

