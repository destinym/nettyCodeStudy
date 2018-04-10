#ServerBootstrap

这个类是给服务端使用的，建立一个ServerChannel

```

//由于ServerBootstrap继承AbstraBootstrap，所以自己的配置都用child开头。
private final Map<ChannelOption<?>, Object> childOptions ;
private final Map<AttributeKey<?>, Object> childAttrs ;
private final ServerBootstrapConfig config ;
private volatile EventLoopGroup childGroup;
private volatile ChannelHandler childHandler;


//这两个方法的区别是，第一个方法中的group会既是paranet又是child
//他们会处理所有的IO和事物
public ServerBootstrap group(EventLoopGroup group)
public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)

//指定childOption,如果null删除
public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value) 

//指定childAttr，如果null删除
public <T> ServerBootstrap childAttr(AttributeKey<T> childKey, T value)


//设置childHandler
public ServerBootstrap childHandler(ChannelHandler childHandler)

//初始化
void init(Channel channel) throws Exception

初始化里面有个ServerBootstrapAcceptor这个是用来




        
```