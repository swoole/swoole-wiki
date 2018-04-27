# WebSocket

 `1.7.9`增加了内置的`WebSocket`服务器支持，通过几行PHP代码就可以写出一个异步非阻塞多进程的`WebSocket`服务器。


```php
$server = new swoole_websocket_server("0.0.0.0", 9501);

$server->on('open', function (swoole_websocket_server $server, $request) {
    echo "server: handshake success with fd{$request->fd}\n";
});

$server->on('message', function (swoole_websocket_server $server, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $server->push($frame->fd, "this is server");
});

$server->on('close', function ($ser, $fd) {
    echo "client {$fd} closed\n";
});

$server->start();
```

onRequest回调
----
swoole_websocket_server 继承自 swoole_http_server

* 设置了`onRequest`回调，websocket服务器也可以同时作为http服务器
* 未设置`onRequest`回调，websocket服务器收到http请求后会返回`http 400`错误页面
* 如果想通过接收http触发所有websocket的推送，需要注意作用域的问题，面向过程请使用`global`对`swoole_websocket_server`进行引用，面向对象可以把`swoole_websocket_server`设置成一个成员属性

1、面向过程代码
```php
$server = new swoole_websocket_server("0.0.0.0", 9501);
$server->on('open', function (swoole_websocket_server $server, $request) {
		echo "server: handshake success with fd{$request->fd}\n";
	});
$server->on('message', function (swoole_websocket_server $server, $frame) {
		echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
		$server->push($frame->fd, "this is server");
	});
$server->on('close', function ($ser, $fd) {
		echo "client {$fd} closed\n";
	});
$server->on('request', function (swoole_http_request $request, swoole_http_response $response) {
		global $server;//调用外部的server
		// $server->connections 遍历所有websocket连接用户的fd，给所有用户推送
		foreach ($server->connections as $fd) {
			$server->push($fd, $request->get['message']);
		}
	});
$server->start();
```
2、面向对象代码
```php
class WebsocketTest {
	public $server;
	public function __construct() {
		$this->server = new swoole_websocket_server("0.0.0.0", 9501);
		$this->server->on('open', function (swoole_websocket_server $server, $request) {
				echo "server: handshake success with fd{$request->fd}\n";
			});
		$this->server->on('message', function (swoole_websocket_server $server, $frame) {
				echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
				$server->push($frame->fd, "this is server");
			});
		$this->server->on('close', function ($ser, $fd) {
				echo "client {$fd} closed\n";
			});
		$this->server->on('request', function ($request, $response) {
				// 接收http请求从get获取message参数的值，给用户推送
				// $this->server->connections 遍历所有websocket连接用户的fd，给所有用户推送
				foreach ($this->server->connections as $fd) {
					$this->server->push($fd, $request->get['message']);
				}
			});
		$this->server->start();
	}
}
new WebsocketTest();
```
客户端
----
* Chrome/Firefox/高版本IE/Safari等浏览器内置了JS语言的WebSocket客户端
* 微信小程序开发框架内置的WebSocket客户端
* 异步的PHP程序中可以使用`Swoole\Http\Client`作为WebSocket客户端
* apache/php-fpm或其他同步阻塞的PHP程序中可以使用swoole/framework提供的[同步WebSocket客户端](https://github.com/swoole/framework/blob/master/libs/Swoole/Client/WebSocket.php)
* 非WebSocket客户端不能与WebSocket服务器通信

