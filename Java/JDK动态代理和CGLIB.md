# JDK动态代理和CGLIB的区别

|          | JDK动态代理                                                  | CGLIB                                                        |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 介绍     |                                                              | Code Generator Library是一个强大的、高性能的代码生产库       |
| 原理     | 类加载顺序中解析和初始化可以互换<br/>1、为接口创建代理类的字节码文件<br />2、使用ClassLoader将字节码文件加载到JVM<br/>3、创建代理类实例对象，执行对象的目标方法 | CGLIB代理主要通过对字节码的操作，为对象引入间接级别，用以控制对象的访问。 |
| 涉及的类 | 涉及到的主要类：<br/>java.lang.reflect.Proxy;<br/>java.lang.reflect.InvocationHandler;<br/>java.lang.reflect.WeakCache;<br/>sun.misc.ProxyGenerator; |                                                              |
| 限制     | 代理类继承了Proxy类并且实现了要代理的接口，由于java不支持多继承，所以JDK动态代理不能代理类 | 1、因为CGlib是生成子类来实现AOP，所以无法支持final Class<br/>2、需要强制无参数构造函数 |
| 实现     | 1、重写了equals()、hashCode()、toString()<br/>2、有一个静态代码块，通过反射或者代理类的所有方法<br>3、通过invoke执行代理类中的目标方法doSomething() |                                                              |
|          |                                                              |                                                              |



