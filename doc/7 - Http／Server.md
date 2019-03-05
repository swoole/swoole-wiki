# Http\Server

> **`Http\Server`对`Http`协议的支持并不完整，建议仅作为应用服务器。并且在前端增加`Nginx`作为代理**

`1.7.7`版本增加了内置`Http`服务器的支持，通过几行代码即可写出一个异步非阻塞多进程的`Http`服务器。

```php
$http = new Swoole\Http\Server("127.0.0.1", 9501);
$http->on('request', function ($request, $response) {
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});
$http->start();
```

通过使用`apache bench`工具进行压力测试，在`Inter Core-I5 4`核 + `8G`内存的普通PC机器上，`Http\Server`可以达到近`11`万`QPS`。远远超过`php-fpm`，`Golang`、`Node.js`自带`Http`服务器。性能几乎接近与`Nginx`的静态文件处理。

```shell
ab -c 200 -n 200000 -k http://127.0.0.1:9501/
```

使用Http2协议
----
* 需要依赖`nghttp2`库，[下载nghttp2](https://github.com/tatsuhiro-t/nghttp2)后编译安装
* 使用SSL下的`Http2`协议必须安装`openssl`, 且需要高版本`openssl`必须支持`TLS1.2`、`ALPN`、`NPN`
* 使用HTTP2不一定要开启SSL

```shell
./configure --enable-openssl --enable-http2
```

设置http服务器的`open_http2_protocol`为`true`
```php
$serv = new Swoole\Http\Server("127.0.0.1", 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$serv->set([
    'ssl_cert_file' => $ssl_dir . '/ssl.crt',
    'ssl_key_file' => $ssl_dir . '/ssl.key',
    'open_http2_protocol' => true,
]);
```

nginx+swoole配置
-----
```
server {
    root /data/wwwroot/;
    server_name local.swoole.com;

    location / {
		proxy_http_version 1.1;
		proxy_set_header Connection "keep-alive";
		proxy_set_header X-Real-IP $remote_addr;
        if (!-e $request_filename) {
             proxy_pass http://127.0.0.1:9501;
        }
    }
}
```
> 通过读取`$request->header['x-real-ip']`来获取客户端的真实`IP`