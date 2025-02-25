# SRPBatcher

0. mesh等数据会加载的时候传入到显存中。卸载的时候从显存移除，texture也是。
1. unity 会将当前帧用到的（位置朝向之类的）数据打包成一个大的cbuffer。上传到显存。
2. unity还会将 用到的material数据，打包上传显存。
3. 渲染的时候，绑定已经上传好显存的 （unity数据 和 材质数据）。
4. srp的话就会有batch的概念了，


setpass： 这个就是切换渲染状态的数量。
batch: 这个应该就是frame debugger中的值
