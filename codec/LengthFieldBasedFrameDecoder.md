#LengthFieldBasedFrameDecoder
继承ByteToMessageDecoder，从字面上可以看出，它是基于长度字段的解码器。  
数据传输过来的时候是连续的，那么分割这包数据的大小。长度分割，一种是定长的，所有消息的长度固定值；一种是变长的，根据协议格式，先获取到该消息的长度值，然后在根据长度值获取到消息。按分隔符分割。  
那么我们这个编码器就是处理第二种情况。
这个解码器，可以通过消息中长度字段的数值，动态的分割收到的bytebufs。如果一个消息用一个整形的头字段来标示消息体长度或者整个消息长度，那么这个解码器会特别的有用。  

LengthFieldBasedFrameDecoder 有很多配置参数，可以处理常见基于长度字段的client-server协议。下面的例子，可以给你一些基本的理解。

##两个字节的长度字段，0偏移，不跳过头字段。
下面例子中，长度字段中12(0x0c)表示内容（“Hello, WORLD")的长度为12，

```
 lengthFieldOffset   = 0
 lengthFieldLength   = 2
 lengthAdjustment    = 0
 initialBytesToStrip = 0 (= do not strip header)

 BEFORE DECODE (14 bytes)         AFTER DECODE (14 bytes)
 +--------+----------------+      +--------+----------------+
 | Length | Actual Content |----->| Length | Actual Content |
 | 0x000C | "HELLO, WORLD" |      | 0x000C | "HELLO, WORLD" |
 +--------+----------------+      +--------+----------------+
```
 
 
##两个字节的长度字段，0偏移，跳过头字段。
 ```
 lengthFieldOffset   = 0
 lengthFieldLength   = 2
 lengthAdjustment    = 0
 initialBytesToStrip = 2 (= the length of the Length field)

 BEFORE DECODE (14 bytes)         AFTER DECODE (12 bytes)
 +--------+----------------+      +----------------+
 | Length | Actual Content |----->| Actual Content |
 | 0x000C | "HELLO, WORLD" |      | "HELLO, WORLD" |
 +--------+----------------+      +----------------+
 ```
##两个字节的长度字段，0偏移，不跳过，长度表示整个消息的长度。
 ```
 lengthFieldOffset   =  0
 lengthFieldLength   =  2
 lengthAdjustment    = -2 (= the length of the Length field)
 initialBytesToStrip =  0

 BEFORE DECODE (14 bytes)         AFTER DECODE (14 bytes)
 +--------+----------------+      +--------+----------------+
 | Length | Actual Content |----->| Length | Actual Content |
 | 0x000E | "HELLO, WORLD" |      | 0x000E | "HELLO, WORLD" |
 +--------+----------------+      +--------+----------------+
  ```
  
##头字段长度为5个字节，后三个字节表示长度，不跳过头字段

```
 lengthFieldOffset   = 2 (= the length of Header 1)
 lengthFieldLength   = 3
 lengthAdjustment    = 0
 initialBytesToStrip = 0

 BEFORE DECODE (17 bytes)                      AFTER DECODE (17 bytes)
 +----------+----------+----------------+      +----------+----------+----------------+
 | Header 1 |  Length  | Actual Content |----->| Header 1 |  Length  | Actual Content |
 |  0xCAFE  | 0x00000C | "HELLO, WORLD" |      |  0xCAFE  | 0x00000C | "HELLO, WORLD" |
 +----------+----------+----------------+      +----------+----------+----------------+
```

##头字段长度为5个字节，前三个字节表示长度，不跳过头字段

```
 lengthFieldOffset   = 0
 lengthFieldLength   = 3
 lengthAdjustment    = 2 (= the length of Header 1)
 initialBytesToStrip = 0

 BEFORE DECODE (17 bytes)                      AFTER DECODE (17 bytes)
 +----------+----------+----------------+      +----------+----------+----------------+
 |  Length  | Header 1 | Actual Content |----->|  Length  | Header 1 | Actual Content |
 | 0x00000C |  0xCAFE  | "HELLO, WORLD" |      | 0x00000C |  0xCAFE  | "HELLO, WORLD" |
 +----------+----------+----------------+      +----------+----------+----------------+

```


## 头字段长度为4，字段长度为2，偏移为1，跳过三个字节

```
 lengthFieldOffset   = 1 (= the length of HDR1)
 lengthFieldLength   = 2
 lengthAdjustment    = 1 (= the length of HDR2)
 initialBytesToStrip = 3 (= the length of HDR1 + LEN)

 BEFORE DECODE (16 bytes)                       AFTER DECODE (13 bytes)
 +------+--------+------+----------------+      +------+----------------+
 | HDR1 | Length | HDR2 | Actual Content |----->| HDR2 | Actual Content |
 | 0xCA | 0x000C | 0xFE | "HELLO, WORLD" |      | 0xFE | "HELLO, WORLD" |
 +------+--------+------+----------------+      +------+----------------+
```

##头字段长度为4，字段长度为2，偏移为1,跳过三个字节 长度代表整个消息的长度


```
 lengthFieldOffset   =  1
 lengthFieldLength   =  2
 lengthAdjustment    = -3 (= the length of HDR1 + LEN, negative)
 initialBytesToStrip =  3

 BEFORE DECODE (16 bytes)                       AFTER DECODE (13 bytes)
 +------+--------+------+----------------+      +------+----------------+
 | HDR1 | Length | HDR2 | Actual Content |----->| HDR2 | Actual Content |
 | 0xCA | 0x0010 | 0xFE | "HELLO, WORLD" |      | 0xFE | "HELLO, WORLD" |
 +------+--------+------+----------------+      +------+----------------+
```


##构造函数

```
    private final ByteOrder byteOrder;  //大端和小端
    private final int maxFrameLength;  //最大长度
    private final int lengthFieldOffset;  //长度字段偏移
    private final int lengthFieldLength;   //长度字段长度
    private final int lengthFieldEndOffset;  //长度字段尾部的偏移
    private final int lengthAdjustment;   //长度矫正
    private final int initialBytesToStrip;  //跳过字段
    private final boolean failFast;     
    private boolean discardingTooLongFrame;  //是否丢弃
    private long tooLongFrameLength;   //过长的字段
    private long bytesToDiscard;    // 丢弃bytes  
    
    
    public LengthFieldBasedFrameDecoder(
            ByteOrder byteOrder, int maxFrameLength, int lengthFieldOffset, int lengthFieldLength,
            int lengthAdjustment, int initialBytesToStrip, boolean failFast) 
            
            
 ```


<https://www.jianshu.com/p/a0a51fd79f62>