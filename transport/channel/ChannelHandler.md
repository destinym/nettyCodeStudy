#ChannelHandler
Handles an I/O event or intercepts an I/O operation, and forwards it to its next handler in its ChannelPipeline.

操作所有的I/O事件,或者拦击I/O操作，以及转发给下一个Handler，这些Handler会在ChannelPipeline中。

##子类型
ChannelHandler本身不提供特别多的方法，但是你通常要实现他的子类型中的方法。  

* ChannelInboundHandler处理进入的I/O事件
* ChannelOutboundHandler处理出去的I/O操作

或者，为了方面，下面的adapter类可以使用

* ChannelInboundHandlerAdapter 处理 inbound I/O 事件
* ChannelOutboundHandlerAdapter 处理 outbound I/O 操作
* ChannelDuplexHandler 处理 inbound和 outbound I/O 事件


## 上下文对象
ChannelHandler提供了ChannelHandlerContext对象。 ChannelHandler通常是通过上下文对象和它所属的ChannelPipeline进行交互的。通过使用上下文对象，这个ChannelHandler可以向上游或者下游传输对象，动态修改pipeline,通过attributekeys村存在该handler的信息。


## 状态管理

成员变量
例如，需要鉴权信息，可以使用状态变量来保持，但是你需要每次都new 一个channelHandler，这样才能对应每个channel的状态。

##AttributeKeys
 通常推荐使用成员变量存储handler的状态，但是如果由于某些原因，你不想创建过多的handler实例，你可以使用 AttributeKeys来存储。
 
 
## @Sharable 注解
可以一次创建，多次添加到channelpipelines中

## 值得阅读的附加资源
ChannelPipeline 

# ChannelHandlerAdapter
简单实现了ChannelHandler的框架。

#ChannelInboundHandler
提供一一系列方法
注册，
注销
激活，
取消
激活
读，完成读取
可写状态变化
用户事件触发，
异常捕获

#ChannelInboundHandlerAdapter
简单实现上面的方法

#ChannelOutboundHandler
绑定
连接
断开连接
关闭
deregister 
读
写
刷缓冲

#ChannelOutboundHandlerAdapter
简单实现

#ChannelInitializer
特殊的ChannelInboundHandler，提供简单方式初始化channel,当channel注册到Eventloop的时候。
todo. 文档这里有错误 最新代码里面的是正确的。

我们通常会继承这个类来初始化channel

```
    public MyChannelInit extends ChannelInitializer{
     
       public initChannel(Channel channel){
           channel.pipeline.addLast("myHandler",  new MyHandler);
       }
       
    }
    
```

我们看下整体的调用流程 

```
    1 bootstrap.childHandler(new MyChannelInit());
      它会将MyChannelInit 加入到bootstarp中。
    2 bootstrap启动后会调用channelRegisterd()
      我们这里只有一个hander，所以它实际上回调ChannelInitializer中的channelRegister.
    3 让我们来看channelRegistered中的方法  
      实际上它会去调用私有的 initChannel(ChannelHandlerContext ctx)这个方法，注意不要和上面的方法混淆了。
      首先，有个initMap，判断是否第一次进入。如果不是，返回false.
      第一次进入，首先去调用initChannel(Channel c)这个方法，就是用户自己定义的加入哪些handler。
      最后的时候，会删除remove(ctx)
       
       
      
```