## xcode 问题汇总

### 1. ios版本提升，但是xcode版本太低不支持调试。
xcode支持不支持调试主要是看安装包里面是否有Devices Support文件，可以手动拷贝其他人的高版本的xcode里面的Devices Support文件。
[github]("https://github.com/iGhibli/iOS-DeviceSupport") 有人已经放了一堆了，可以下载下来，解压到`Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport`目录中。或者可以使用它提供的deploy.py文件批量将repo中所有文件解压进去。

### 2.不支持调试的情况
证书不是development证书，而是发布证书。