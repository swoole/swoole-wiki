# Coroutine

从`2.0`版本开始`Swoole`提供了**协程(Coroutine)**特性，可使用协程+通道的全新编程模式，代替异步回调。应用层可使用完全同步的编程方式，底层自动实现异步`IO`。

```php
go(function () {
	$redis = new Swoole\Coroutine\Redis();
	$redis->connect('127.0.0.1', 6379);
	$val = $redis->get('key');
});
```

> `4.0.0`或更高版本仅支持`PHP7`  
> `4.0.1`版本开始去除了`--enable-coroutine`编译选项，改为[动态配置](https://wiki.swoole.com/wiki/page/949.html)

协程可以理解为纯用户态的线程，其通过**协作**而不是抢占来进行切换。相对于进程或者线程，协程所有的操作都可以在用户态完成，创建和切换的消耗更低。`Swoole`可以为每一个请求创建对应的协程，根据`IO`的状态来合理的调度协程，这会带来了以下优势：

1. 开发者可以无感知的用同步的代码编写方式达到异步IO的效果和性能，避免了传统异步回调所带来的离散的代码逻辑和陷入多层回调中导致代码无法维护
2. 同时由于底层封装了协程，所以对比传统的`PHP`层协程框架，开发者不需要使用`yield`关键词来标识一个协程`IO`操作，所以不再需要对`yield`的语义进行深入理解以及对每一级的调用都修改为`yield`，这极大的提高了开发效率


可以满足大部分开发者的需求。对于私有协议，开发者可以使用协程的`TCP`或者`UDP`接口去方便的封装。

环境要求
----
* PHP版本要求：**>= 7.0**，包括`7.0`、`7.1`、`7.2`
* 基于`Server`、`Http\Server`、`WebSocket\Server`进行开发，底层在`onRequet`, `onReceive`, `onConnect`等事件回调之前自动创建一个协程，在回调函数中使用协程`API`
* 使用`Coroutine::create`或`go`方法创建协程，在创建的协程中使用协程`API`

相关配置
---
在`Server`的`set`方法中增加了一个配置参数`max_coroutine`，用于配置一个`Worker`进程最多同时处理的协程数目。因为随着`Worker`进程处理的协程数目的增加，其占用的内存也会增加，为了避免超出php的`memory_limit`限制，请根据实际业务的压测结果设置该值，默认为`3000`。

使用示例
-----

```php
$http = new swoole_http_server("127.0.0.1", 9501);

$http->on("request", function ($request, $response) {
	$client = new Swoole\Coroutine\Client(SWOOLE_SOCK_TCP);
	$client->connect("127.0.0.1", 8888, 0.5);
	//调用connect将触发协程切换
	$client->send("hello world from swoole");
	//调用recv将触发协程切换
	$ret = $client->recv();
	$response->header("Content-Type", "text/plain");
	$response->end($ret);
	$client->close();
});

$http->start();
```

当代码执行到`connect()和recv()`函数时，底层会触发进行协程切换，此时可以去处理其他的事件或者接受新的请求。当此客户端`连接`成功或者后端服务`回包`后，底层会恢复协程上下文，代码逻辑继续从切换点开始恢复执行。开发者整个过程不需要关心整个切换过程。

注意事项
----

1. 全局变量：协程使得原有的异步逻辑同步化，但是在协程的切换是隐式发生的，所以在协程切换的前后不能保证全局变量以及static变量的一致性。
2. 请勿在**4.0以下的版本**的以下场景中触发协程切换：
    * 析构函数
    * 魔术方法`__call()` `__get()` `__set()` 等
3. 与xdebug、xhprof、blackfire等zend扩展不兼容，例如不能使用xhprof对协程server进行性能分析采样。

协程组件
---------
1. TCP/UDP Client：`Swoole\Coroutine\Client`
2. HTTP/WebSocket Client：`Swoole\Coroutine\HTTP\Client`
3. HTTP2 Client：`Swoole\Coroutine\HTTP2\Client`
4. Redis Client：`Swoole\Coroutine\Redis`
5. Mysql Client：`Swoole\Coroutine\MySQL`
6. Postgresql Client：`Swoole\Coroutine\PostgreSQL`

* 在协程`Server`中需要使用协程版`Client`，可以实现全异步`Server`
* 其他程序中可以使用`go`关键词手工创建协程
* 同时`Swoole`提供了协程工具集：`Swoole\Coroutine`，提供了获取当前协程`id`，反射调用等能力。

