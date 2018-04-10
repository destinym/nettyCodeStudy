#Bootstrap
Bootstrap是客户端用来构建channel的工具。   
bind方法是用来udp连接，connect用来tcp连接。

```
//成员变量
// 地址解析器
private static final AddressResolverGroup<?> DEFAULT_RESOLVER; 
//配置
private final BootstrapConfig config
//地址解析器
private volatile AddressResolverGroup<SocketAddress> resolver 
//远端地址
 private volatile SocketAddress remoteAddress;
 
 //参考NameResolver， 指定一个resolver，如果为空则使用默认的
 public Bootstrap resolver(AddressResolverGroup<?> resolver) 
 
 //服务器的地址，connect的使用
 public Bootstrap remoteAddress(SocketAddress remoteAddress) 
 public Bootstrap remoteAddress(String inetHost, int inetPort)
 public Bootstrap remoteAddress(InetAddress inetHost, int inetPort) 
 
 //连接到远端服务器
 public ChannelFuture connect() 
 public ChannelFuture connect(String inetHost, int inetPort)
 public ChannelFuture connect(InetAddress inetHost, int inetPort)
 public ChannelFuture connect(SocketAddress remoteAddress)
 public ChannelFuture connect(SocketAddress remoteAddress, SocketAddress localAddress) 
  
 
 
```