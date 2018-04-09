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

```