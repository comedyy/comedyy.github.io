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


### 脚本相关
- 去掉代码里面的OnMouse_的回调，只要它在脚本中，无论是否挂到GameObject上，它都会在设备上产生OnMouse_的消耗。

### animator消耗过多。
- 控制animator的数量
- 注意animator的cullingmode。有些animator对象缓冲池子把它放到了屏幕外，但是这个culling没设置，就会大量消耗。

### ui相关
- 如果是CanvasUpate.PreRender，那么大概率是ui刷新了，重新生成网格，需要注意不要每帧刷新。
- EventSystem消耗问题，它的流程是 1. 遍历所有的raycaster，找出需要处理的ui，2. 对这个ui抛出drag消息。 主要消耗再第一部分，但是第二部分的消耗需要拆分出来，放到逻辑处理模块做。
- ugui需要重新生成网格，如果ui的元素重绘，或者是元素的位置，缩放，旋转改变了的话。 我理解的是unity会把ui元素的子mesh缝合起来，做一个缝合怪。位置或者子网格变了的话，缝合怪也需要重新制作mesh。


### 特效相关
- 如果的话，跟animator一样，都有一个culling选项。默认是automatic，这个是指如果是loop特效在屏幕外就停止simulate，非loop的还会继续simulate。如果想要在屏幕外的特希奥直接停止，那么用pause这个culling选型。
- 一次性特效一般情况下是要有个脚本来控制它被放入缓存池子。这个需要把它弄精确。 => 把duration 时间 = startLifeTime,这样应该可以更好的控制粒子的生命周期。
- 我看很多特效有一个空的root特效，这个空的特效同样消耗性能。因为它没有renderer，所以它不会被cull，所以每帧都在simulate，如果在屏幕外的话。但是duration到了之后，同样会停止消耗。
- 使用ParticleSystem.Stop(true, StopEmitAndClear) 可以让ParticleSystem（粒子发射器）完全没有了消耗（simulation跟render）。
    - particlesystem有个duration是粒子发射器的生命周期。 还有一个startLifeTime， 这个是粒子的生命周期。当这两个周期都走完了之后，粒子才完全无消耗
    - 所以需要好好设置好duration, 实际测试中，iphone6s上的特效性能提升1ms（冰飞弹）
    - 发现空的粒子发射器似乎有setpass？确实如此，duration如果还在的话，就还有setpass消耗。但是没有render的消耗。
- 所以是否可以考虑弄一个特效pool。它就放在很远的地方，然后它不关闭，只stop。
- prewarm需要好好的评估，不一定所有的loop特效都需要。它有性能问题。
- 特效氛围procudual模式跟nonprocedual模式，nonprocedual会无法被cull。
- 试试particlesystem的dynamicbatching，思路是给每个特效弄一个orderinlayer，这样提升DynamicBatch的能力。