### 介绍
尽管 Go 编译器产生的是本地可执行代码，这些代码仍旧运行在 Go 的 runtime（这部分的代码可以在 runtime 包中找到）当中。
这个 runtime 类似 Java 和 .NET 语言所用到的虚拟机，它负责管理包括内存分配、垃圾回收（第 10.8 节）、栈处理、goroutine、channel、切片（slice）、map 和反射（reflection）等等。

runtime 调度器是个非常有用的东西，关于 runtime 包几个方法:

### 方法
|方法名|介绍|其他|
|:---|:---|:---|
|Gosched|让当前线程让出 cpu 以让其它线程运行,它不会挂起当前线程，因此当前线程未来会继续执行||
|NumCPU|返回当前系统的 CPU 核数量||

Gosched：让当前线程让出 cpu 以让其它线程运行,它不会挂起当前线程，因此当前线程未来会继续执行

NumCPU：返回当前系统的 CPU 核数量

GOMAXPROCS：设置最大的可同时使用的 CPU 核数

Goexit：退出当前 goroutine(但是defer语句会照常执行)

NumGoroutine：返回正在执行和排队的任务总数

GOOS：目标操作系统
