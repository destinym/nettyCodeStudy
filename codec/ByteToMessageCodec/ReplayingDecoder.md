#ReplayingDecoder
这个是一个特殊的变化的ByteToMessageDecoder，实现了在blocking I/O中实现非blocking的decoder。    
ReplayingDecoder和 ByteToMessageDecoder最大不同是，ReplayingDecoder允许你使用decode和decoeLast方法，就像是你已经全部收到了数据，而不是检查是是否所需要的数据都被获取到了。

例如.

```
public class IntegerHeaderFrameDecoder extends ByteToMessageDecoder {
    @Override
   protected void decode(ChannelHandlerContext ctx,
                           ByteBuf buf, List<Object> out) throws Exception {

     if (buf.readableBytes() < 4) {
        return;
     }

     buf.markReaderIndex();
     int length = buf.readInt();

     if (buf.readableBytes() < length) {
        buf.resetReaderIndex();
        return;
     }

     out.add(buf.readBytes(length));
   }
 }
```

可以简化为

```
public class IntegerHeaderFrameDecoder extends ReplayingDecoder<Void> {

   protected void decode(ChannelHandlerContext ctx,
                           ByteBuf buf) throws Exception {

     out.add(buf.readBytes(buf.readInt()));
   }
 }
 
```
## 工作原理
ReplayingDecoder经过一个特殊的ByteBuf实现，这个buffer中如果没有足够的数据，就会抛出一个特定的错误异常。例如上面的IntegerHeaderFrameDecoder，你可以假设buffer中正好有4个或者更多的字节，你可以调用buf.readInt()。如果buffer中确实有4个字节，那么你就能得到你想要的Int类型。否则，会产生一个错误，返回给ReplayingDecoder。 如果ReplayingDecoder抓到这个错误，那么他会将buffer中的readerIndex重置到之前的位置。然后当有数据来的时候再次调用decode。   

注意，ReplayingDecoder总是会返回同一个Error，避免总要新建error而可能造成堆栈溢出。

##局限性
实现这种简便性会有一些代价，ReplayingDecoder会迫使你有下面的限制：  
* 部分buffer操作被禁止  
* 在网络不好，或者消息很复杂的时候，性能可能很差。以为你会一遍又一遍的去解析不完整的消息。  
* 你必须留心decode(...)方法，它会被多次调用去解析同一个消息。 下面的例子会不工作。

```
public class MyDecoder extends ReplayingDecoder<Void> {

   private final Queue<Integer> values = new LinkedList<Integer>();

    @Override
   public void decode(.., ByteBuf buf, List<Object> out) throws Exception {

     // A message contains 2 integers.
     values.offer(buf.readInt());
     values.offer(buf.readInt());

     // This assertion will fail intermittently since values.offer()
     // can be called more than two times!
     assert values.size() == 2;
     out.add(values.poll() + values.poll());
   }
 }
```

应该使用下面的方法

```
public class MyDecoder extends ReplayingDecoder<Void> {

   private final Queue<Integer> values = new LinkedList<Integer>();

    @Override
   public void decode(.., ByteBuf buf, List<Object> out) throws Exception {

     // Revert the state of the variable that might have been changed
     // since the last partial decode.
     values.clear();

     // A message contains 2 integers.
     values.offer(buf.readInt());
     values.offer(buf.readInt());

     // Now we know this assertion will never fail.
     assert values.size() == 2;
     out.add(values.poll() + values.poll());
   }
 }
 
```
  
## 提高性能
  幸运的是，复杂解码器的性能可以使用checkpoint来显著的提高。 checkpoint方法可以更新buffer的'initial'位置，所以 ReplayingDecoder可以退回buffer的readerIndex，到你调用checkpoint()。
  
###调用checkpoint(T) 使用枚举
尽管你可以使用checkpoint，由你自己管理decoder的状态，但是最简单的管理decoder状态的方法就是创建一个枚举，这个枚举可以表示当前decoder的状态，然后当state改变的是调用checkpoint. 如果你要解析的消息比较复杂，你可以定义尽可能多的状态。


```
public enum MyDecoderState {
   READ_LENGTH,
   READ_CONTENT;
 }

 public class IntegerHeaderFrameDecoder
      extends ReplayingDecoder<MyDecoderState> {

   private int length;

   public IntegerHeaderFrameDecoder() {
     // Set the initial state.
     super(MyDecoderState.READ_LENGTH);
   }

    @Override
   protected void decode(ChannelHandlerContext ctx,
                           ByteBuf buf, List<Object> out) throws Exception {
     switch (state()) {
     case READ_LENGTH:
       length = buf.readInt();
       checkpoint(MyDecoderState.READ_CONTENT);
     case READ_CONTENT:
       ByteBuf frame = buf.readBytes(length);
       checkpoint(MyDecoderState.READ_LENGTH);
       out.add(frame);
       break;
     default:
       throw new Error("Shouldn't reach here.");
     }
   }
 }

```
###Calling checkpoint() with no parameter

```
 public class IntegerHeaderFrameDecoder
      extends ReplayingDecoder<Void> {

   private boolean readLength;
   private int length;

    @Override
   protected void decode(ChannelHandlerContext ctx,
                           ByteBuf buf, List<Object> out) throws Exception {
     if (!readLength) {
       length = buf.readInt();
       readLength = true;
       checkpoint();
     }

     if (readLength) {
       ByteBuf frame = buf.readBytes(length);
       readLength = false;
       checkpoint();
       out.add(frame);
     }
   }
 }
 
```

#在pipeline中替换decoder

如果你要一个多路选择器的协议，你可能需要替换 ReplayingDecoder 用另外一个decoder( ReplayingDecoder, ByteToMessageDecoder or MessageToMessageDecoder ).
不能简单的调用 ChannelPipeline.replace(ChannelHandler, String, ChannelHandler)，你需要多做一些事情。

```
 public class FirstDecoder extends ReplayingDecoder<Void> {

      @Override
     protected void decode(ChannelHandlerContext ctx,
                             ByteBuf buf, List<Object> out) {
         ...
         // Decode the first message
         Object firstMessage = ...;

         // Add the second decoder
         ctx.pipeline().addLast("second", new SecondDecoder());

         if (buf.isReadable()) {
             // Hand off the remaining data to the second decoder
             out.add(firstMessage);
             out.add(buf.readBytes(super.actualReadableBytes()));
         } else {
             // Nothing to hand off
             out.add(firstMessage);
         }
         // Remove the first decoder (me)
         ctx.pipeline().remove(this);
     }
```