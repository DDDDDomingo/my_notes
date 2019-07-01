# JVM——逃逸分析（Escape Analysis）

### 简介：

JIT全称是 `just in time` （即时编译技术），理论上能够加快Java程序的运行速度。

通常情况下，Java代码经由 `javac` 将程序源码编译成 .class 字节码