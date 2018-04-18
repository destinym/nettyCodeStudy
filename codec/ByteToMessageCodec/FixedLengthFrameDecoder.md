#FixedLengthFrameDecoder
定长的
```
 +---+----+------+----+
 | A | BC | DEFG | HI |
 +---+----+------+----+
  FixedLengthFrameDecoder(3) 将会将上面的变成下面的
 +-----+-----+-----+
 | ABC | DEF | GHI |
 +-----+-----+-----+
```

代码很简单，如果长度不够就等待，到了调用readRetainedSlice方法，保存并分割。