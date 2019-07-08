# 测试环境Jvm参数

`-XX:+DisableExplicitGC` 						//禁止代码中显示调用GC
`-XX:+HeapDumpOnOutOfMemoryError` 	//JVM在出现内存溢出时候Dump出当前的内存转储快照
`-XX:HeapDumpPath=/data/bts-test/orderservice/gclogs/`  //指定dump文件的位置
`-XX:InitialHeapSize=268435456`		//初始化堆大小	
`-XX:MaxHeapSize=536870912` 				//最大堆大小
`-XX:+PrintGC` 											//开启简单GC日志
`-XX:+PrintGCDateStamps` 						//记录的是系统时间

`-XX:+PrintGCTimeStamps` 						//记录的是Jvm启动时间为起点的相对时间

`-XX:+PrintGCDetails`
`-XX:+PrintGCTimeStamps` 						//打印GC发生的时间戳

-`XX:SurvivorRatio=10` 							//8表示两个survivor:Eden = 2:8，即每个survivor占1/10
`-XX:+UseCompressedClassPointers` 	//压缩指针，

`-XX:+UseCompressedOops` 						//压缩指针
`-XX:+UseG1GC` 											//启动G1

# 开发环境Jvm参数（基本一致，无变化）