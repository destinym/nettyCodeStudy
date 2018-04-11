#ChannelPipeline
为什么叫outbound operations  和inbound events 

ChannlePipeline是一个Channel处理或者拦击inbound事物或者oubound操作的列表。 它采用拦截过滤模式从而给用户充分控制权，用户可以处理事物控制，以及handler本身在pipeline中的关系。

## pipleline创建
每个channel都有自己的pipeline，在channel创建的时候，pipeline自己创建。
## pipleine中事件流
下图说明事件流，通过ChannelHandlerContext.fireChannelRead(Object) and ChannelOutboundInvoker.write(Object) 这些方法传递给下一个handler


```
                                             I/O Request
                                            via Channel or
                                        ChannelHandlerContext
                                                      |
  +---------------------------------------------------+---------------+
  |                           ChannelPipeline         |               |
  |                                                  \|/              |
  |    +---------------------+            +-----------+----------+    |
  |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  |               |
  |               |                                  \|/              |
  |    +----------+----------+            +-----------+----------+    |
  |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  .               |
  |               .                                   .               |
  | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
  |        [ method call]                       [method call]         |
  |               .                                   .               |
  |               .                                  \|/              |
  |    +----------+----------+            +-----------+----------+    |
  |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  |               |
  |               |                                  \|/              |
  |    +----------+----------+            +-----------+----------+    |
  |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  |               |
  +---------------+-----------------------------------+---------------+
                  |                                  \|/
  +---------------+-----------------------------------+---------------+
  |               |                                   |               |
  |       [ Socket.read() ]                    [ Socket.write() ]     |
  |                                                                   |
  |  Netty Internal I/O Threads (Transport Implementation)            |
  +-------------------------------------------------------------------+

```

一个inbound事物会被inbound handlers 从上向下处理（左边图所示),通常inbound handler产生的数据来自于socked.read这个I/O线程。
If an inbound event goes beyond the top inbound handler, it is discarded silently, or logged if it needs your attention. (不太理解这个的含义)



一个oubound事物会被oubound handler处理，由上到下 （右图所示),通常outbound从write请求开始的。到最底层就是SocketChannel.write(ByteBuffer).这样的处理。


下面的例子很经典，基本所有的介绍netty中都会用到。
```
 ChannelPipeline p = ...;
 p.addLast("1", new InboundHandlerA());
 p.addLast("2", new InboundHandlerB());
 p.addLast("3", new OutboundHandlerA());
 p.addLast("4", new OutboundHandlerB());
 p.addLast("5", new InboundOutboundHandlerX());
```

inbound 1  -->2 ---> 5
outbound 5 --->4 ---->3



##向下一个handler传递事物

```
Inbound event propagation methods:
	ChannelHandlerContext.fireChannelRegistered()
	ChannelHandlerContext.fireChannelActive()
	ChannelHandlerContext.fireChannelRead(Object)
	ChannelHandlerContext.fireChannelReadComplete()
	ChannelHandlerContext.fireExceptionCaught(Throwable)
	ChannelHandlerContext.fireUserEventTriggered(Object)
	ChannelHandlerContext.fireChannelWritabilityChanged()
	ChannelHandlerContext.fireChannelInactive()
	ChannelHandlerContext.fireChannelUnregistered()
	
Outbound event propagation methods:
	ChannelOutboundInvoker.bind(SocketAddress, ChannelPromise)
	ChannelOutboundInvoker.connect(SocketAddress, SocketAddress, ChannelPromise)
	ChannelOutboundInvoker.write(Object, ChannelPromise)
	ChannelHandlerContext.flush()
	ChannelHandlerContext.read()
	ChannelOutboundInvoker.disconnect(ChannelPromise)
	ChannelOutboundInvoker.close(ChannelPromise)
	ChannelOutboundInvoker.deregister(ChannelPromise)
```


## 创建一个pipeline

一个用户有多个 ChannelHandlers 在pipeline中，去处理I/O读写。 举个例子，一个典型的服务器会去处理下面的handlers在pipeline中，每个系统都有自己复杂个性化的协议和业务逻辑。

1 Protocol Decoder  
2 Protocol Encoder      
3 Business Logic Handler

// 可以让你的逻辑在另外一个group里面运行，从而保证I/O不会被阻塞，如果你的业务逻辑是纯异步的，那么久不用指定另外一个group了。  
pipeline.addLast(group, "handler", new MyBusinessLogicHandler());


## 线程安全的
 任何时候都可以创建
 
 ``` 
     //头部增加handler
     ChannelPipeline addFirst(String name, ChannelHandler handler);
     //指定group
     ChannelPipeline addFirst(EventExecutorGroup group, String name, ChannelHandler handler);
     //尾部增加
      ChannelPipeline addLast(String name, ChannelHandler handler);
      //group
      ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler);
      
       ChannelPipeline addBefore(String baseName, String name, ChannelHandler handler);
       
        ChannelPipeline addBefore(EventExecutorGroup group, String baseName, String name, ChannelHandler handler);
        
    ChannelPipeline addAfter(String baseName, String name, ChannelHandler handler);
    ChannelPipeline addAfter(EventExecutorGroup group, String baseName, String name, ChannelHandler handler);
    
    
     ChannelPipeline addFirst(ChannelHandler... handlers);
     
     ChannelPipeline addFirst(EventExecutorGroup group, ChannelHandler... handlers);
     
    ChannelPipeline addLast(ChannelHandler... handlers);
      
    ChannelPipeline addLast(EventExecutorGroup group, ChannelHandler... handlers);
      
      
    ChannelPipeline remove(ChannelHandler handler);
      
    ChannelHandler remove(String name);
       
     <T extends ChannelHandler> T remove(Class<T> handlerType);
        
 ChannelHandler removeFirst();
 ChannelHandler removeLast();
          
    ChannelPipeline replace(ChannelHandler oldHandler, String newName, ChannelHandler newHandler);
          
   ChannelHandler replace(String oldName, String newName, ChannelHandler newHandler);
            
    ChannelHandler first();
              
              
    ChannelHandlerContext firstContext();
               
               
   ChannelHandler last();
                
    ChannelHandlerContext lastContext();
                
    ChannelHandler get(String name);
                 
     <T extends ChannelHandler> T get(Class<T> handlerType);
                   
   ChannelHandlerContext context(ChannelHandler handler);
                     
    ChannelHandlerContext context(String name);
                     
    ChannelHandlerContext context(Class<? extends ChannelHandler> handlerType);
                     
    Channel channel();

    List<String> names();
      
   Map<String, ChannelHandler> toMap();    

 ```

