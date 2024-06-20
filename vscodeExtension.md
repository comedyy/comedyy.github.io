### vscode 插件
1. 熟悉插件：我弄fork了两个项目。
`https://github.com/comedyy/vscode-csproj` 使用的是ts，修改了编译错误，默认增加把cs文件添加到`Compile`标签中。
`https://github.com/comedyy/colonize` 使用的是js，增加了泛型支持。

2. 如何打包。
    1. 需要安装nodejs
    2. npm i vsce -g, 安装vsce, 用来把工程打包成插件文件。
    3. vsce package 使用vsce工具来编译工程目录下package.json文件。输出插件文件。

3. 使用插件。vscode 在插件那里可以使用本地插件安装。

说明：
ts就是 nodejs的一个插件，如果项目是ts的话，需要安装ts插件。 可能还有很多其他插件。跟python很像，使用 `npm i xxx -g` 方式来安装。
