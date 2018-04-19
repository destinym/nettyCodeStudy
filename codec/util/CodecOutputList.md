#CodecOutputList

```
//垃圾回收
CodecOutputListRecycler NOOP_RECYCLER 
//每个thread分配CodecOutputLists
FastThreadLocal<CodecOutputLists> CODEC_OUTPUT_LISTS_POOL 
每个线程有16个CodecOutputLists

//CodecOutputLists
CodecOutputLists
  {
  CodecOutputList[] elements;
  int mask;
  int currentIdx;
  int count;
  
  }
```

这里的elements是个列表的数组，数组的大小为16，每个列表的大小也为16

getOrCreate 获取一个元素，没有则创建

这个结构其实就是一个二维数组，CodecOutputList是获取当前线程中的一个数组。
每次使用完就会clear掉。



#DateFormatter
