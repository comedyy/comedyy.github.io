# git submodule

具体参考git[文档](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

## 创建一个submodule
添加一个submodule，并且执行clone操作。
```
git submodule add git@github.com:comedyy/math_code.git AAAA
```
它会自动添加一个`.gitmodules`文件到根目录，同时在`.git/config`文件中写入子模块的信息，然后再AAA目录下clone子仓库

## 拉取一个submodule

这个时候如果你提交到服务器了，别人要从工程中拉出这些子模块。
```
git submodule init
git submodule update
```
这个时候其他人已经把`.gitmodules`提交到了仓库了，`git submodule init`把submodule的信息写入到`.git/config`, `git submodule update` 则开始拉取文件。

或者如果你还没有clone工程，你可以直接使用下面的命令拉取仓库跟子仓库
```
git clone --recurse-submodules https://github.com/chaconinc/MainProject
```
