# Async

 `1.6.12`版本增加了异步文件读写，异步DNS，异步Http/WebSocket客户端等特性。开发纯异步非阻塞IO的程序时，不能使用`PHP`自带的网络客户端，如`curl`、`file_get_contents`、`stream`、`sockets`、`mysql`、`redis`。

* `Swoole\Server`的`Task进程`是同步阻塞的，没有`EventLoop`，因此无法使用除定时器之外的任何异步`API`
* `signalfd`是`Linux-2.6.27`提供文件句柄方式处理信号特性，优点是可以将信号加入到`EventLoop`中，`Reactor`操作不会被信号打断提高了性能。缺点是有些同步阻塞的程序可能会出现问题，无法从阻塞中中断，可以使用`swoole_async_set`关闭`signalfd`特性

swoole_async_set
----
此函数可以设置异步IO相关的选项。

```php
swoole_async_set(array $setting);
```

* `thread_num` 设置异步文件IO线程的数量
* `aio_mode` 设置异步文件IO的操作模式，目前支持`SWOOLE_AIO_BASE`（使用类似于Node.js的线程池同步阻塞模拟异步）、`SWOOLE_AIO_LINUX`（Linux Native AIO） 2种模式
* `enable_signalfd` 开启和关闭`signalfd`特性的使用
* `enable_reuse_port` 开启端口复用，需要`Linux-3.10`或更高版本内核，开启后`BASE`模式下每个工作进程都会监听端口，可避免惊群问题
* `socket_buffer_size` 设置SOCKET内存缓存区尺寸
* `socket_dontwait` 在内存缓存区已满的情况下禁止底层阻塞等待
* `log_file` 设置日志文件路径
* `log_level` 设置错误日志等级

> `Linux Native AIO`的优点是由内核支持是真正的异步文件`IO`，缺点是只支持`DirectIO`，无法利用到系统的`PageCache`  
> `4.0`或更高版本已移除`Linux AIO`
