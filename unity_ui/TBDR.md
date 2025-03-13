# Tile Base Deferred Render

1. 将屏幕划分为N*N的块，每一块是一个Tile，每个块独立在Gpu Core中完成Fragment绘制。
2. Vertex跟Fragment是分开执行的
    1. 所有的Vertex执行完，然后执行光栅化，进一步EarlyZ,然后每个tile记录它需要渲染哪些三角形跟材质。
    2. 每个GpuCore执行一个Tile的像素着色，如果一个设备GpuCore多，就着色快。每个tile中的Color，Depth，Stencil都是每个tile私有的，不需要在一个像素渲染结束之后回写内存，而只要记录在Tile的memory中。（渲染结束，或者其它tile需要我们tile的数据才回写内存），它写入系统内存是一整个tile同时写回的。
    3. vertex阶段执行完（drawcall完了，或者内存达到系统限制），转入到fragment阶段。然后再到vertex阶段等待或者处理。

4. LoadAction : 从 系统内存加载数据到TileMemory。
   StoreAction：从 TileMemory加载数据到系统内存。
这些都是很消耗内存带宽的，所以尽量避免，而TBDR正好是可以减少这些操作。减少耗电跟延迟。

5. TBDR 如何减少带宽消耗以及减小发热和性能消耗。
    1. 传统 IMR 架构需要频繁读写显存中的深度和颜色缓冲，而 TBDR 仅在处理完整个 Tile 后一次性写入结果，避免逐像素的显存访问。 Fragment 阶段都存在Tile Memory中。
    2. EarlyZ（HSR）
    3. ASTC
    4. 贴图采样效率高？？？

gpu时间超过10ms手机发热的概率很大。

Tile memory：用来存放tile的color，depth，stencil
纹理缓存： 用来存档texture采样的数据。 使用astc，或者使用mipmap的时候，更小尺寸的texure可以更好的命中缓存。