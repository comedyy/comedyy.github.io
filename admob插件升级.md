删除依赖检查器。
保留 PListProcessor 来处理, 添加sknetwork内容。
manifestProcessor 跟 gradleprocessor 去掉
删除Utils.cs
删除android的库。
prebuild 直接删掉, link.xml 放plugins里面可以的。
admob 的库需要只保留ios arm64的。
倒入到 plugins目录下的GoogleAdmob文件夹中。