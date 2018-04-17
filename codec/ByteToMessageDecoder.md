#ByteToMessageDecoder
这个类是继承ChannelInboundHandlerAdapter， 将数据流的bytes（bytebuf)解码成message。 下面例子读取到bytebuf创建一个新的bytebuf.

```
 public class SquareDecoder extends ByteToMessageDecoder {
          @Override
         public void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out)
                 throws Exception {
             out.add(in.readBytes(in.readableBytes()));
         }
     }
 
```

##帧检测
DelimiterBasedFrameDecoder, FixedLengthFrameDecoder, LengthFieldBasedFrameDecoder, 和 LineBasedFrameDecoder 是常用的decoder。

如果自定义一个frame decoder,那么你需要小心实现ByteToMessageDecoder。 确保有足够的bytes buffer保存一个完整的数据帧，通过检查ByteBuf.readableBytes()，如果收到的数据，不是一个完整的数据帧，不要修改readindex直接返回，来获取更多的数据。

为了检查完整帧，而不修改reader index,使用 ByteBuf.getInt(int)这个方法。

##陷阱
确定 ByteToMessageDecoder不能是Sharable的。  
有一些方法类似于ByteBuf.readBytes(int)，会造成内存泄露，如果返回的buffer没有被释放或者添加。可以使用 ByteBuf.readSlice(int) 避免这个问题。


<https://www.jianshu.com/p/4c35541eec10>
