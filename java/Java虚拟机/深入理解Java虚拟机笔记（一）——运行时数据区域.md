# 深入理解Java虚拟机笔记（一）——运行时数据区域

![运行时数据区](https://cdn.jsdelivr.net/gh/LoveLifeEveryday/FigureBed@master/typora202003/26/151354-661702.png)

## 1.程序计数器 
程序计数器（Program Counter Register）是一块较小的内存区域，可以看作当前线程执行的字节码的行号指示器。字节码解释器工作时就是通过改变该值来选取下一条需要执行的字节码指令，**是程序控制流的指示器，分支，循环，跳转，异常处理，线程恢复等基础操作都需要该计数器来完成。**  
因为Java虚拟机的多线程是通过多个线程轮流切换，分配处理器执行时间来实现的，在任何一个确定的时刻，一个处理器只会处理一条线程中的指令，所以，为了线程切换后能立刻恢复到正确的执行位置，**每条线程都需要一个独立的程序计数器，各个线程之间的计数器互不影响，独立存储。**

## 2.Java虚拟机栈
Java虚拟机栈（Virtual Machine Stack）也是私有的，它的生命周期与线程相同。虚拟机栈描述的是**Java方法执行的线程内存模型。** 每个方法被执行的时候，Java虚拟机都会同步创建一个**栈帧（Stack Frame）** 用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从被调用到执行完毕，就代表着一个栈帧在虚拟机栈中**入栈到出栈**的过程。  
若线程请求的栈深度大于虚拟机所允许的深度，会抛出 StackOverflowError异常；若Java虚拟机栈容量可以动态扩展，当栈扩展时无法申请到足够的内存会抛出OutOfMemoryError异常。

## 3.本地方法栈
本地方法栈（Native Method Stack）与虚拟机栈发挥的作用非常相似，区别只是虚拟机栈为虚拟机执行Java方法服务，而本地方法栈是为虚拟机使用本地（Native）方法服务。  
在HotSpot虚拟机中，直接将本地方法栈与虚拟机栈合二为一。  

## 4. Java堆
Java堆（Heap）是虚拟机管理的内存中最大的一块。堆是被所有线程共享的一块区域，在虚拟机启动时创建，此内存区域唯一目的是**存放对象实例**。Java世界里所有的对象实例都在这里分配内存。  
Java堆可以处于物理上不连续的内存空间中，但在逻辑上应被视为连续的，类似用磁盘空间去存储文件一样，并不要求每个文件都连续存放。但对于大对象（例如数组），多数虚拟机处于实现简单、存储高效的考虑，会要求连续的内存空间。  
Java堆既可以固定大小，也可以进行扩展。

## 5.方法区
方法区（Method Area）与Java堆一样，是线程共享的区域，用于存储已经被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。
在JDK8之前，方法区与**永久代**有一定关联，但在JDK8中，完全废弃了永久代，改为在本地内存中实现元空间（MetaSpace）来代替。  
方法区可以选择不实现垃圾回收，该区域的内存回收目标是针对常量池的回收和堆类型的卸载，但是回收条件非常苛刻，且效果难令人满意。  

## 6.运行时常量池
运行时常量池（Runtime Constant Pool）算是方法区的一部分。Class文件中常量池表用于存放编译期生成的各种字面量与符号引用，这部分内容在类加载后存放到方法区的运行时常量池。
运行时常量池相较于Class文件常量池的一个重要特征就是具备动态性。在运行期间也可以将新的常量放入池中。

## 7.直接内存
直接内存（Direct Memory）并不hi虚拟机运行时数据区的一部分，但这部分内存被频繁使用。  
在JDK1.4中新加入的NIO类，引入了一种基于通道与缓冲区的I/O方式。他可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆里的DirectByteBuffer对象作为这块内存的引用进行操作。这样可以在一些场景显著提升性能，因为避免了在Java堆和Native堆中来回复制数据。  
本机直接内存分配不会受到Java堆大小的限制，但会受到本机总内存大小及处理器寻址空间的限制。