CS61C

学着玩



### Lecture 02.2

相比之下 C语言是更加不想高级语言的高级语言(higher language)

它的程序描述语言 **刚好**超出了CPU的理解范围

 在运行的时候 他是通过编译 将c语言的代码 变为**汇编**语言(Assemble language)

在这一基础上 再次将其编程机器语言（机器码—— machine language）

这一过程称为**编译**



**优点**：



总所周知 Java的一大优点是 当操作系统等环境更换时，只需要有JDK即可运行

这一特点的实现主要是JVM的存在（Java virtual machine）

这一事物（环境）的存在可以将java代码 随时随地编写成为java当中的**字节码文件**

所有当我们说运行java项目的时候 并不直接跟CPU联系起来 而是借助JVM的架构上运行

而Python则是解释性语言 Python会将所写的代码进行解释 从而运行 也与CPU无直接关系

相比之下 C的编译过程和原理决定了 他是直接运行在CPU和hardware硬件上的

这也决定C的运行效率会更加快 一般情况下会比Py快很多

Typically下也会比Java快（原描述）



同时编译这一过程是针对文件而言的 所以可以进行公平的编译（fair compilation）

当某个文件需要修改的时候 不需要整个项目重新进行编译 仅需要rebuilt对应文件即可



但是C的这一编译的过程也会存在一定的

**缺点**：

- 在debug 运行调试等的时候 需要经过 **编辑 编译 运行** 三个步骤 而这一步骤是比较繁琐的
- 与Java相反的 编译和运行可执行文件 这一操作 是**基于当前的操作系统，体系结构**的 当操作系统或者运行环境改变的时候 需要重新编译 构建 (称之为 architecture-specific)



老生常谈的东西 温故而知新



### Lecture 02.3

计算机中使用数字表示万物

Ascii 128个字符（包括0） 规定了常用的字符和相关

128 = 2^8  （0~127）

算上0 本质上就是 从 0000000 到 1111111

所以一般都可以使用7bits来表示一个字符（64位中）

但是方便计算 计算机中会将其加多一位 变为8位

一char一byte



C为弱数据类型语言 （js）

因为c中 一切数据 到底层都是存储为位（bits）的 所以一般来说 可以随意切换数据类型 而没有影响

乐

![image-20240724171236024](C:\Users\chenz\OneDrive\桌面\note\else\images\image-20240724171236024.png)



结构体中的 数据对齐和填充 

![image-20240724171830059](C:\Users\chenz\OneDrive\桌面\note\else\images\image-20240724171830059.png)

每一个数据类型的长度不相同 但是c语言具有自动对齐填充的机制

在这一前提下 CPU执行操作会更快 



联合 Unions

![image-20240724172405455](C:\Users\chenz\OneDrive\桌面\note\else\images\image-20240724172405455.png)



两个参数 第一个说明参数的数量 第二个具体各个参数的值

![image-20240724173715704](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240724173715704.png)



先声明再调用

声明不用放垃圾

![image-20240724173630360](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240724173630360.png)



关于Bool

0 & Null位false  其他为true

![image-20240724173833495](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240724173833495.png)