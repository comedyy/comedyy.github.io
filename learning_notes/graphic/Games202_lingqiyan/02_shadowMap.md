## shadow map

shadowmap 问题：自遮挡问题(精度问题) 解决方式：对比的时候可以加一个误差值（bias）

如果光照的方向垂直平面，那么子遮挡情况比较小，否者比较大。所以可以根据角度来调整bias的值。

#### Percentage Closer Filtering(PCF)

用来实现软引用

1. 实现方式：通过定义一个filter（意思就是一个N*N的平均）来平均当前的阴影值。取N乘以N范围内所有的点的硬阴影值（0或者1），然后把他们平均起来，就得到当前点的值(0-1的浮点数)
2. filter大小的如何取值。和遮挡物距离（blocker distance ）不同，filter不同。
3. filter 跟卷积比较像



#### Percentage Closer Soft Shadows(pcss)

是用PCF的实现，通过判定平均遮挡物的距离，来确定filter大小。

demo game： Dying light

1. 
2.  
3. 



#### Variance Soft Shadow Mapping

加速。



### 切比雪夫不等式

自己查资料





#### sat（prefix sum）

用来快速计算一个范围内的数值和。

方式：

1. 一维： 输入是一个数组，输出是一个同样长度的数组。求任意一段[M,N]的和。输出的数组值 oi = sum(输入数组的前i项)，这样如果要求[M，N]的和，只要用 On - Om
2. 二维：输入时二维数组，输出也是。Oij = sum(输入的[0, 0, i, j] 的数值)。那么[x,y,width, height ] = [0, 0, width + x, height + y] - [0, 0, x, height + y, 0, 0, x + width, y] - [0, 0, x, y] 

  