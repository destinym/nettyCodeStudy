#DelimiterBasedFrameDecoder
decoder可以通过一个或者多个定界符，去分割收到ByteBufs。
特别在使用null或者换行符的时候，这个类特别有用。
##定界符的含义
 定界符是为了快速的分割。

##定义多于一个分隔符
例如你定义了多个定界符，那么会以能分割最短的来决定。

```
 +--------------+
 | ABC\nDEF\r\n |
 +--------------+

```
会被分割为
```
 +-----+-----+
 | ABC | DEF |
 +-----+-----+
```

源码分析

```
    //分隔符
    private final ByteBuf[] delimiters;
    //最大帧长度
    private final int maxFrameLength; 
    //是否跳过分隔符
    private final boolean stripDelimiter;
    //是否快速跳出
    private final boolean failFast;
    //是否丢弃过长frame
    private boolean discardingTooLongFrame;
    //过长frame长度
    private int tooLongFrameLength;
    //为回车分隔符定制
    /** Set only when decoding with "\n" and "\r\n" as the delimiter.  */
    private final LineBasedFrameDecoder lineBasedDecoder;
    
    
     //构造函数
     public DelimiterBasedFrameDecoder(
            int maxFrameLength, boolean stripDelimiter, boolean failFast, ByteBuf... delimiters) 
     // todo  这里Bytebuf转换为ByteBuf[],为什么使用slice方法？
      public DelimiterBasedFrameDecoder(
            int maxFrameLength, boolean stripDelimiter, boolean failFast,
            ByteBuf delimiter) {
        this(maxFrameLength, stripDelimiter, failFast, new ByteBuf[] {
                delimiter.slice(delimiter.readerIndex(), delimiter.readableBytes())});
    }
```



这里的作用就是返回一个bytebuf,同时自己保留了readIndex和writeIndex,防止外部改变这些。
Returns a slice of this buffer's readable bytes. Modifying the content of the returned buffer or this buffer affects each other's content while they maintain separate indexes and marks. This method is identical to {@code buf.slice(buf.readerIndex(), buf.readableBytes())}. This method does not modify {@code readerIndex} or {@code writerIndex} ofthis buffer.


可以学到的东西，第一，单个参数转换为可变参数，变成数组， 第二，可变参数使用，实际是个arra