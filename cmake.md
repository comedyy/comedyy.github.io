### CMake简单介绍

因为开源代码很多使用 `cmake` 配置，所以使用初略的使用了一下，顺便记录。

主要用于c++



#### 跨平台的需求

一份c++代码编译成exe文件需要经过

1. compile
2. link

这两个过程需要用到各种的 头文件跟lib，这些头文件跟lib文件在各个操作系统上都有区别。

而且用到的编译器也不一定一样，linux上使用make来编译，windows下使用visualstudio编译。



通过`cmake` 就可以把一个`cmake`工程 转成 `make`工程之后，你执行make命令就可以编译。

或者转成visual studio工程，使用ide来编译。



#### 相关命令

一般下载下来的工程在根目录下有一个`CMakeLists.txt`文件，以及源文件。

先创建一个build目录，把生成的中间文件跟最终文件放在那里。

```
mkdir build
cd build
// windows 下
cmake <path/to/the/source/of/MPQExtractor> -G "Visual Studio 15 2017 Win64" 
或者 cmake <path/to/the/source/of/MPQExtractor> -G "Visual Studio 16 2019" 
然后使用visual studio 进行编译

// linux
$ cmake <path/to/the/source/of/MPQExtractor>
然后开始编译
```



