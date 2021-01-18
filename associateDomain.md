1. 它的主要功能是使用url来唤起app，而不是使用传统的schema；

   schema形式，比如你在app的schema中添了 snake,那么你就可以通过`snake://` 来唤起你的app，同时你的app可以通过注册`openurl` 这个函数来捕获到其他app传来的信息。

   而Universual Link则不同，它使用一个url来启动一个app，如果你的app还没安装，那么它直接打开网址。好处就是不会跟别人的schema冲突。并且不会被别人冒牌自己的shema，毕竟这个url服务器上配置了这个app的信息。而app中也配置了这个服务器。

   同时，Universual Link也支持在官网上显示app信息。可以直接点击打开app，如果本地有安装app的话。

   使用`continueUserActivity`这个函数来处理其他app传来的信息。

2. 配置方式

   配置apple-app-site-association放在服务器的根目录。

   里面配置你这个服务器中所有的app信息。

   ```
   {
       "applinks": {
           "apps": [],
           "details": [
               {
                   "appID": "Y3L548DK48.com.diaapple.hitball1",
                   "paths": [ "/config/hitball/*" ]
               },
               {
                   "appID": "Y3L548DK48.com.diaapple.hitball2",
                   "paths": [ "/config/hitball2/*" ]
               }
           ]
       }
   }
   
   ```

   差不多就是这样，我们就可以通过`https://serverDomain/config/hitball`来访问到这个app。

3. Xcode配置。

   添加Associate Domain这个功能，然后把服务器地址写进去，`applinks:bufeiniao.cn`，这样应用安装完之后，设备就去服务器上拉去这个应用对应的Universual Link的url。
   
   ![xcode图片](/Users/zhangdunyong/work/comedyy.github.io/img/xcode_associate.png)
   
4. 友盟的微信分享配置

   配置成：https://bufeiniao.cn/config/hitball，确保在你安装了hitball这个app之后，你用苹果浏览器打开这个网址可以看到网站顶部有一个打开app的bar。

