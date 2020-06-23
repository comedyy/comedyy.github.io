# adb 一些常用命令

1. `adb connect 127.0.0.1:7555` 连接到端口

2. `adb shell pm list packages` 查看所有包名
    - `adb shell pm list packages -3 ` 查看第三方包名
    - `adb shell dumpsys window | findstr mCurrentFocus` 正在运行的包名跟activity名

3. `adb shell am start -n com.qiroad.mlomru.android/com.road7sdk.unityinterface.Road7SDKActivity` 启动app

4. `adb shell am force-stop com.qiroad.mlomru.android` 关闭app

5. `adb shell dumpsys package com.qiroad.mlomru.android | findstr version ` 查找版本号

6. 模拟输入
    - `adb shell input keyevent 3` 模拟按键
    - `adb shell input text tes` 模拟输入文字
    - `adb shell input tap X Y ` 模拟鼠标点击
    - `adb shell input swipe X1 Y1 X2 Y2` 模拟滑动

7. 推送跟拉取
    - `adb push C:\test.apk /data` push操作
    - `adb pull /data/test.apk C:\` pull操作

8. 截图 跟录像
    - `adb shell screencap /data/screen.png` 截图
    - `adb shell screenrecord /data/test.mp4` 录像

9. 内存分析
    - `adb shell dumpsys meminfo com.the7road.wartune2018.master` [输出内存快照](./memory/dumpsys.md)