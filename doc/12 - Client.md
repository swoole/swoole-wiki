# Client

 `Client`提供了`TCP/UDP` `socket`的客户端的封装代码，使用时仅需 `new Swoole\Client` 即可。

优势
----
* `stream`函数存在超时设置的陷阱和`Bug`，一旦没处理好会导致`Server`端长时间阻塞
* `stream`函数的`fread`默认最大`8192`长度限制，无法支持`UDP`的大包
* `Client`支持`waitall`，在有确定包长度时可一次取完，不必循环读取
* `Client`支持`UDP connect`，解决了`UDP`串包问题
* `Client`是纯`C`的代码，专门处理`socket`，`stream`函数非常复杂。`Client`性能更好
* `Client`支持长连接

除了普通的同步阻塞+`select`的使用方法外，`Client`还支持异步非阻塞回调。

同步阻塞客户端
-----
```php
$client = new swoole_client(SWOOLE_SOCK_TCP);
if (!$client->connect('127.0.0.1', 9501, -1))
{
	exit("connect failed. Error: {$client->errCode}\n");
}
$client->send("hello world\n");
echo $client->recv();
$client->close();
```
> `php-fpm/apache`环境下只能使用同步客户端  
> `apache`环境下仅支持`prefork`多进程模式，不支持`prework`多线程

异步非阻塞客户端
----
```php
$client = new Swoole\Client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
$client->on("connect", function(swoole_client $cli) {
    $cli->send("GET / HTTP/1.1\r\n\r\n");
});
$client->on("receive", function(swoole_client $cli, $data){
    echo "Receive: $data";
    $cli->send(str_repeat('A', 100)."\n");
    sleep(1);
});
$client->on("error", function(swoole_client $cli){
    echo "error\n";
});
$client->on("close", function(swoole_client $cli){
    echo "Connection close\n";
});
$client->connect('127.0.0.1', 9501);
```
> 异步客户端只能使用在`cli`命令行环境
