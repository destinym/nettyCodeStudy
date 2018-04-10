
# AbstractBootstrap
```
AbstractBootstrap is a helper class that makes it easy to bootstrap a Channel.
It support method-chaining to provide an easy way to configure the AbstractBootstrap.
When not used in a ServerBootstrap context, the bind() methods are useful for connectionless transports such as datagram (UDP).
```
正如说明文档中所说，这个类的主要作用是用来引导建立channel。

直接使用它的子类有Bootstrap, ServerBootstrap

```
public abstract class AbstractBootstrap
<B extends AbstractBootstrap<B, C>, C extends Channel>
 implements Cloneable
```
这里泛型使用十分巧妙，但是我目前还不是特别理解为什么这么使用。
泛型B是AbstractBootstrap的子类， C是Channel的子类。

## 成员变量
```
    volatile EventLoopGroup group;
    @SuppressWarnings("deprecation")
    private volatile ChannelFactory<? extends C> channelFactory;
    private volatile SocketAddress localAddress;
    private final Map<ChannelOption<?>, Object> options = new LinkedHashMap<ChannelOption<?>, Object>();
    private final Map<AttributeKey<?>, Object> attrs = new LinkedHashMap<AttributeKey<?>, Object>();
    private volatile ChannelHandler handler;
```
## 函数
###构造函数
```
AbstractBootstrap（） //防止错误的package
AbstractBootstrap(AbstractBootstrap<B, C> bootstrap)

// EventLoopGroup 是用来处理所有将创建的事物
public B group(EventLoopGroup group) 

// channelClass是用来创建channel实例的
public B channel(Class<? extends C> channelClass) 

// 和上面方法一样也是用来创建channel实例的，但是一般只有在复杂的需求的时候才会使用。
// 如果你使用无参数的构造函数，强烈推荐使用Channel简化你的代码
 public B channelFactory(io.netty.channel.ChannelFactory<? extends C> channelFactory) 
 
 //绑定socekt address 网卡
public B  localAddress(SocketAddress localAddress) 
// 绑定端口,多网卡都会绑定
public B  localAddress(int inetPort)
// 通上
public B  localAddress(String inetHost, int inetPort)
// 通上
public B localAddress(InetAddress inetHost, int inetPort)

//设置channnl创建时候的属性，但是如果value为空，则清空之前设置的option。
 public <T> B option(ChannelOption<T> option, T value) 
 
 // 设置一写初始化的属性，但是如果value为空，则清除该属性。
  public <T> B attr(AttributeKey<T> key, T value) 
  
//参数验证
 public B validate() 
   
//clone 深拷贝
public abstract B clone();

// 创建一个新的channel, 注册到eventloop上面
public ChannelFuture register()

// 绑定 channel
 public ChannelFuture bind()  
 public ChannelFuture bind(int inetPort)
 public ChannelFuture bind(String inetHost, int inetPort) 
 public ChannelFuture bind(InetAddress inetHost, int inetPort)
 public ChannelFuture bind(SocketAddress localAddress)
 
//设置handler
public B handler(ChannelHandler handler) 

//获取配置
public abstract AbstractBootstrapConfig<B, C> config()



  
  
  
```
