# Timer

毫秒精度的定时器。底层基于`epoll_wait`和`setitimer`实现，数据结构使用`最小堆`，可支持添加大量定时器。

* 在同步进程中使用`setitimer`和信号实现，如`Manager`和`TaskWorker`进程
* 在异步进程中使用`epoll_wait/kevent/poll/select`超时时间实现


性能
----
底层使用最小堆数据结构实现定时器，定时器的添加和删除，全部为内存操作，因此性能是非常高的。官方的基准测试脚本 <https://github.com/swoole/swoole-src/blob/master/benchmark/timer.php> 中，添加或删除`10万`个随机时间的定时器耗时为`0.08s`左右。

```shell
~/workspace/swoole/benchmark$ php timer.php
add 100000 timer :0.091133117675781s
del 100000 timer :0.084658145904541s
```

> 定时器是内存操作，无`IO`消耗

差异
-----
`Timer`与`PHP`本身的`pcntl_alarm`是不同的。`pcntl_alarm`是基于时钟信号 + `tick`函数实现存在一些缺陷：

* 最大仅支持到秒，而`Timer`可以到毫秒级别
* 不支持同时设定多个定时器程序
* `pcntl_alarm`依赖`declare(ticks = 1)`，性能很差

零毫秒定时器
----
底层不支持时间参数为`0`的定时器。这与`Node.js`等编程语言不同。在`Swoole`里可以使用`Swoole\Event::defer`实现类似的功能。

```php
Swoole\Event::defer(function () {
	echo "hello\n";
});
```

上述代码与`JS`中的`setTimeout(0, func)`效果是完全一致的。