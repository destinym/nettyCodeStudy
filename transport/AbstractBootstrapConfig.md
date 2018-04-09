#AbstractBootstrapConfig
暴露AbstractBootstrap的配置文件

```
//构造函数，关联bootstrap
protected AbstractBootstrapConfig(B bootstrap)

//获取localaddres
public final SocketAddress localAddress() 

//获取handler
public final ChannelHandler handler() 

//获取options的拷贝
public final Map<ChannelOption<?>, Object> options()
 
// 获取attr的拷贝
public final Map<AttributeKey<?>, Object> attrs() 
        
```