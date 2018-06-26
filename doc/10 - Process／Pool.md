# Process\Pool

进程池，基于`Server`的`Manager`模块实现。可管理多个工作进程。该模块的核心功能为**进程管理**，相比`Process`实现多进程，`Process\Pool`更加简单，封装层次更高，开发者无需编写过多代码即可实现进程管理功能。

> 此特性需要`2.1.2`或更高版本

常量定义
----
* `SWOOLE_IPC_MSGQUEUE` ：系统消息队列通信
* `SWOOLE_IPC_SOCKET`：`SOCKET`通信

异步支持
----
* 可在`onWorkerStart`中使用`Swoole`提供的异步或协程`API`，工作进程即可实现异步
* 底层自带的消息队列和`SOCKET`通信均为同步阻塞`IO`
* 如果进程为异步模式，请勿使用任何自带的同步`IPC`进程通信功能

> 低于`4.0`版本需要在`onWorkerStart`末尾添加`swoole_event_wait`进入事件循环

使用实例
----
```php
$workerNum = 10;
$pool = new Swoole\Process\Pool($workerNum);

$pool->on("WorkerStart", function ($pool, $workerId) {
    echo "Worker#{$workerId} is started\n";
    $redis = new Redis();
    $redis->pconnect('127.0.0.1', 6379);
    $key = "key1";
    while (true) {
         $msgs = $redis->brpop($key, 2);
         if ( $msgs == null) continue;
         var_dump($msgs);
     }
});

$pool->on("WorkerStop", function ($pool, $workerId) {
    echo "Worker#{$workerId} is stopped\n";
});

$pool->start();
```