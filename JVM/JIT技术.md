# JIT技术

### JIT简介：

JIT全称是 `just in time` （即时编译技术），理论上能够加快Java程序的运行速度。

通常情况下，Java代码经由 `javac` 将程序源码编译成 `.class` 字节码，再由JVM一行一行解析字节码，翻译成相应的机器指令，其执行速度会比机器码慢很多。

~~JIT会将使用频率很高的字节码部分直接编译成机器码并保存下来，这样能够加快Java程序的运行速度。~~

JIT是动态编译的一种表示形式，允许自适应优化，如**动态重新编译（dynamic recompilation）**和**微架构特定的加速（microarchitecture-specific speedups）**。在运行时，实现了JIT编译器的系统通常**连续分析**正在执行的代码，并从正在执行的代码中选出被**频繁重复执行**的部分，将其转换成机器代码，理论上来说，JIT编译可以比静态编译的执行速度更快（某些情况下程序运行速度并不能更快，甚至可能更慢），然后在不需要重新编译的情况下进行**缓存和重用**。

解释和JIT编译特别适用于动态编程语言，因为系统可以处理后期绑定数据类型并实施安全保证。



#### 1、JIT的优缺点：

JIT编译结合了两种传统机器代码翻译方法（提前编译AOT和解释）。

他具有两者的优点和缺点：

- JIT编译结合了编译代码的**速度**和解释的**灵活性**
- JIT编译不仅拥有解释器的**开销**，还拥有编译的**额外开销**



#### 2、使用JIT可能优于静态编译器的原因：

系统能够收集有关程序在其所处环境中的实际运行的统计信息，并且可以重新排列

- JIT编译可以针对目标CPU和运行应用程序的操作系统模型进行优化。
- 能够收集程序在其所处环境中实际运行的统计信息，并且可以重新排列和重新编译以获取最佳性能。但是一些静态编译器也可以将配置文件信息作为输入。
- 在进行全局内联替换时，静态编译过程可能需要运行时检查，确保对象的实际类重写了内联方法，否则可能发生虚拟调用，或者可能需要在循环中处理对数组访问的边界条件检查。而通过即时编译，可以将这种处理移出循环，通常可以大大提高速度。（Java会把数组越界检查的字节码插入.class，后面会将其移除）
- 字节码系统可以容易地重新安排执行的代码以获得更好的缓存利用率，虽然这可以通过静态编译的垃圾收集实现。



### HotSpot中JIT工作流程图：

<div align="center"> <img src="./images/JIT_Schematic.png"> </div><br>

### 启动延迟问题：

由于加载和编译字节码所花费的时间，JIT在应用程序的初始执行中会有轻微的明显延迟，这种延迟就称为“启动时间延迟”或“预热时间”。

一般情况下，JIT执行的优化越多，它将生成的代码越好，但初始延迟也会增加。因此，JIT必须在编译时间和生成的代码质量之间进行权衡，除了JIT编译之外，启动时间可以增加IO绑定操作，例如：JVM



> IBM Develop文章
>
> <https://www.ibm.com/developerworks/cn/java/j-lo-just-in-time/index.html>
>
> https://www.ibm.com/support/knowledgecenter/SSB23S_1.1.0.14/com.ibm.java.lnx.80.doc/diag/understanding/jit.html
>
> Wikipedia百科
>
> <https://en.wikipedia.org/wiki/Just-in-time_compilation>
>
> CSDN博客文章
>
> <https://blog.csdn.net/zhaohong_bo/article/details/89421055>
>
> 简书
>
> <https://www.jianshu.com/p/fbced5b34eff>