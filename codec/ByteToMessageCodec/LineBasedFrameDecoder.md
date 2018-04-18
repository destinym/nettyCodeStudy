#LineBasedFrameDecoder
基于换行符的decoder，\n, \r\n都会被处理  
这个在Telnet里面其实是最常用的。

```
    private final int maxLength; 最大长度
    /** Whether or not to throw an exception as soon as we exceed maxLength. */
    private final boolean failFast;
    private final boolean stripDelimiter;
    
    //是否丢弃
    private boolean discarding;
    private int discardedBytes;
    
        /** Last scan position. */
    private int offset;

```



```
1 查找到\n的位置， 如果是\r\n，需要往前移动一位。  -1 表示没有找到。
2 处理超过最大长度问题。
3 根据是否跳过分隔符，保留对应的bytebuf.

4 如果没有找到，则检查是否超过最大长度，没有继续等待。

5 丢弃模式的逻辑和LengthFieldBasedFrameDecoder一样，这里逻辑代码更简单。


```