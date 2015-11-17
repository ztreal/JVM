# 提问

#### 为什么使用Native Memory？有哪些使用例子？


#### JVM都有永久代么？
HotSpot方法区在永久代中，1.7中永久代的字符串常量池已经移除
对于其他虚拟机（如BEA
JRockit、IBM J9等）来说是不存在永久代的概念的。即使是HotSpot虚拟机本身，根据官方发布的路线图信息，现在也有放弃永久代并“搬家”至Native
Memory来实现方法区的规划了。

#### 什么是Nativ Memory/Direct Memory受什么限制？
Native Memory受到操作系统virtual process size限制，例如Xms为1G，系统可用总共3G，则Native Memory可能使用接近2GB空间。
调整-XX:MaxDirectMemorySize
http://www.techpaste.com/2012/07/steps-debugdiagnose-memory-memory-leaks-jvm/


#### Native Memory和heap访问哪个快？
http://mentablog.soliveirajr.com/2012/11/which-one-is-faster-java-heap-or-native-memory/
#####拓展问题
nginx怎么处理内存操作的，效率会那么高

#### Single Procss Maximum Possible Memory是多少？
系统和CPU级别:http://stackoverflow.com/questions/8799481/single-process-maximum-possible-memory-in-x64-linux
系统级别
http://shortrecipes.blogspot.de/2009/04/limitsconf-virtual-memory-limit.html

#### jvm启动加参数使System.gc()无效的话会导致什么问题？
XX:+DisableExplicitGC会导致Bits.reserveMemory无效，造成Native Memory OOM



#### 并行和并发概念区别解是什么？

答：并行和并发这两个概念都是编程中的概念，在垃圾收集器的上下文语境中，我们应该这样理
	（1）并行（Parallel）：指多条垃圾收集线程并行工作，但此此时用户线程仍然处于等待状态。
	（2）并发（Concurrent）：指用户线程与垃圾收集线程同时执执行（但不一定是并行的，可能会交替执行），用户程序继续运行，而垃圾收集程序运行于另一个CPU上。

#### 栈帧和方法去运行时常量池的关联？
可以通过

#### 栈帧是相互独立的么
栈帧之间直接可以有重叠，前后两个栈帧之间可以共享一部分区域用来传递参数

#### 栈帧是一个线程一个么？
Java 虚拟机是基于栈架构设计的,它的大多数操作是从当前栈帧的操作数栈取出 1 个或多个 操作数,或将结果压入操作数栈中。每调用一个方法,都会创建一个新的栈帧,并创建对应方法所 需的操作数栈和局部变量表(参见§2.6 “栈帧”)。每条线程在运行时的任意时刻,都会包󿰄若 干个由不同方法嵌套调用而产生的栈帧,当然也包括了若干个栈帧内部的操作数栈,但是只有当前 栈帧中的操作数栈才是活动的。
