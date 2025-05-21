## 注意事项
1. hash的计算时候，需要同时把entity的hash加入进去一起计算，用于检测Entity的顺序乱掉问题，从而发现找出entity乱序的原因。
2. 不能在表现层操作entity对象，这个会导致entity在chunk中的顺序不一致，从而导致hash不一致。
3. job中注意一定不能同时修改相同内存，一个情况就是如果你的对象保存的rvo的id是相同的（id可回收的），那么一定注意要把死亡的对象移除，不然同时写入相同内存，会导致rvo的不一致。
4. 逻辑代码里面一定不要参杂表现的对象，比如判断表现是否播放完成。
5. 不要再job中访问静态对象，并且对他们进行操作，需要加锁。
6. burst的开关一定要相同，pc包跟editor的burst配置相同。（主要是float）
7. job的struct对象的是共用的，不能再job中修改，不然会导致不一致。（多线程往同一个地址写）
8. 不要再job中抛事件， 会导致消息顺序不对，进而导致因为随机数不同 效果触发不同。
9. system最好使用jobComponentSystem，并且使用WithReadonly把外部的变量导入到Foreach中，这样减少gc，同时burst加速。
10. job一定要complete。
11. dictionary 的遍历不一定保证正确，遇到一个删除了又添加，添加了又删除了，遍历结果不一样的顺序。
12. nan在iphone6s跟iphone7上的在burst的加持下，行为不一致。
13. 


## 不一致情况排查记录。
1. 发现editor下的MainLogicSync的输出是同步的，但是pc跟Editor不同步。 请检查是否是Editor的Burst开关未打开。
2. 如果发现preMove是正常，preRvo是错误， 请检查move跟lookat操作是否同步。
3. 检查： random， transform，deltaTime, Aync代码。system 一定要放在fixedupdate中。 表现层一定不要修改tranlation等逻辑 static 一定要检查是否有问题。
4. entity的顺序如果不一致，一般是操作的时机不同步。
5. 在job中，random一定要每个entity一个，如果每个job一个，可能出现random这个对象的操作不同步。多线程问题。不能保证都一样多的线程。
6. SuspendController的状态不一定是一致的。
7. 2022的新版本的dots需要增加一个宏 ： ENTITY_STORE_V1 ， 不然会出现不同步。


## c++中的不同步。
如果是release不同步，debug同步，那么就是被编译器优化了。
案例：
1.  (a * a) >> 16 会被clang优化。 clang 不考虑乘法溢出。


## 我们如何检测不同步问题？
    0. 弄个checkSum系统。
    1. 底层框架，正常情况是最后弄一个 system，来检测主要的数据： 血量，位置，朝向，技能cd之类的。生产一个总的checksum，对比就可以了。
    2. 如果想调试 具体什么原因导致的问题，
        1. 就得把所有的属性记录下来，
        2. 就得在关键的函数里面添加 checksum，并记录所有属性。
    3. 这些在单机模式是很简单的，但是在组队联网模式就不容易了。因为你如果一直往服务器发送hash的debug信息，数据会太大。
        1. 我们在客户端和服务器保留一份30帧率的checksum（加上debug信息），服务器也保存着30帧率的checksum信息，
        2. 发生不同步的时候，通知两个设备，让他们把自己的不同步的hash（加上debug信息）传输到一个文件进行对比。
    4. 我们在开始组队的时候，关键的数据需要进行传输，比如二进制，表格，算出一个hash，保存在包体中，这样可以保证因为版本号错误导致的不同步，查找不到原因。
    