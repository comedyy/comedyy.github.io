## dotnet的各种问题。
1. 遇到linux上一个litnetlib的无法接收消息的问题，使用top查看是发现死循环了。
2. 可以使用 ps -T -p <pid> 来查看是哪个线程卡死了。 或者使用 top -H -p <pid>
3. 使用dotnet-stack来输出堆栈，需要安装这个工具：`dotnet tool install --global dotnet-stack`
4. dotnet-stack ps 查看所有的应用列表
5. dotnet-stack report -p <pid> 输出堆栈。

