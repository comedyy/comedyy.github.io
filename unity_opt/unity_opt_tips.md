## unity 性能问题汇总

## 渲染相关
主要存在的问题是
- 绘制任务太多
- 绘制顺序不合理

setpass 会显著增加渲染时间。需要用framedebuger来查看哪些东西导致的。

### Cpu消耗
CPU 这边的工作主要是 Culling，生成drawcall跟提交drawcall到GPU
draw call 太高，drawcall 太多导致了cpu一直忙于提交drawcall，而gpu却在等待。
25k batches/s @ 100% 1GHz CPU ，大量的drawcall会消耗很多的cpu， 而gpu却一直空闲。

- 多线程渲染 可以将cpu的工作分成两个线程，减少主线程的消耗。

### GPU消耗
主要的指标是：Gfx.WaitForPresent

#### 带宽
GPU跟显存的交互的通道，GPU有小块的存储空间，用来加载当前渲染的对象。
每次sampling，需要从显卡中拿去图片（如果缓存失效的话），或者Ztest，alpha blending之类的需要用到这些buffer不在缓存中的话。
- 减少sampling
- 压缩图片
- 使用mipmap 减小远处的物体的图片大小

-- 减少图片的品质（QualitySettings中有选项）如果可以提升帧率，说明瓶颈在带宽

#### 填充率
是fragment shader的工作太多
- shader太复杂
- 屏幕上重复绘制的区域太多

减小分辨率如果帧率提升，说明瓶颈在填充率

### 场景相关

- GPU Skinning， 把繁重的skinning任务移动到任务最轻的vertex shader中
- GPU Instance， 减少drawcall
- 简化模型，去掉不需要的uv，normal等，如果不需要法线贴图，就不需要导入切线
- Mesh Lod 可以减少场景中需要渲染的顶点数量
- Culling Group 可以实现逻辑层的LOD 
- Occlussion Culling 
- Camera LayerCull Distance
- 缩短相机拍摄距离，远处使用雾来防止穿帮
- Particle Culling（注意particle上的感叹号，会告诉你为什么不能cull）

### ui相关

- 使用更多的Canvas切分各个ui
- 动静分离，避免动态的元素引起整个Canvas的mesh重构
- 关闭无用的RayCast Target
- 关闭Canvas来隐藏UI
- 不要修改alpha值来隐藏UI，同样有drawcall，可以使用canvas group。
- 最好使用RectMask2d, 并手动关闭scrollRect的滑动 （RectMask2D太多也会导致Layout消耗太高）
- 点击阻挡最好使用无文字的Text，来减少重绘和drawcall， 而不是透明的图片
- 

### shader相关

- 使用小精度的数据结构
- 减少复杂的运算
- 删除无用的测试代码
- 暴露在外部的数据越少越好（不依赖于外部输入，跟减少if else 类似）
- 减少if else
- 减少Sampling（带宽）
- 使用合适的LOD来做高低端适配
- 尽量使用内置函数（优化的比较好，兼容性比较好)如：LinearRgbToLuminance（在unityCG.cginc）




