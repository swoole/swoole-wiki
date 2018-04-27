# 协程 Socket

 `Swoole-2.2`版本增加了更底层的`Coroutine\Socket`模块，相比`Server`和`Client`相关模块`Socket`可以实现更细粒度的一些`IO`操作。

可使用`Co\Socket`短命名简化类名。

> 需要`2.2.0`或更高版本

协程调度
----
`Coroutine\Socket`模块提供的`IO`操作接口均为同步编程风格，底层自动使用协程调度器实现异步非阻塞`IO`。

错误码
----
在执行`socket`相关系统调用时，可能返回`-1`错误，底层会设置`Coroutine\Socket->$errCode`属性为系统错误编号`errno`，请参考响应的`man`文档。如`$socket->accept()`返回错误时，`errCode`含义可以参考`man accept`中列出的错误码文档。
