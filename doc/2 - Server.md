# Server

创建一个异步服务器程序，支持`TCP`、`UDP`、`UnixSocket` `3`种协议，支持`IPv4`和`IPv6`，支持`SSL/TLS`单向双向证书的隧道加密。使用者无需关注底层实现细节，仅需要设置网络事件的回调函数即可。

**请勿在使用`Server`创建之前调用其他异步`IO`的`API`，否则将会创建失败。**可以在`Server`启动后`onWorkerStart`回调函数中使用。

> `Server`只能用于`php-cli`环境，在其他环境下会抛出致命错误

构建Server对象
----
```php
$serv = new Swoole\Server('0.0.0.0', 9501, SWOOLE_BASE, SWOOLE_SOCK_TCP);
```

设置运行时参数
----
```php
$serv->set(array(
	'worker_num' => 4,
	'daemonize' => true,
	'backlog' => 128,
));
```
注册事件回调函数
----
```php
$serv->on('Connect', 'my_onConnect');
$serv->on('Receive', 'my_onReceive');
$serv->on('Close', 'my_onClose');
```
PHP中可以使用[4种回调函数的风格](https://wiki.swoole.com/wiki/page/458.html)

启动服务器
----
```php
$serv->start();
```

属性列表
----
```php
$serv->manager_pid;  //管理进程的PID，通过向管理进程发送SIGUSR1信号可实现柔性重启
$serv->master_pid;  //主进程的PID，通过向主进程发送SIGTERM信号可安全关闭服务器
$serv->connections; //当前服务器的客户端连接，可使用foreach遍历所有连接
```

运行流程图
-----
![Swoole扩展架构图](https://wiki.swoole.com/static/uploads/swoole.jpg)

进程/线程结构图
-----
![Swoole进程/线程结构图](https://wiki.swoole.com/static/image/process.jpg)
![进程/线程结构图2](https://wiki.swoole.com/static/uploads/wiki/201808/03/635680420659.png "进程/线程结构图2")
