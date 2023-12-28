1. udp最小包体是它的包头，一共46字节。
以太网首部 14个字节。以太网尾部 4字节。IP首部 20个字节， TCP首部， 20字节，UDP首部 8字节。

UDP首部： from端口，to端口，长度，校验和。 都是short，一共8字节。
TCP首部： from端口(short)，to端口(short)，序号（int）,确认号（int）,包头长度（int）数据包偏移（4bit）, 保留，flag，校验和。一共20字节。
IP首部： 版本号，包头长度，服务类型，ip包总长，标识符， 行分段，标记，片偏移，TTL，PROTOCAL(下面介绍), headChecksum，from地址，to地址 一共20字节。


协议（Protocol）：长度8比特。标识了上层所使用的协议。以下是比较常用的协议号：
1 ICMP（ping， Tracert（Debug包的路径）， NAQ（网络质量分析）
2 IGMP
6 TCP
17 UDP
88 IGRP
89 OSPF

UDP可以使用来给局域网的设备广播消息。这个就催生了一种服务器，DHCP服务器，设备加入网络的时候，广告DHCP消息来寻找DHCP服务器，然后通DHCP来自动配置网络。
TCP则不能广告，它是面向连接的。