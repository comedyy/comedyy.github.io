# iterm2 介绍

iterm2是mac上的Terminal的替代品，有更多的功能。

## rz && sz
在Terminal上无法使用这两个命令，因为他们会要求打开选择框。而在iterm2上可以添加触发器，这些触发器可以运行脚本弹出选择文件的框。
[相关的配置](https://blog.51cto.com/oldboy/588592)

## profile
可以配置多个profile，每个profile对应一个特定的窗口，而每个profile又可以在开始运行的时候执行一串命令。因此它可以实现很多功能。
- 配置一个连接你的远程服务器 `ssh root@xx.xx.xx.xx`
- 配置进入你的工作区的git `cd ~/workspace`
- 等等

同时每个profile都可以配置快捷键，可以快速打开。

## hotkeys
这个使用使用到了mac的控制功能，需要把iterm2加入到控制列表。同时在 `Preferences->Keys->HotKeys` 选择你要的快捷键，这样你就可以在其他的界面通过快捷键唤起你的iterm2。

设置iterm2默认打开.sh文件：
右键.sh文件->显示简介->修改打开方式。