## 包体破解相关文章。
1. 新项目开始，开始做slg，开始各种解包分析别人的逻辑。。
    工具先来：
        1. AssetStudio: 可以查看assetBundle里面有哪些资源（导出资源只能是正常的导出）
        2. AssetRipper：可以以正常的文件夹目录导出资源。
        3. luac（官方）: 用来加密lua变成字节码，unluac（github）可以用来解密。 字节码是用来减少游戏的compile时间的。
            luac54.exe xxx.lua      // 变成字节码
            java -jar ./unluac_2023_12_24.jar luac.out > tt.lua  // 变成源码。
        4. il2cppDumper: 用来通过global-metadata.dat 和 libil2cpp.so 来导出游戏的代码框dll。
        5. ilspy： 把dll导出成代码。
