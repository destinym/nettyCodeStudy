#ChannelHandlerContext
ChannelHandlerContext是该Handler和它所属的ChannePipeline中其他handler相互作用的工具。另外，一个handler可以通知它所属的pipleline中的下一个handler,也可以动态改变他所属的pipeline。
## 通知
很多方法可以调用
## 修改pipeline
 调用pipeline()
 