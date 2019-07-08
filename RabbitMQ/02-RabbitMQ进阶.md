### 消息何去何从

------

```java
void basicPublish(String exchange, String routingKey, boolean mandatory, boolean immediate, BasicProperties props, byte[] body) throws IOException;
```

<!--mandatory和immediate参数详解-->

| 参数类型 | 参数名称  | 含义   | 可选值                                                       |
| -------- | --------- | ------ | ------------------------------------------------------------ |
| boolean  | mandatory | 强制性 | true--交换器无法根据自动的类型和路由键找到一个符合条件的队列，那么RabbitMQ会调用Basic.Return命令将消息返回给生产者；false--出现异常直接丢弃 |
| boolean  | immediate |        | true--如果交换器在消息路由到队列时发现没有任何消费者，那么这个消息将不会存到队列，会调用Basic.Return返回给生产者； |




#### mandatory参数

------

值为true时，交换器无法根据自动的类型和路由键找到一个符合条件的队列，那么RabbitMQ会调用Basic.Return命令将消息返回给生产者；

值为false时，交换器无法根据自动的类型和路由键找到一个符合条件的队列，直接丢弃；

<u>**Q：生产者如何获取到没有被正确路由到合适队列的消息呢？**</u>

A：通过调用`channel.addReturnListener`来添加`ReturnListener`监听器来实现。




#### immediate参数

------

值为true时，如果交换器在消息路由到队列时发现没有任何消费者，那么这个消息将不会存到队列，当与路由键匹配到的所有队列都没有消费者时，该消息会通过Basic.Return返回给生产者；

值为false时，... ... ；

> 概括来说，mandatory参数告诉服务器至少将消息路由到一个队列中，否则将消息返回给生产者。
>
> immediate参数告诉服务器，如果将消息关联的队列上有消费者，则立刻投递；如果所有匹配的队列上都没有消费者，则直接将消息返还给生产者，不用将消息存入队列而等待消费者了。

> RabbitMQ 3.0 版本开始去掉了对immediate参数的支持，RabbitMQ的官方解释是：该参数会影响镜像队列的性能，增加了代码复杂性，建议采用TTL和DLX的方法替代。




#### 备份交换器（Alternate Exchange）

------

对应于mandatory参数设置的情况（true使得生产者逻辑变得复杂、false使得消息丢失），这种情况下可以使用备份交互器，这样可以将未被路由的消息存储再RabbitMQ中，再在需要的时候去处理这些消息。

##### 使用

在申明交换器 (调用`channel.exchangeDeclare(...)`方法) 的时候添加 alternate-exchange 参数来实现，也可以通过策略 (policy) 的方式实现。同时使用的话，前者的优先级更高，会覆盖掉Policy的设置。

##### 区别

| 备份交换器                    | 客户端     | RabbitMQ服务端 | 消息              |
| ----------------------------- | ---------- | -------------- | ----------------- |
| 备份交换器不存在              | 无异常出现 | 无异常出现     | 丢失              |
| 备份交换器没有绑定任何队列    | 无异常出现 | 无异常出现     | 丢失              |
| 备份交换器没有任何匹配的队列  | 无异常出现 | 无异常出现     | 丢失              |
| 备份交换器和mandatory参数一起 | ——         | ——             | mandatory参数无效 |



### 过期时间（TTL）

------

TTL，Time to Live的简称，即过期时间。RabbitMQ可以对消息和队列设置TTL。

#### 消息的TTL

------

##### 1、通过设置队列属性

`channel.queueDeclare(...)`方法中加入`x-message-ttl`参数(ms)

还可以通过RabbitCtl设置Policy方式或者通过curl调用HTTP API接口

##### 2、通过消息本身单独设置

`channel.basicPublish(...)`方法中加入`expiration`参数(ms)

还可以通过curl调用HTTP API接口

##### 3、总结

第一种设置队列TTL属性的方法，一旦消息过期，就会从队列中抹去；而在第二种方法中，即使消息过期，也不会马上从队列中抹去（单条消息是否过期是在即将投递到消费者之前判定的）。

第一种方法设置的队列，已过期的消息肯定在队列头部，只要定期从队头开始扫描是否有过期的消息即可；第二种方法设置的队列，由于每条消息设置的过期时间不同，要删除所有过期消息必须扫描整个队列，所以要等到此消息即将被消费时再判定是否过期，再进行删除即可。

> 除此之外还可以通过HTTP API接口设置，Policy的方式来设置TTL



