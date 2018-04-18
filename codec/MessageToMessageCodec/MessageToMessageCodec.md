#MessageToMessageCodec
这个是一个Codec可以on-the-fly编解码消息。  
可以认为是MessageToMessageDecoder和MessageToMessageEncoder的组合。  
下面例子是一个decode （Integer to Long） encode( Long to Integer)的。

```
 public class NumberCodec extends MessageToMessageCodec<Integer, Long> {
          @Override
         public Long decode(ChannelHandlerContext ctx, Integer msg, List<Object> out)
                 throws Exception {
             out.add(msg.longValue());
         }

          @Override
         public Integer encode(ChannelHandlerContext ctx, Long msg, List<Object> out)
                 throws Exception {
             out.add(msg.intValue());
         }
     }
```