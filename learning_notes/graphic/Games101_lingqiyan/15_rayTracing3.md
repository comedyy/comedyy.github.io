## 空间划分
1. uniform grid
2. 空间划分(Spatical partitions)
    1. kdtree，不适合的原因是因为可能会有同一个对象被划分到多个子节点中。
3. 物体划分
    bvh(Bounding Volume Hierachy)：比较像松散结构的octree，就是要把物体的aabb划分到对应的节点中。但是它只有子节点才能存储数据。

## 取第K大的数，o(n)的复杂度
使用快速排序法，来快速定位第K大的数。
0. 先分把数组分两半，A跟B
1. 如果B的长度比K大，就直接在B中继续快速排序，A就不用排序了，
2. 如果B的长度比K小，那么直接在A中排序，B不排序，
3. 如果找到的中间数正好是第K大，那么返回第K大的数。

## 辐射度量学
光源的传播过程中，计算任意时刻的能量。
    1. 球的表面积 = 4r*r*pi
    2. radiant flux：单位时间的能量（w）
    3. radiant intensity： 单位立体角的能量
    4. 立体角（steraradius）： 表面积 / R / R， 一个球一共有4Pi的立体角，它跟半径R无关。
    5. 弧度（radius）： l/r = 2Pi, 圆有2Pi的弧度角度
    6. 单位立体角的亮度： 总flux / 4pi