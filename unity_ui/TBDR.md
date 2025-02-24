# Tile Base Deferred Render

1. 将屏幕划分为N*N的块，每一块是一个Tile，每个块独立在Gpu Core中完成Fragment绘制。
2. Vertex跟Fragment是分开执行的
    1. 所有的Vertex执行完，然后执行光栅化，进一步EarlyZ,然后每个tile记录它需要渲染哪些三角形跟材质。
    2. 每个GpuCore执行一个Tile的像素着色，如果一个设备GpuCore多，就着色快。每个tile中的Color，Depth，Stencil都是每个tile私有的，不需要在一个像素渲染结束之后回写内存，而只要记录在Tile的memory中。（渲染结束，或者其它tile需要我们tile的数据才回写内存），它写入系统内存是一整个tile同时写回的。
    3. 

4. LoadAction : 从 系统内存加载数据到TileMemory。
   StoreAction：从 TileMemory加载数据到系统内存。
这些都是很消耗内存带宽的，所以尽量避免，而TBDR正好是可以减少这些操作。减少耗电跟延迟。

gpu时间超过10ms手机发热的概率很大。