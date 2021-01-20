### git lfs

为了更好的适应大文件而推出的一个git配套工具，是个独立应用软件，不过新版本的git中已经内置了它。

各大GUI版本的git一般都包含它。 使用前执行`git lfs install`,它会添加配置到`.gitconfig`中。



#### 适用场景

git本来是用来处理代码的，随着越来越多的大文件的加入，就需要使用`lfs`来优化体验。

项目如果做了很久了，那么clone一次工程需要很多的时间。



`lfs`就是为此而生，它在历史记录中，把大文件（用户自己标记的）替换成一个文本指针，然后clone只clone这些文本指针，当checkout的时候，才把这些指针真正替换成大文件。 然而这些大文件并不跟着git工程一起clone，它需要去服务器上重新下载。



做个类比，很像一个网站。

`lfs`标记的大文件就像放在CDN上的，尚未使用的网页贴图。而git中的内容则是网页页面html。当打开对应的网页的时候，才从图片服务器上下载对应的图片。完成展示，否者不需要把所有的网站都下载下来。



有几种资源不会被立刻下载：

1. 其他分支的`lfs`标记的大资源
2. 历史记录中的`lfs`标记的打资源。

只有当真正切到那个git节点的时候，才会去下载那个git节点的大资源。



#### 内部实现

在.git目录中，git的对象放在object目录下，而`lfs`对象单独放在`lfs`文件目录下。（因为git基于hash来存储，所以多份相同的资源在git和`lfs`中都是一份存储）

在工程中，我们需要标记哪些文件放在`lfs`中，哪些文件放在git中。

```
# 在工程中执行这个命令，就可以在.gitattribute加入 JJJ/** filter=lfs diff=lfs merge=lfs -text 这一行。
# 同时给当前工厂添加各种hook，来应对push，pull等操作。
git lfs track "JJJ**

===》 .gitattribute
JJJ/** filter=lfs diff=lfs merge=lfs -text
*.a  filter=lfs diff=lfs merge=lfs -text
```

你可以使用命令或者直接在`.gitattribute` 文件中添加来配置。



#### 注意事项

你需要尽量早的把需要track的文件通配符放到 `.gitattribute`中，这样在提交的时候才会被识别成`lfs`对象，从而在commit中被替换成文本指针。

否者如果你的大文件已经提交到commit中了，那么它还是放在git工程中，就算你下次把它 track了，但是历史记录中还是会有一份这个大文件。同时`lfs`中也会有一份。







