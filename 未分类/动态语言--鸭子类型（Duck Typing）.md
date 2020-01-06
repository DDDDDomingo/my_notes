# 动态语言--鸭子类型（Duck Typing）

在学习Go语言的接口上发现了一个很有意思的特性。



正在学习《自己动手写Java虚拟机》，刚好正在学习Golang，又可以更加深入的了解Jvm，真的是很不错的一本书，不过还是要先看下基本的语法。



场景：

当时定义了一个读取常量池数据的接口，并且给出了获取字段或者方法名称的名称和描述符的实现

```go
//接口
type ConstantInfo interface {
	readInfo(reader *ClassReader)
}

//实现类
type ConstantNameAndTypeInfo struct {
nameIndex uint16
descriptorIndex uint16
}
func (self *ConstantNameAndTypeInfo) readInfo(reader *ClassReader) {
self.nameIndex = reader.readUint16()
self.descriptorIndex = reader.readUint16()
}
```



接下来这段调用就让我看不懂了，稍微简化一下

```go
var cpInfo ConstantInfo = self[index]
var ntInfo *ConstantNameAndTypeInfo = cpInfo.(*ConstantNameAndTypeInfo)
```

如果从Java语言的角度出发，可以理解为获取接口的实现类对象，但是我在编写代码的过程中并没有显式地声明`ConstantNameAndTypeInfo`和`ConstantInfo`的关系，Go又是如何做到这一点的呢？



接下来就是这次的主角了，鸭子类型--**Duck Typing。**



由于了解有限就贴出一个给我解惑的链接地址，学有所成再写出自己的理解，本片做个学习笔记。

[Go语言接口与动态类型](http://c.biancheng.net/view/5120.html)

[浅入浅出 Go 语言接口的原理](https://www.infoq.cn/article/4xS7beawU7e4VSOjSYqE)



Golang还支持静态检查

目前除了不用显示声明