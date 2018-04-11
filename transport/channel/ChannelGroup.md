#ChannelGroup
线程安全的Set包含了所有open的channel,同时提供各种操作。 使用channelgroup, 你可以将channel分到不同的组里面。而且channel会被自动移除，如果他被关闭了，所以你完全不用担心他的生命周期。 一个channel可以属于多个channelgroup.  
todo 可以试试用来做群管理功能。


##向多个channel广播消息。

下面例子，将channel加入group,然后调用write即可。

```
 ChannelGroup recipients =
         new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
 recipients.add(channelA);
 recipients.add(channelB);
 ..
 recipients.write(Unpooled.copiedBuffer(
         "Service will shut down for maintenance in 5 minutes.",
         CharsetUtil.UTF_8));
```

#通过ChannelGroup简化关闭进程
如果 ServerChannels和non-ServerChannels，在同一个channelgroup里面，先会给 ServerChannels操作，才会进行其他的。



```
//group name
String name();

//find
Channel find(ChannelId id);

//向group中的所有channel写数据，如果Bytebuf/ByteBufHolder类型，那么自动复制，防止删除引起的问题。 另外这个是纯异步的。
ChannelGroupFuture write(Object message);

// 符合ChannelMatcher的发送
ChannelGroupFuture write(Object message, ChannelMatcher matcher);

//If voidPromise is true ChannelOutboundInvoker.voidPromise() is used for the writes and so the same restrictions to the returned ChannelGroupFuture apply as to a void promise. （这句话不太明白)
 ChannelGroupFuture write(Object message, ChannelMatcher matcher, boolean voidPromise);

// flush
ChannelGroup flush();

ChannelGroup flush(ChannelMatcher matcher);

ChannelGroupFuture writeAndFlush(Object message); 

ChannelGroupFuture writeAndFlush(Object message, ChannelMatcher matcher);

ChannelGroupFuture writeAndFlush(Object message, ChannelMatcher matcher, boolean voidPromise);


//disconnect
  ChannelGroupFuture disconnect();
  ChannelGroupFuture disconnect(ChannelMatcher matcher);
  ChannelGroupFuture close();
   ChannelGroupFuture close(ChannelMatcher matcher);
   
   ChannelGroupFuture newCloseFuture();
   
    ChannelGroupFuture newCloseFuture(ChannelMatcher matcher);

```


#ChannelGroupException
exception

#ChannelGroupFuture
异步ChannelGroup的操作的结果。 ChannelGroupFuture是由多个channelFuture组成的，而ChannelFutures是多个不同IO操作的结果。  
所有的I/O操作都是异步的。

其他两点参考CHannelFuture.
##Prefer addListener(GenericFutureListener) to await()
##Do not call await() inside ChannelHandler


#ChannelGroupFutureListener
#ChannelMatcher
#ChannelMatchers

#CombinedIterator

#DefaultChannelGroup
#DefaultChannelGroupFuture
#VoidChannelGroupFuture

