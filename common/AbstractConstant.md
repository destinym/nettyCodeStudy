#AbstractConstant
实现了Constant方法，重写了equals和compareTo方法。

#AbstractReferenceCounted
一个引用计数的对象，需要特别指明回收。
当ReferenceCounted被实例化后，他通过retain增加引用计数的个数，通过release减少引用计数的个数。如果引用计数降低到0，那么对象就会被显式的回收。
如果实现ReferenceCounted的对象是包含者其他实现ReferenceCounted的对象， 那么被包含的对象也会被释放掉，如果这个包含者的计数个数降低到0

#AsciiString




#ConstantPool
里面是一个ConcurrentMap,  
valueof方法调用getOrCreate方法，有的时候直接返回，没有创建返回

putIfAbsent方法,表示map存在调用get, 不存在调用put。 返回值为null,不存在关联，否则返回get结果。

```
// constants的map
ConcurrentMap<String, T> constants 
AtomicInteger nextId 

//
valueOf()
newInstance()

```
#AttributeKey
用于获取 AttributeMap中的 Attribute的值。 确定的是，每个key的名字不能重复。


#Attribute
线程安全。因为存储他的都是concurrentMap，支持原子操作。

#ByteProcessor
提供一个处理bytes集合的机制

#CharsetUtil
工具类，用来处理字符集相关的各种操作。


#DefaultAttributeMap

```
//细节，晚初始化，降低内存消耗，用AtomicReferenceFieldUpdater更新元素，
//不使用concurrentHashMap是因为内存消耗的原因。
private volatile AtomicReferenceArray<DefaultAttribute<?>> attributes;


public <T> Attribute<T> attr(AttributeKey<T> key)
创建attributes。

head是空，表示我们增加attribute的时候，可能会使用增加属性，只是用了cas。 最坏情况，我们回退到sync，浪费两次分配。

DefaultAttribute 是一个链表节点
```


#DomainNameMapping
dns域名解析map

#DomainNameMappingBuilder
builder

#HashedWheelTimer
wheelTimer定时器。
todo后续分析一下源代码


#Recycler
 轻量级的回收器