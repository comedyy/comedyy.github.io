# nps 内网渗透代理服务器

### 1. 简介

[github](https://github.com/ehang-io/nps) 用于搭建从外网穿透到内网的穿透服务器

主要功能就是可以在外网搭建一个服务器来穿透内网。在外部用户无感知的情况下跟内网的机器进行同行。`nps`负责这一块的协议数据转发，支持`tcp`，`udp`协议。



### 2.使用

下载 `server`跟`client`二进制文件，分别安装在服务器跟目标机器上。(按照`github`上的安装步骤)



服务器记得使用管理员权限!!



登入到`server`对应的web后台，创建并配置`client`信息。

配置添加client的隧道信息。（可以有`tcp隧道`,`udp隧道`,`http代理`，`sock代理`,`私密链接`，`p2p连接` ）一个client似乎只能有一个隧道。

配置完`client`信息之后，在`client`的详细信息那里有启动客户端的参数。使用那些参数来在目标机器上启动`client`。

```
./npc.exe -server=localhost:8024 -vkey=dbz5qmhwq6l7gxrg -type=tcp  // 客户端启动命令
```



以`tcp`隧道为例子，把服务器端口配置为 8888， 目标端口配置成 8080，服务器的`ip`为1.1.1.1。

浏览器中访问`http:/1.1.1.1:8888` 就可以访问目标机器的8080端口了。



### 3. 代码分析









