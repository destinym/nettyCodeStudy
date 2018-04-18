#MessageToByteEncoder
这个类继承ChannelOutboundHandlerAdapter，它会将一个类似于流媒体的message转换为一个ByteBuf.  
下面例子实现了一个将数字类型的转换为一个Bytebuf.
```
  public class IntegerEncoder extends MessageToByteEncoder<Integer> {
          @Override
         public void encode(ChannelHandlerContext ctx, Integer msg, ByteBuf out)
                 throws Exception {
             out.writeInt(msg);
         }
     }
```


写入步骤：  
检测是msg否符合类型。  
分配内存. 
调用encode方法（用户自己定义).  
释放msg
写入. 



