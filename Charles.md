## Charles查看ios的https请求

我们知道https请求是加密的，而且是很安全的，可以防止中间人攻击的一系列的攻击方式。可以保证一条安全的链路。
别人无法通过网络来查看我们的https请求中的内容，只能知道我们和哪个服务器连接。

但是如果你有发送方的设备，你还是可以把app中的https请求给解析出来。因为你有安装系统证书的权利，从而通过中间人攻击来获取https请求信息。

charles是一款抓包工具，它同时也可以用来查看iphone上的app的https请求。

1. 安装charles，破解注册： https://zhile.io 48891cf209c6d32bf4
2. 运行之后自动运行代理，可以查看到chrome的http请求。
3. 点击 help-> ssl Proxy -> Install Charles Root xxxx to Devices;
4. 就可以看到使用提示，包括安装证书，和代理的地址。
5. 手机上安装信任证书。
6. 手机上 通用-> 关于本机 -> 添加根证书
7. Charles ： Proxy -> ssl proxy setting ，include 添加 *:*

