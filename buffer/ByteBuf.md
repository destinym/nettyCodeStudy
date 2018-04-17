#packeinfo
byte buffer是最基本的数据结构，代表底层二进制和文本信息。netty使用自己的api去代替ByteBuffer.比起Bytebuffer有显著的优点。他设计就是为了解决ByteBuffer的问题。  
* 你可以定义你的buffer类型
* 零拷贝，被composite buffer type实现
* 提供动态buffer类型， 例如String buffer
* 你不需要调用flip()
* 比ByteBuffer快

##可扩展性
丰富的优化快速协议实现。快速查找，易于扩展。
## 零拷贝
为了提高网络程序的性能，你需要降低拷贝操作的数目。例如你需要裁剪组合一些buffers去创建一个新的消息。Netty就会提供 composite buffer，让你能够创建一个新的buffer，而不用拷贝数据。  
下面的例子，你可以在模块化的程序中，分别创建header和body，并且随后把他们组合起来。

```
 +--------+----------+
 | header |   body   |
 +--------+----------+
```

如果是使用BuyteBuffer,你要做的就是新建一个大的buffer，然后把这两个拷贝到新的buffer里面，或者是你可以在NIO中进行一个收集写操作(gathering writing operation),但是它限制你去表示这是一个bytebuffers数组的组合buffer，而不是一个单独buffer.这样会使得状态的管理变得很复杂，和不够抽象。而且，你也无法从NIO信道中读写。




#ByteBuf
一个随机和顺序的获取多个字符集合（可以是0个）。

##创建buffer
推荐使用Unpooled，而不是使用自己的构造方法。
## 随机获取
0为起始位置。
## 顺序获取
通过readIndex来读， 通过writeIndex去写
```
      +-------------------+------------------+------------------+
      | discardable bytes |  readable bytes  |  writable bytes  |
      |                   |     (CONTENT)    |                  |
      +-------------------+------------------+------------------+
      |                   |                  |                  |
      0      <=      readerIndex   <=   writerIndex    <=    capacity
```

##可读字节数
使用read和skip的时候，会改变readerIndex，同时也会改变writerIndex

##可写字节数

##丢弃字节数
丢弃后
```
BEFORE discardReadBytes()

      +-------------------+------------------+------------------+
      | discardable bytes |  readable bytes  |  writable bytes  |
      +-------------------+------------------+------------------+
      |                   |                  |                  |
      0      <=      readerIndex   <=   writerIndex    <=    capacity


  AFTER discardReadBytes()

      +------------------+--------------------------------------+
      |  readable bytes  |    writable bytes (got more space)   |
      +------------------+--------------------------------------+
      |                  |                                      |
 readerIndex (0) <= writerIndex (decreased)        <=        capacity
```


##清除index
BEFORE clear()

      +-------------------+------------------+------------------+
      | discardable bytes |  readable bytes  |  writable bytes  |
      +-------------------+------------------+------------------+
      |                   |                  |                  |
      0      <=      readerIndex   <=   writerIndex    <=    capacity


  AFTER clear()

      +---------------------------------------------------------+
      |             writable bytes (got more space)             |
      +---------------------------------------------------------+
      |                                                         |
      0 = readerIndex = writerIndex            <=            capacity
      
      
##查找 
indexof 
##标记重置

##衍生buffer


#unpooled
unpooled的含义就是非池化的分配，每次回分配一个新的，而不是从内存池里面获取。
创建一个新的空间 或者从存在的byte arrays, byte buffers, string中创建。

可以看到分为两种directBuffer 和 heap

这里有unsafe的方法和普通的方法(safe)， 区别在于unsafe会使用java本身的unsafe方法进行内存读写，效率会高一点。
##Use static import
5个静态导入的方法
##分配一个新的
buffer(int)   
directBuffer(int)
##创建一个wrapped
## 创举一个copied

#UnpooledByteBufAllocator

