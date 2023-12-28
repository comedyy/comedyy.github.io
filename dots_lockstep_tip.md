## 注意事项
1. hash的计算时候，需要同时把entity的hash加入进去一起计算，用于检测Entity的顺序乱掉问题，从而发现找出entity乱序的原因。
2. 不能在表现层操作entity对象，这个会导致entity在chunk中的顺序不一致，从而导致hash不一致。
3. job中注意一定不能同时修改相同内存，一个情况就是如果你的对象保存的rvo的id是相同的（id可回收的），那么一定注意要把死亡的对象移除，不然同时写入相同内存，会导致rvo的不一致。
4. 逻辑代码里面一定不要参杂表现的对象，比如判断表现是否播放完成。
5. 不要再job中访问静态对象，并且对他们进行操作，需要加锁。
6. burst的开关一定要相同，pc包跟editor的burst配置相同。
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
