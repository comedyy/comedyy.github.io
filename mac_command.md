# mac命令行

## launchctl
看它的名字，可以看到是控制启动用的。用来操作mac的开机启动项的。
- `launchctl list` 可以看到当前所有的开启项目，以及当前的状态。
- `launchctl load/unload` 可以用来启动或者关闭这些启动项

这些启动项的plist位置在
- `/System/Library/LaunchDaemons/`系统级别后台守护进程
- `/System/Library/LaunchAgents/` 系统级别启动任务
- `/Library/LaunchDaemons` 管理员定义的后台守护进程
- `/Library/LaunchAgents` 管理员定义的启动任务
- `~/Library/LaunchAgents` 用户定义的启动任务


## grep && cat
`cat` 可以把文件读入并显示到终端上
`grep` 可以用来选出包含关键字的行，或者连带附近的行
`grep "FIND_TEXT" -A10` 查找连带后10行
`grep "FIND_TEXT" -B10` 查找连带前10行
`grep "FIND_TEXT" -C10` 查找连带前后10行

于是可以用管道把它们连起来：
`cat xxx.log | grep "LUA_ERROR" -A20` 用来查找错误日志并且显示它包含的堆栈信息（堆栈信息一般在错误信息下面）

## rz && sz （第三方）
- mac上需要使用brew 安装 `brew install lrzsz`
- centos 上需要 `yum install lrzsz -y`
就可以使用他们来上传下载了。

相关链接：[mac上安装iterm2](./mac/iterm2.md)

## md5 && md5sum
`md5` 是mac版本
`md5sum` 是linux版本

## du && df
`df` 是在/bin, 用来统计当前的硬盘用量 `du -h` 用可读性好的方式输出硬盘使用 `-T`还可以查看磁盘的文件类型。`-i`显示[inode](https://www.ruanyifeng.com/blog/2011/12/inode.html)信息。
`du` 是在/usr/bin，用来计算当前目录的硬盘占用 `-h` 同样是可读性 `-s` 标示读一个目录 `du -sh *`可以输出当前目录所有的文件大小

## ln && mklink
`ln`是linux跟mac上的创建链接的方式 `ln -s source/ link/`
`mklink` 是windows上的创建链接方式 `mklink /j link/ source`