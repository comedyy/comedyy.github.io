# mac命令行

## launchctl
看它的名字，可以看到是控制启动用的。用来操作mac的开机启动项的。
`launchctl list` 可以看到当前所有的开启项目，以及当前的状态。
`launchctl load/unload` 可以用来启动或者关闭这些启动项
这些启动项的plist位置在
`/System/Library/LaunchDaemons/`系统级别后台守护进程
`/System/Library/LaunchAgents/` 系统级别启动任务
`/Library/LaunchDaemons` 管理员定义的后台守护进程
`/Library/LaunchAgents` 管理员定义的启动任务
`~/Library/LaunchAgents` 用户定义的启动任务
