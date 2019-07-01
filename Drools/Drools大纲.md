## Drools大纲

### Docker使用

1. 开发人员编写，编程drl文件，规则打包成jar包，部署到workbench,或者kie-server；
2. 业务人员配置，规则可配置，把规则以字符串形式保存到数据库；

> 频繁变动建议采用字符串形式，对数据库进行增删改查



### Drools文件

可以分为多种方式：*.drl文件、*.xml文件、.xls文件、.xlsx文件

#### .drl文件结构介绍

hello.drl（规则文件名）

```
package rules.testwrod
	rule "test001"
		when
			eval(true)	//LHS部分
		then
			System.out.println("hello world!");	//RHS部分
	end
```



基础语法分为三块内容：

- 包路径

  按照Java规范编写路径

- 引用

  - 导入类	  import xx.xx.xxxx;
  - 导入方法  import function xx.xx.xxxx;

- 规则体

  - LHS：when与then的中间部分，可以包含0-n个条件
    - 如果LHS部分没空的话，自动添加一个eval(true)条件，即总是true）
    - LHS中就是用来放置条件的
  - RHS：then后面的部分
    - 当LHS部分都满足时，RHS部分才会执行
    - RHS中尽量不要有条件判断
    - 可以使用LHS中定义的绑定变量名、设置的全局变量、Java代码
      - RHS中提供了对当前Working Memory实现快速操作的宏函数或对象
        - drools——可以使用更多的操作WM的方法
        - kcontext——可以通过该对象直接访问WM中的KnowledgeRuntime
  - 属性：

  最简单的规则至少要包括“包路径”，“规则体”两部分。



demo2.drl

```
package rules.testwrod
import cn.test.Person;
	rule "test002"
		when
			$p:Person();
		then
			System.out.println("hello "+$p.getName());
	end
		
```



### Drools的API调用

API总体分为三类：

- 规则编译
- 规则收集
- 规则的执行

具体步骤

1. Kmodule.xml的编辑
2. pom.xml导入相应的依赖
3. 编写.drl文件、KieSession

> 执行同一个drl文件里的多条规则，可以只用update(X)方法；
>
> 这样fact对象没有真正改变，只是引用发生了改变。当fact对象真正改变时，规则将重新执行，容易产生死循环；
>
> 解决方案再rule的属性中有说明



### Drools7版本关于session的不同方法

KieSession用于与规则引擎进行交互的会话。

- 有状态的KieSession

会在多次与规则引擎进行交互中维持会话的状态

<!--stateful是type的默认状态-->

```xml
<ksession name="all-rules" type="stateful"/>
```

获取KieSession实例通过以下语句

```java
KieSession kieSession = kieContainer.newKieSession("all-rules");
```



- 无状态的StatelessKieSession

隔离了每次与规则引擎的交互，不会维护会话的状态

<!--stateless-->

```xml
<ksession name="all-rules" type="stateful"/>
```

获取StatelessKieSession实例通过以下语句

```java
StatelessKieSession kieSession = kieContainer.newStatelessKieSession("all-rules");
```



### Drools内部功能详细介绍

#### 规则文件

```java
//包名（必须），逻辑上的管理，若自定义查询或者函数属于同一包名（即使物理位置不同），都可以调用
package com.rules.demo;
//需要导入的类名
import
//全局变量
global
//函数
function
//查询
query
//规则，可以有多个
rule
```

#### 规则语言

```java
rule "name"
    //可选
    attributes
    when
    	//详细见上方
        LHS
    then
    	//详细见上方
        RHS
end
```

<!--LHS部分补充-->

LHS部分是由一个或多个条件组成，条件又称之为pattern（匹配模式），多个pattern之间用and或or来进行连接，同时还可以使用小括号来确定pattern的优先级。

pattern没有符号连接，在Drools当中就用and来作为默认连接，都满足才会返回true，每行可以用;作为结束符，也可以不加

#### 约束连接

<!--约束连接-->

| 符号 | 含义 | 使用                                               |
| ---- | ---- | -------------------------------------------------- |
| &&   | and  |                                                    |
| \|\| | or   |                                                    |
| ,    | and  | 当有”&&“和“\|\|”出现的LHS中，不可以有","，反之亦然 |

<!--类型比较操作符-->

除了常见的>、<、≥、≤、==、!=之外，还有

| 操作符       | 含义                                                         | 语法                                                   |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| contains     | 检查一个Fact对象的某个字段（Collection/Array类型的对象）包含/不包含指定的对象 | Object(field[Collection/Array] contains value)         |
| not contains | 检查一个Fact对象的某个字段（Collection/Array类型的对象）不包含指定的对象 | Object(field[Collection/Array] not contains value)     |
| memberOf     | 判断某个Fact对象的某个字段是否存在一个集合（Collection/Array）当中 | Object(fieldName memberOf value[Collection/Array])     |
| not memberOf | 判断某个Fact对象的某个字段是否不存在一个集合（Collection/Array）当中 | Object(fieldName not memberOf value[Collection/Array]) |
| matches      | 是用来对某个Fact的字段与标准的Java正则表达式进行相似匹配（无需考虑“\”转义问题） | Object(fieldName matches "正则表达式")                 |
| not matches  | 是用来对某个Fact的字段与标准的Java正则表达式进行不相似匹配（无需考虑“\”转义问题） | Object(fieldName not matches "正则表达式")             |

