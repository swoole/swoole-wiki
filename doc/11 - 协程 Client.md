# 协程 Client

 `Swoole`提供了`6`种协程`Client`：

1. TCP/UDP Client：`Swoole\Coroutine\Client`
2. HTTP/WebSocket Client：`Swoole\Coroutine\HTTP\Client`
3. HTTP2 Client：`Swoole\Coroutine\HTTP2\Client`
3. Redis Client：`Swoole\Coroutine\Redis`
4. Mysql Client：`Swoole\Coroutine\MySQL`
4. Postgresql Client：`Swoole\Coroutine\PostgreSQL`

* 在协程`Server`中需要使用协程版`Client`，可以实现全异步`Server`
* 其他程序中可以使用`go`关键词手工创建协程
* 同时`Swoole`提供了协程工具集：`Swoole\Coroutine`，提供了获取当前协程`id`，反射调用等能力。

