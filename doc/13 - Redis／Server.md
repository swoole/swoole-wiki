# Redis\Server

 `Swoole-1.8.14`版本增加一个兼容`Redis`服务器端协议的`Server`框架，可基于此框架实现`Redis`协议的服务器程序。`Swoole\Redis\Server`继承自`Swoole\Server`，可调用父类提供的所有方法。

`Redis\Server`不需要设置`onReceive`回调。实例程序：<https://github.com/swoole/swoole-src/blob/master/examples/redis/server.php>

可用的客户端
----
* 任意编程语言的redis客户端，包括PHP的redis扩展和phpredis库
* Swoole扩展提供的异步Redis客户端
* Redis提供的命令行工具，包括`redis-cli`、`redis-benchmark`

协程
-----
在`2.0`协程版本中，无法使用`return`返回值的方式发送响应结果。应当使用`$server->send`方法发送数据。

```php
use Swoole\Redis\Server;
use Swoole\Coroutine\Redis;

$serv = new Server('0.0.0.0', 10086, SWOOLE_PROCESS, SWOOLE_SOCK_TCP);
$serv->setHandler('set', function ($fd, $data) use ($serv) {
	$cli = new Redis;
	$cli->connect('0.0.0.0', 6379);
	$cli->set($data[0], $data[1]);
    $serv->send($fd, Server::format(Server::INT, 1));
});

$serv->start();
```