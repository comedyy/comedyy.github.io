# github page 创建过程

# 基本原理
- 当前网页就是使用github page弄的，是markdown格式。把当前网页的后缀修改成.md就可以看到当前网页的源码。markdown格式的网页在page中，自动转为html来显示。
- 当前的[网站](https://github.com/comedyy/comedyy.github.io)也是托管在github上，只要像这样把markdown格式的文件传到自己的一个仓库上就可以
- 为了使用comedyy.github.io这个域名作为根域名，假如你的github用户名是comedyy，那么你需要创建一个名为comedyy.github.io的仓库，同时把网站的所有资源上传到这里。
- 可以在设置里面设置page中的Markdown的样式。仓库Settings里面专门有一个Github Page的设置。

# 过程
1. 需要创建一个 用户名.github.io 的仓库
2. clone这个仓库，并往里面添加index.md，上传到github仓库
3. 往Setting里面设置当前github page的样式
4. 访问 用户名.github.io 就可以看到 index.md 里面的内容了