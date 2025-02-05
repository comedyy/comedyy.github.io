# shader graph

如果你想入门shader，shadergraph是个好主意。它提供了很多方法。

原理很简单，问题不大，就在普通的shader中，做一些修改。

节点：

channel节点：
1. combine ： 把一堆数字组成一个vector
2. split： 把vector转成一堆数值
3. swizzle：通过一个式子，来把一些vector转成另外，比如(0,1,0,0) 通过(yyyy) 转成了（1,1,1,1)
4. flip： 说是翻转一个vector，如果是（1, 0, 0) 会翻转成（-1, 0, 0)
5. 

artistic 节点
adjustment: 调节颜色
    1. channel mixer: 一个乘法节点
    2. Contrast ： 跟0.5颜色的插值。1的就是原先颜色， 0就是0.5
    3. invert color： 这个才是翻转一个颜色。
    4. Replace Color Node： ？？？？
    5. Hue: ???
    6. Saturation : 调整颜色的饱和度。
    7. White Balance: ???
blend： 混合颜色
    1. Blend：
    

