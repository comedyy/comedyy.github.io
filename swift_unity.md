## unity 使用swift的一些问题。

unity可以直接调用c代码，在oc文件中，extern "C" {} 中的函数可以被识别。
但是如何进一步调用swift代码呢。

https://developer.apple.com/documentation/swift/importing-swift-into-objective-c

1. #include "UnityFramework/UnityFramework-Swift.h"
这个swift的自带的功能，它会给你生成一份从swift 导出的header。但是需要你在swift中，设定特殊的标志。 需要在building-setting中打开module 设置。（DEFINES_MODULE）
    0. 需要 对象继承自 NSObject
    1. 需要 @objc public， 不仅仅是类，导出的函数也需要。
这样调用swift的函数就解决了。


如何从swift中调用oc代码呢。
https://developer.apple.com/documentation/swift/importing-objective-c-into-swift
我们在swift中想要调用oc的 UnitySendMessage(), 这个头文件在UnityInterface.h中。swift无法直接访问。
这个时候需要创建一个modulemap文件，把UnityInterface.h暴露给swift。

具体做法是 BuildingSetting的 DEFINES_MODULE 打开，然后创建一个modulemap 文件。在submodule中把unityInterface.h暴露出去。

framework module UnityFramework {
  umbrella header "UnityFramework.h"

  export *
  module * { export * }

  module UnityInterface {
      header "UnityInterface.h"

      export *
  }
}

然后就可以在 swift中使用了。

不过 后面遇到一个问题：
暴露UnityInterface之后发现xcode编译报错。UnityInterface中的类型被定义多次了。（真几把噩梦，最怕这种定义多次的报错）虽然知道是哪里又多加了头文件，找不到。
后面看了打包日志，发现是unityad里面加了一个没用使用的header。 把它去掉就行了。

