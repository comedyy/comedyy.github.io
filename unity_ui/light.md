## unity 烘焙相关

### 场景制作
Light的Mode
- Baked 全部走烘焙贴图
- Mixed 部分实时渲染，部分烘焙贴图
- RealTime 全部实时渲染

对应于Lighting界面有三个选项

- Realtime Lighting 
- Mixed Lighting 
- Lightmapping Settings

其中 Mixed Lighting 有三个选项

- Baked Indirect 只烘焙间接光
- Subtractive 烘焙间接光，直接光，shadow
- Shadowmask 烘焙间接光跟shadow mask信息

lightingmap 记录这个当前的场景的光照信息。
使用shadowmask可额外生成一张shadowmask贴图，它存储着每个像素的灯光遮挡信息。用它来做后续的阴影计算。最多可以存储4个灯光的遮挡信息。

### 高低画质配置。
在QualitySetting中，可以配置当前的阴影信息。

- Disable Shadows 关闭shadow，如果有烘焙贴图，那么使用烘焙贴图。
- Hard Shadows Only 性能比较好
- Hard and Soft Shadows 效果比较好

其中还有shadowmask的选项
- Shadowmask 静态物体使用烘焙贴图，动态物体使用实时阴影
- Distance Shadowmask 动态物体跟静态物体都使用实施阴影

通过调整场景中Light的Mode，可以烘焙出烘焙贴图。

- 低画质，Disable shadow，showdowmask
- 中画质，Hard Shadow Only，shadowmask
- 高画质，Hard and Soft Shadow， distance shadowmask