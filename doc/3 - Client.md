# Client

 `swoole_client`提供了`tcp/udp socket`的客户端的封装代码，使用时仅需 `new swoole_client` 即可。
`swoole`的`socket client`对比PHP提供的`stream`族函数有哪些好处：

* `stream`函数存在超时设置的陷阱和Bug，一旦没处理好会导致Server端长时间阻塞
* `fread`有8192长度限制，无法支持UDP的大包
* `swoole_client`支持`waitall`，在知道包长度的情况下可以一次取完，不必循环取。
* `swoole_client`支持UDP connect，解决了UDP串包问题
* `swoole_client`是纯C的代码，专门处理socket，`stream`函数非常复杂。`swoole_client`性能更好

除了普通的同步阻塞+select的使用方法外，swoole_client还支持异步非阻塞回调。

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
> php-fpm/apache环境下只能使用同步客户端  
> apache环境下仅支持`prefork`多进程模式，不支持`prework`多线程

异步非阻塞客户端
----
```php
$client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
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
> 异步客户端只能使用在cli命令行环境
异步的swoole client的使用场景对于新手同学来说可能比较陌生，因为异步客户端是不可以应用在apache或fpm中的，而且仅能用于cli环境。一种比较典型的使用场景就是你的后端服务器前面挡了一个网关服务器，网关和后端之间是通过内网TCP长链接方式通信，网关对所有前端实现http协议，那么，异步的swoole client此时就可以在网关服务器上得到价值实现。具体来说，就是使用swoole http server实现一个常驻内存级的http服务器，然后在swoole http server中使用异步client连接后端服务器。