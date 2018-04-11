

#DefaultChannelId
阅读代码：  
1 private static final long serialVersionUID
 好处以及怎么自动生成 <https://blog.csdn.net/qq_35246620/article/details/77686098>

2 这个类并没有public 构造函数，而是使用newInstance  
好处参考<https://blog.csdn.net/ruiguang21/article/details/79026294>   
总结为：  
newInstance: 弱类型。低效率。只能调用无参构造。  
new: 强类型。相对高效。能调用任何public构造。  
在我们自己项目中如何使用？

```

  static 先执行，生成PROCESS_ID和MACHINE_ID,
  虽然简单的生成代码，但是代码逻辑比较严谨，首先从系统变量里面读取，读取不到使用默认的，保证这两个有指。
  MACHINE_ID  +  PROCESS_ID  + SEQUENCE + TIMESTAMP + RANDOM
  byte写入的时候，字符串使用System.arraycopy, int,long类型的写入，采用自定义的写法。我们一般的是写法都是讲int或者long转换为字符串，然后再写入到byte。但是效率远低于位移写入的方法。  todo测试一下。
  SEQUENCE 是一个AtomicInteger类型，不断累加的。
  
  Long.reverse(System.nanoTime()) ^ System.currentTimeMillis()) 这样计算timestamp的好处？ 没有太看懂。
  random 使用PlatformDependent.threadLocalRandom().nextInt() 为什么不直接使用Random?   
  
  
```