#### 队列的TTL

------

channel.queueDeclare(...)方法中的x-expires参数可以控制队列被自动删除前处于未使用状态的时间。

未使用的意思是队列上没有任何的消费者，队列也没有被重新声明，并且在过期时间段内也未调用过Basic.Get命令。

RabbitMQ会确保在过期时间到达后将队列删除，但是不保障删除的动作有多及时。在RabbitMQ重启后，持久化的队列的过期时间会被重新计算。



### 死信队列

------

DLX，全称为Dead-Letter-Exchange，可以称之为死信交换器（死信邮箱），当消息在一个队列中变成死信。

消息变成死信一般是以下几个原因：

1. 消息被拒绝（Basic.Reject/Basic.Nack），并且设置requeue参数为false；
2. 消息过期；
3. 队列达到最大长度；

DLX也是一个正常的交换器，和一般的交换器没有区别，能在任何队列上被指定，就是设置某个队列的属性。当队列中存在死信时，RabbitMQ自动将消息重新发布到设置的DLX上去，进而被路由到**死信队列**。

*可以监听这个队列中的消息以进行相应的处理，这个特性与消息的TTL设置为0配合使用可以弥补immediate参数的功能*

通过在`channel.queueDeclare`方法中设置`x-dead-letter-exchange`参数来为这个队列添加DLX：

```java
channel.exchangeDeclare("dlx_exchange", "direct");//创建DLX：dlx_exchange
Map<String, Object> args=new HashMap<String, Object>();
args.put("x-dead-letter-exchange", "dix_exchange");
//为队列myqueue添加DLX
channel.queueDeclare("myqueue", false, false, false, args);
```

也可以为这个DLX指定路由键，如果没有特殊指定，则使用原队列的路由键：

```java
args.put("x-dead-letter-exchange", "dlx_routing-key");
```

还可以通过Policy的方式设置：... ...；



DLX可以处理异常情况下，消息不能够被消费者正确消费（Basic.Nack或Basic.Reject）而被置入死信队列中的情况。

DLX配合TTL使用还可以实现延时队列的功能。



### 延迟队列

------

延迟队列存储的对象是对应的延迟消息，所谓“延迟消息”是指当消息被发送后，并不让消费者立刻拿到消息，而是等待特定时间后，消费者才可以拿到这个消息进行消费。

延迟队列的使用场景有很多，比如：

- 在订单系统中，一个用户下单之后通常有30分钟的时间进行支付，如果30分钟之内没有支付成功，那么这个订单将进行异常处理。
- 用户希望通过手机远程遥控家里的设备在指定的时间进行工作，这时就可以将用户指令发送到延迟队列，当指令设定的时间到了再将指令推送到智能设备。

死信队列的用法也是延迟队列的用法，将每条消息都设置为10s的延迟，队列中的消息过期之后再推送到被订阅的队列中，这样就实现了延迟10s的作用。

延迟队列根据延迟时间的长短分为多个等级，一般分为5s、10s、30s、1min、5min、10min、30min、1hour这几个维度。



### 优先级队列

------

即具有高优先级的队列具有高的优先权，优先级高的消息具备优先被消费的特权。

可以通过设置`x-max-prioruty`参数来实现

```java
Map<String, Object>  args = new HashMap<String,Object>();
args.put("x-rnax-priority" ,10);
channel.queueDeclare( "queue.priority", true, fa1se, false, args) ;
```

设置完成后，可以看到“Pri”的标识。

*当消费者的消费速度大于生产者的速度且Broker中没有消息堆积的情况下，对发送的消息设置优先级也就没有什么实际意义。*

代码中设置消息的优先级为5，默认最低为0，最高为队列设置的最大优先级。优先级高的消息可以被优先消费。



### RPC实现

------

RPC，是Remote Procedure Call的简称，即远程过程调用。是一种通过网络从远程计算机上请求服务，而不需要了解底层网络的技术。

RPC的协议有很多，比如最早的CORBA、Java RMI、WebService的RPC风格、Hessian、Thrift甚至还有Restful API。

一般在RabbitMQ中进行RPC是很简单，客户端发送请求消息，服务端回复响应的消息。为了接收响应的消息，我们需要在请求消息中发送一个回调队列。

- replyTo：通常用来设置一个回调队列；
- correlationId：用来关联请求（request）和其调用RPC之后的回复（response）。



### 持久化

### 生产者确认

### 消费端要点介绍

### 消息传输保障

### 小结