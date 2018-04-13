#NIO
NIO（Non-blocking I/O，在Java领域，也称为New I/O），是一种同步非阻塞的I/O模型，也是I/O多路复用的基础，已经被越来越多地应用到大型应用服务器，成为解决高并发与大量连接、I/O处理问题的有效方式。
#AbstractNioChannel
从这个代码学习NIO的处理方式

```
//初始化parent, 处理ch, 以及操作op
//后续学习SelectableChannel. java本身提供的
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) 

  里面有有一个 AbstractNioUnsafe类
 public final void connect代码
  调用doConnect,如果成功则继续往下走，如果失败，定义一个定时任务，超时的时候再去进行检查。
  
```


#AbstractNioByteChannel

```
NioByteUnsafe


@Override
public final void read() 

里面有个allocHandle进行循环，如果能够continueReading，进入循环，循环中，会从allocHandle中新建bytebuf, 将byteBufy传递给下一个 pipeline。

 // todo 这里bytebuf的释放问题
protected final int doWrite0(ChannelOutboundBuffer in) 
该函数会调用doWriteInternal， 
判断msg类型，
如果bytebuf,调用dowritebytes方法，同时增加progress监控进度。
如果是FileRegion， doWriteFileRegion， 同时也是progress监控进度
 
 protected void doWrite(ChannelOutboundBuffer in) 
   //writeSpinCount  调用几次doWriteInternal
 
 protected final Object filterOutboundMessage(Object msg)

```

#AbstractNioMessageChannel
```
大部分流程都是类似的
使用一个List<Object> readBuf，来进行存储消息。


```


#NioEventLoop
selecotr学习 todo

<https://www.jianshu.com/p/0d0eece6d467>.  
这个继承了SingleThreadEventLoop。这里的run是一个无限循环。

调用selectStrategy.calculateStrategy来判断，
如果是CONTINUE；
如果是SELECT

select(wakenUp.getAndSet(false));
里面也是个for(;;)循环  
1 定时任务timneout时间快到了，中断本次查询，调用selectnow方法  
2 当wakeup是true的时候，如果有任务加入，则中断本次查询 调用selectNow  
3 阻塞式select。  

   int selectedKeys = selector.select(timeoutMillis);  
   这里会阻塞住。(如果很久怎么办？？） （这里有新任务，会再次调用上面的方法）
   
   调用完成了，五个条件满足一个(选择到了，用户wakeup, 有任务，有计划任务)  
   线程被中断了
   //为什么重建参考<http://www.10tiao.com/html/308/201602/401718035/1.html>.  
   超时selectCnt =1 如果没有超时selectCnt大于最大值，就重建selector
   
   processSelectedKeys
   <https://www.jianshu.com/p/467a9b41833e>
   
   runAllTasak
   
#NioEventLoopGroup

 loop group
 