#UnpooledDirectByteBuf
#UnpooledHeapByteBuf
#UnpooledSlicedByteBuf
#UnpooledUnsafeDirectByteBuf
#UnpooledUnsafeHeapByteBuf
#UnpooledUnsafeNoCleanerDirectByteBuf


#PooledByteBuf

#PoolChunk
1 page poolchunk中中最小的分配单位  
2 chunk 是page的集合  
chunkSize = 2 ^{maxOrder} * pageszie

开始的时候，我们分配字节数组的大小等于chunkSize.  
当需要创建给定大小的bytebuf的时候，我们先查找能够满足该bytebuf大小的第一个位置，返回一个handle去编码这个偏移信息。（剩余的内存会被标记为保留 ）

为了方便，我们这里所有的size都会被标准化normalizeCapacity方法，确保他们需求的大小都是2^n

为了查找定义额offsetinchunk，我们构建了一个完全平衡二叉树，然后把他们存储在array里面，就像heap一样。memoryMap

```
 * depth=0        1 node (chunkSize)
 * depth=1        2 nodes (chunkSize/2)
 * ..
 * ..
 * depth=d        2^d nodes (chunkSize/2^d)
 * ..
 * depth=maxOrder 2^maxOrder nodes (chunkSize/2^{maxOrder} = pageSize)
 *
 * depth=maxOrder is the last level and the leafs consist of pages
```

参考<https://www.jianshu.com/p/c4bd37a3555b>. 
盗用一张图 
![分配图](https://upload-images.jianshu.io/upload_images/2184951-a74c7b675cfa0aa3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/596)

这样就能快速的提高查找效率
为了分配chunkSize/2^k  那么我们只需要在第K层，从做开始寻找没有使用过得节点即可。

算法：

```
把上面的树编码到memoryMap中
memoryMap[id] = x，  x指的是子树在id(大小的id)，第一个可以被分配的节点坐标，如果这个值等于x表示，没有节点可以分配了。

当我们分配，或者释放nodes的时候，我们会更新memoryMap。

```

初始化;  
  memoryMap[id] = depth_of_id
  
观察：

```
 * 1) memoryMap[id] = depth_of_id  =>  未分配
 * 2) memoryMap[id] > depth_of_id  => 至少有一个根节点被分配了，但是还有个一些根节点可以用。
 * 3) memoryMap[id] = maxOrder + 1 =>  都被占用了
```


算法

```
allocateNode(d)， 从第h高度开始分配。
1）从root开始
2) 如果 memoryMap[1] > d ,则表示没办法从chunk上面分配。
3) 如果left node value <= h;  我们可以在左子树上分配，重复直到找到
4）否则，在右子数上面 查找。

allocateRun(size)
1  d = log_2(chunkSize/size)
2 allocateNode(d)

allocateSubpage(size)]

1  allocateNode(maxOrder) 在叶子节点上面查找
2 使用handler去构建一个object, 如果他已经存在，使用初始化， 注意PoolSubpage被添加到subpagesPool 在PoolArena， 当我们初始化的时候。

为了提高缓存的连续性，我们用short value去存储两个信息在memroyMap中。
 * memoryMap[id]= (depth_of_id, x)
 
```


marorder 10 是指 2^10*pagesize的大小。
memoryMap的大小为2048 depthMap也是2048 ？？为什么
```
d [0, 10]
  depth = 1 << d 
  
   d = 0
       depth = 1 
          memoryMap(1) = 0;
   d = 1 
       depth = 2
           memoryMap(2) = 1;
           memoryMap(3) = 1;
   d = 2  
       depth = 4
           memoryMap(4) = 2;
           memoryMap(5) = 2;
           memoryMap(6) = 2;
           memoryMap(7) = 2;
           
           
    d = n  
       depth = 2 ^ n
           memoryMap(2 ^ n ) = n;
           memoryMap(2 ^ n + 1) = n;
         
   d =10       
        depth = 1024
           memoryMap(1024 ) = 10;
           memoryMap(1025) = 10;
           ...
           memoryMap(2047) = 10;
    
           

```

PoolArena
<https://www.jianshu.com/p/4856bd30dd56>
