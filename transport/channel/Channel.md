#channel

```
A nexus to a network socket or a component which is capable of I/O operations such as read, write, connect, and bind.
```

channel是一个网络socket 或者是一个提供IO操作（读，写，连接，绑定)的组件。

channel可以提供给用户下面功能  

* 当前channnel状态(open?connected?)  
* channel配置参数  
* IO操作（读，写，连接，绑定)  
* ChannelPipeline可以控制所有IO事件和channel相关的请求

所有的IO操作都是异步的。  
Channel是分层的。  
向下获取传输特有的操作。  
释放资源。

```
//返回channel唯一的标示
 ChannelId id() 
 EventLoop eventLoop();
 Channel parent();
 ChannelConfig config();
 
 // 是否open,针对服务端的？
 boolean isOpen();
 //是否在eventloop上面注册
 boolean isRegistered();
 //是否活跃和连接
 boolean isActive();
 

  ChannelMetadata metadata();
  
  //本地和远程的address
  SocketAddress localAddress();
  SocketAddress remoteAddress();
  
  ChannelFuture closeFuture();
  
 boolean isWritable();
   
 long bytesBeforeUnwritable();
    
 long bytesBeforeWritable();
   
   Unsafe unsafe();
     
  ChannelPipeline pipeline();
       
ByteBufAllocator alloc();
       
Channel read();
           
Channel flush();
  
 
```



# ChannelConfig
ChannelConfig是配置channel属性的。
如果需要特定的配置，需要降级。
## Option map

| 名字                   |方法        |
|:-------------         |:---------------:|
| ChannelOption.CONNECT_TIMEOUT_MILLIS      | setConnectTimeoutMillis(int) |
| ChannelOption.WRITE_SPIN_COUNT	     |setWriteSpinCount(int) |
|ChannelOption.WRITE_BUFFER_WATER_MARK | setWriteBufferWaterMark(WriteBufferWaterMark) |
|ChannelOption.ALLOCATOR|setAllocator(ByteBufAllocator)|
|ChannelOption.AUTO_READ|setAutoRead(boolean)|




```
Map<ChannelOption<?>, Object> getOptions();
boolean setOptions(Map<ChannelOption<?>, ?> options);
 <T> T getOption(ChannelOption<T> option);

int getConnectTimeoutMillis（）
 ChannelConfig setConnectTimeoutMillis(int connectTimeoutMillis);
 int getWriteSpinCount();
    ChannelConfig setWriteSpinCount(int writeSpinCount);
     ByteBufAllocator getAllocator();
      ChannelConfig setAllocator(ByteBufAllocator allocator);
      <T extends RecvByteBufAllocator> T getRecvByteBufAllocator();
      ChannelConfig setRecvByteBufAllocator(RecvByteBufAllocator allocator);
       boolean isAutoRead();
       ChannelConfig setAutoRead(boolean autoRead);
       int getWriteBufferHighWaterMark();
        ChannelConfig setWriteBufferHighWaterMark(int writeBufferHighWaterMark);
        int getWriteBufferLowWaterMark();
        ChannelConfig setWriteBufferLowWaterMark(int writeBufferLowWaterMark);
         MessageSizeEstimator getMessageSizeEstimator();
          ChannelConfig setMessageSizeEstimator(MessageSizeEstimator estimator);
          WriteBufferWaterMark getWriteBufferWaterMark();
          hannelConfig setWriteBufferWaterMark(WriteBufferWaterMark writeBufferWaterMark);
        
 
```


#ChannelFactory

 T newChannel();
 
#ChannelException

#ChannelFlushPromiseNotifier
todo 

#ChannelFuture
Channel IO操作的异步结果。
所有的IO操作在Netty中都是异步的。 这意味着，任何IO操作都要立即返回，而不需要保证IO操作都要完成。所以，你需要返回一个ChannelFuture，它会在以后通知你IO操作的状态和结果。

ChannelFuture要么是完成的或者未完成的。当IO操作开始，一个新的future就会别创建，刚开始这个future肯定是没有完成的， 当I/O完成了，他就会被标记为更加详细的信息。


```

                                      +---------------------------+
                                      | Completed successfully    |
                                      +---------------------------+
                                 +---->      isDone() = true      |
 +--------------------------+    |    |   isSuccess() = true      |
 |        Uncompleted       |    |    +===========================+
 +--------------------------+    |    | Completed with failure    |
 |      isDone() = false    |    |    +---------------------------+
 |   isSuccess() = false    |----+---->      isDone() = true      |
 | isCancelled() = false    |    |    |       cause() = non-null  |
 |       cause() = null     |    |    +===========================+
 +--------------------------+    |    | Completed by cancellation |
                                 |    +---------------------------+
                                 +---->      isDone() = true      |
                                      | isCancelled() = true      |
                                      +---------------------------+
                                      
                                      
```
 
有很多方法，让你检测I/O. 你可以使用 add ChannelFutureListener，当IO完成就会收到通知。  

##倾向于使用addListener(GenericFutureListener) 而不是await().  
因为addListener是非阻塞的，而await是阻塞的。  


## 不要在 ChannelHandler内部调用await()
因为ChannelHandler本来就是一个IO线程，这样等待，可能导致死锁。

## 不要混淆I/O超时 await超时

```
 ChannelFuture addListener(GenericFutureListener<? extends Future<? super Void>> listener);
 ChannelFuture addListeners(GenericFutureListener<? extends Future<? super Void>>... listeners);
 ChannelFuture removeListener(GenericFutureListener<? extends Future<? super Void>> listener);
 ChannelFuture removeListeners(GenericFutureListener<? extends Future<? super Void>>... listeners);
 ChannelFuture sync() throws InterruptedException;
 ChannelFuture syncUninterruptibly();
  ChannelFuture await() throws InterruptedException;
  ChannelFuture awaitUninterruptibly();
```

#ChannelFutureListener

监听ChannelFuture的结果，调用ChannelFuture.addListener(GenericFutureListener)方法后，IO操作结果就会通知。
##调用者快速的获取控制权
GenericFutureListener.operationComplete(Future) 可以直接被IO线程调用。 因此，执行一个耗时的任务或者一个阻塞的操作在handler里面的时候，可能导致非预期的停顿在IO的时候。  如果你想在IO完成后，进行一个阻塞的操作，那么你需要运行上面的操作在另外一个线程中。


#ChannelId
channel id
#ChannelMetadata
#ChannelOption