> 比较项一定要是一个变量（绑定变量或者是global对象）

> 关于global对象
>
> **xx.drl**
>
> global java.util.List list;
>
> **xx.java**
>
> List list = new ArrayList();
>
> list.add(xxx);
>
> ...
>
> kieSession.setGlobal("list",list);

#### 语法拓展部分

访问List数据结构

$customer.accounts[3]等同于$customer.getAccounts(3)

访问Map数据结构

$customerMap["123"]等同于$customerMap.get["123"]



### Drools属性说明

规则的属性一共有12个

<!--简单介绍-->

| 属性名           | 作用                               | 取值                                                 | 使用范例                      |
| ---------------- | ---------------------------------- | ---------------------------------------------------- | ----------------------------- |
| salience         | 优先级                             | 默认情况0，可以是负数，越大执行优先级越高            | salience 1                    |
| no-loop          | 防止死循环                         | true（如果两个rule之间构成循环也无法阻止死循环）     | np-loop true                  |
| date-effective   | 系统日期大于设置值时执行           | 日期格式为“dd-MM-yyyy”的字符串，中英文操作系统有区别 | date-effective "50-二月-2017" |
| date-expires     | 系统日期小于设置值时执行           | 日期格式为“dd-MM-yyyy”的字符串，中英文操作系统有区别 | date-expires "50-二月-2017"   |
| dialect          | 定义规则中要使用的语言类型         |                                                      | dialect "mvel" dialect "java" |
| enabled          | 是否可用                           | 定义一个规则是否可用，值是一个布尔值                 | enabled true enabled false    |
| duration         | 在指定的值之后在另外一个线程里触发 | 单位毫秒                                             | duration 1000                 |
| lock-on-active   | 规则只执行一次                     |                                                      | lock-on-active true           |
| activation-group | 分组名称                           |                                                      |                               |
| agenda-group     | 议程分组                           |                                                      |                               |
| auto-focus       | 焦点分组                           |                                                      |                               |
| ruleflow-group   | 规则流                             |                                                      |                               |

> Tips：以下所有属性都是在rules范围内才可以生效的

#### 1.slience优先级

设置规则执行的优先级，属性的值是一个数字，数字越大执行优先级越高，同时值可以是一个负数。默认情况下，规则的salience默认值为0，所以如果我们不手动设置该属性，执行则随机。



#### 2.no-loop防止死循环

避免Fact修改或调用了insert、restart、update之类而导致规则再次激活执行

**死循环问题场景**：

某个Fact对象进行了修改，比如使用update将其更新到当前的WM当中，这时引擎会再次检查所有的规则是否满足条件，如果满足会再次执行，可能出现死循环。

只要将no-loop设置为true，就表示该规则只会被引擎检查一次，满足条件就执行规则的RHS部分。

> Tips：如果引擎内部因为对Fact更新引起引擎再次启动检查规则，那么它会忽略掉所有的no-loop属性设置为true的规则。
>
> 这可能会产生no-loop也无法解决的死循环，如rule1的LHS刚好是rule2的RHS执行结果，rule2的LHS刚好是rule1的执行结果，且rule1和rule2的RHS中都有update()方法。



#### 3.date-effective日期比较小于等于

该属性控制规则只有在到达后才会触发，在规则运行时，引擎会自动拿当前操作系统的时间进行比对，只有当前**操作系统时间**≥date-effective设置的时间才会触发。

> 该属性的值可以不按照正确的日期，比如“50-二月-2017”



#### 4.date-expires日期比较大于

该属性的作用与date-effective恰恰相反，当**操作系统时间**≤date-effective才能执行

> 可以用System.setProperty("drools.dateformat", "yyyy-MM-dd");修改日期格式，必须要写在初始化kie相关代码之前（静态代码块）



#### 5.Dialect方言

该属性用来定义规则当中要使用的语言类型，如果没有手工设置规则的dialect，那么使用java语言。

> Tips：特殊情况accumulate下会用到



#### 6.Enabled是否可用

enabled定义一个规则是否可用，该属性的值是一个布尔值，默认该属性的值为true，表示规则是可用的，如果enabled设置为false，那么引擎就不会执行该规则。



#### 7.Duration定时器（被淘汰）





#### 8.lock-on-active规则只执行一次

no-loop的增强版属性，避免Fact修改或调用了insert、restart、update之类而导致规则再次激活执行。

主要作用在使用ruleflow-group属性或agenda-group属性

设置为true后，该规则只被执行一次



#### 9.activation-group分组

具有相同activation-group属性的规划中只有一个会被执行，其他的规则都不再执行

可以用salience之类的属性来实现



#### 10.agenda-group议程分组

规则的调用与执行通过`StatelessSession`或`ksession`来实现的。

1. 执行顺序创建一个`StatelessSession`或`ksession`将各种经过编译的规则package添加到session中
2. 将规则中可能用到的Global对象和Fact对象插入到Session当中
3. 调用fireAllRules方法来触发、执行规则
4. 



#### 11.auto-focus焦点分组



#### 12.ruleflow-group规则流

作用是用来将规则划分为一个个的组，然后在规则流中通过使用ruleflow-group属性的值，从而使用对应的规则，**该属性会通过流程的走向确定要执行哪一条规则**。