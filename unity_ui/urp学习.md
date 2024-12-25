## urp 学习：

urp的代码结构还可以，可以考虑借鉴。

1. asset：
    UniversalRenderPipelineAsset: 一份QualitySetting的值。每个qualitysetting可以使用不同的UniversalRenderPipelineAsset 配置，UniversalRenderPipelineAsset 可以配置一些性能跟效果的参数。
        1. Rendering : 我们知道Unity的渲染氛围Forward，Forward+，延迟渲染， 包含一个 UniversalRenderData 对象。
            1. 包含Filtering, Rendering, RenderPass, Shadows, PostProcess, 以及renderfeature。
            2. 每个相机可以从这里选择它是哪个 Rendering。

        2. Quality： 就是从QualitySetting的配置移动到这里。
        3. Lighint：配置qualitySetting。
        4. Shadow: 配置qualitySeting。
        5. PostProcess: 配置qualitySetting。
        6. 它对应于一个生成一个 UniversualRenderPipeline对象。
        7. 包含一个List<UniversalRenderData>以及它生成的List<UniversalRender>

2. UniversalRenderData： 每个对应一个渲染对象。
    1. Filtering： mask（未知）
    2. Rendering： Forward， Forward+， Deferred
    3. RenderPass: (未知)
    4. Shadows：(未知)
    5. PostProcess: postProcessData； 配置后处理的文件。
    6. RenderFeature： 配置各种feature。（未知）

3. postProcessData: 包含各种postprocess的shader跟贴图。
4. VolumeProfile： 后处理的配置文件。 需要场景有GameObject挂Volume脚本，然后挂上这个后处理。
    包含一个List<VolumeComponent>:它是VolumeProfile的一个具体配置。
5. VolumeComponent: 具体各种后处理配置文件的基类。
    Bloom, ToneMapping等等



6. 生成的对象：
    - UniversalRenderPipeline：由UniversalRenderPipelineAsset的基类函数：InternalCreatePipeline() 生成。同时初始化。
        1. 每帧由引擎调用Render()来完成数据更新。
        2. 当切换UniversalRenderPipelineAsset的时候，应该是重新生成。
    
    - UniversalRender: 由UniversalRenderData创建出来。每个相机都由一个这个对象。可以动态切换。
        包含各种pass在里面，同时还有一个List<ScriptableRenderPass> m_ActiveRenderPassQueue。继承自ScriptableRenderer

    - ScriptableRenderer： UniversalRender的基类。
        包含基础的渲染操作： m_ActiveColorAttachments， m_ActiveDepthAttachment, m_CameraDepthTarget（深度rendertexture）m_CameraColorTarget（颜色rendertexture）

    - ScriptableRenderPass: 一个pass，渲染过程。
        比如 PostProcessPass, TransparentSettingsPass, CopyDepthPass, DepthOnlyPass。各种pass在frameDebuger中都能看得到。
        同时根据配置，看看哪些pass需要加入到UniversalRenderer的m_ActiveRenderPassQueue中。

    - Volume： 用来处理后处理，里面存放着一个VolumeProfile文件。用来处理后处理。
    - VolumeManager: 每个Volume的onEnable向VolumeManager注册自己。OnDisable的时候删除。
        VolumeStack： 这个是比较重要的东西，所有后处理的数据都在这里。这里有所有类型的后处理列表，有默认参数。然后通过外部我们配置的VolumeProfile中的参数来修改这里的。
        比如外部配置了ToneMapping, Bloom,那么每帧stack数据都会先变成默认的值，然后再从VolumeProfile中赋值参数列表过去。
        最后在UniversalRenderer中，如果有开启后处理，就把postProcessPass加入到renderqueue中。
        在postprocess中，去stack中找各个后处理的参数。
    

    - ScriptableRenderContext： 是个unity对象，主要负责各种渲染函数调用。
        BeginRenderPass, BeginScopedRenderPass, cull 等等。

    - RenderPipelineManager: 这个对象我感觉就是维护一个list，是引擎代码，用来保存各种配置的RenderPipeline.
        它有各种回调：beginContextRendering, beginFrameRendering, beginCameraRendering, endCameraRendering, endFrameRendering, endContextRendering, 
        activeRenderPipelineTypeChanged, activeRenderPipelineAssetChanged
        activeRenderPipelineDisposed
        activeRenderPipelineCreated


7. 调用过程：
    通过调用 UniversalRenderPipeline.Render(ScriptableRenderContext renderContext, List<Camera> cameras) 来调用。
    1. 切换HDRState（未知）
    2. 调用 BeginContextRendering()=> RenderPipelineManager.BeginContextRendering. 触发RenderPipelineManager.beginContextRendering，RenderPipelineManager.beginFrameRendering
    3. SetupPerFrameShaderConstants() 设置shader的全局变量。_DitheringTextureInvSize, _DitheringTexture, _RendererColor(未知)
    4. SortCameras() 相机通过depth排序
    5. RenderCameraStack()
        1. 计算anyPostProcessingEnabled=> opengl3.0以及以上，并且它的堆叠相机中有一个包含postprocess。
        2. 判断isStackedRendering => 是否是堆叠渲染。
        3. foreach每个相机，进行渲染，beginCameraRendering
            1. UpdateVolumeFramework => VolumeManager.instance.stack 更新到最新参数。里面有个是否每帧更新，看的不是很明白。（未知）
            2. InitializeCameraData => 获得一个 CameraData，反正这个相机的所有参数都在这里，感觉可以换成下来。除了几个matrix
            3. RenderSingleCamera（）=> 渲染一个相机。
                1. renderer.clear()
                2. renderer.cull()
                3. SetupPerCameraShaderConstants() 设置shader全局变量：_GlossyEnvironmentColor, _GlossyEnvironmentCubeMap, _GlossyEnvironmentCubeMap_HDR,unity_AmbientSky,unity_AmbientEquator, unity_AmbientGround,_SubtractiveShadowColor
                4. additionalCameraData.motionVectorsPersistentData.update()
                5. UpdateTemporalAATargets()
                6. InitializeRenderingData() => 获得renderData。
                7. renderer.AddRenderPasses(ref renderingData); 把renderfeature的pass加入到渲染队列。
                8. renderer.Setup() 把renderer里面的pass加入到渲染队列，包含后处理。
                    这里会根据配置加入所有的pass。有两个pass需要注意，就是绘制opaque跟transparent的pass，他们对应的是 DrawObjectsPass， 仅仅是调用renderContext.DrawRenderers，把culling跟一些渲染配置传过去，由底层的srpbatcher自己去搞定合批和渲染。
                9. renderer.Execute()
                    0. InternalStartRendering
                    1. clear
                    2. setTimeShaderValue
                    3. SortStable pass 排序
                    4. configure pass
                    5. setupLights
                    6. ExecuteBlock(beforeRendering)
                    7. setupCamera => 设置相机的所有参数。
                    8. ExecuteBlock(opaque)
                    8. ExecuteBlock(transparent)
                    9. DrawGizmos()
                    10. AfterRendering
                    11. InternalFinishRendering
                10. context.Submit()
            4. endCameraRendering
    6. 调用 EndContextRendering() => RenderPipelineManager.EndContextRendering(), 触发RenderPipelineManager.endFrameRendering，RenderPipelineManager.endCameraRendering
    
8. 所以看完整个流程，我们发现我们代码层面仅仅是配置一个流程，很多api还是调用unity的，比如DrawObjectsPass。不过大致可以让我们知道unity渲染的一些流程罢了。
