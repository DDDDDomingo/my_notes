# JVM——逃逸分析（Escape Analysis）

### 简介：

JIT全称是 `just in time` （即时编译技术），理论上能够加快Java程序的运行速度。

通常情况下，Java代码经由 `javac` 将程序源码编译成 `.class` 字节码，再由JVM一行一行解析字节码，翻译成相应的机器指令，其执行速度会比机器码慢很多。

JIT会将使用频率很高的字节码部分直接编译成机器码并保存下来，这样能够加快Java程序的运行速度。



### 工作原理图：

![](E:\my_notes\JVM\images\JIT工作原理图.png)

### 编译器调优：