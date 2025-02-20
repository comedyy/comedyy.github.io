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

        这些回调是给外部 注册的，来实现一些效果。
        OnBeginCameraRendering 可以用来设置一些参数，比如设置天空盒，fog，之类的。

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
    1. 你可以自定义pass， 放在renderfeature中，然后它就会加入进去进行渲染。（比如迷雾）
    2. 自定义VolumeComponent， 来自定义一个后处理。
    3. 你还可以做一个新的renderer。比如2DRenderer，来添加，修改，删除一些流程。 比如你可以不绘制skybox😂。你需要把universalRenderData改成Renderer2DData，就可以了。
    
9. 结构： urp分为urp跟srp core.
    srp core, urp 扩展它。
    

9. ShaderLibrary
    

10. Rendering Debugger 用于监控渲染的窗口
    1. 

11. CommandBuffer.EnableEnableShaderKeyword("DEBUG_DISPLAY") 打开全局的shader macro
    CommandBuffer.SetGlobalFloat(Shader.PropertyToID("_DebugMaterialMode"), 1);  这个
    ```
        int _DebugMaterialMode; 这个定义在DebuggingCommon.hlsl中
    ```


struct Attributes  // 顶点输入。
{
    float4 positionOS    : POSITION;
    float3 normalOS      : NORMAL;
    float4 tangentOS     : TANGENT;
    float2 texcoord      : TEXCOORD0;
    float2 staticLightmapUV    : TEXCOORD1;
    float2 dynamicLightmapUV    : TEXCOORD2;    // ???
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

顶点着色器：



struct Varyings // 输入到fragment中的参数
{
    float2 uv                       : TEXCOORD0;

    float3 positionWS                  : TEXCOORD1;    // xyz: posWS

    #ifdef _NORMALMAP
        half4 normalWS                 : TEXCOORD2;    // xyz: normal, w: viewDir.x
        half4 tangentWS                : TEXCOORD3;    // xyz: tangent, w: viewDir.y
        half4 bitangentWS              : TEXCOORD4;    // xyz: bitangent, w: viewDir.z
    #else
        half3  normalWS                : TEXCOORD2;
    #endif

    #ifdef _ADDITIONAL_LIGHTS_VERTEX
        half4 fogFactorAndVertexLight  : TEXCOORD5; // x: fogFactor, yzw: vertex light
    #else
        half  fogFactor                 : TEXCOORD5; // 通过雾的参数来计算出当前顶点雾的浓度。
    #endif

    #if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
        float4 shadowCoord             : TEXCOORD6;
    #endif

    DECLARE_LIGHTMAP_OR_SH(staticLightmapUV, vertexSH, 7);

#ifdef DYNAMICLIGHTMAP_ON
    float2  dynamicLightmapUV : TEXCOORD8; // Dynamic lightmap UVs
#endif

    float4 positionCS                  : SV_POSITION;
    UNITY_VERTEX_INPUT_INSTANCE_ID
    UNITY_VERTEX_OUTPUT_STEREO
};

12. urp shader的editor
    1. Surface Type: Transparent vs Opaque. 主要是alpha输出多少，如果是opaque而且没有alphatest的时候，直接输出1.0，Transparent的情况输出正常alpha。
    2. RenderFace: 就是Cull选项。
    3. AlphaClipping: 增加_ALPHATEST_ON选项， Threshold：底层并不是真正使用clip函数来实现，而是使用alpha来实现。
    4. receiveShadow: 通过控制整个开关： _RECEIVE_SHADOWS_OFF， 来实现是否渲染shadow到它身上
    _NORMALMAP(有_BumpMap的时候就会被设置), _EMISSION(emission color被设置的时候会打开)

    InputData:  
    {
        positionWS, normalWS, viewDirectionWS, 
        shadowCoord, 如果定义MAIN_LIGHT_CALCULATE_SHADOWS，去计算。定义REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR，直接取input.shadowCoord,否者返回0
        fogCoord,  // 雾的深度百分比。  
        vertexLighting, 有 _ADDITIONAL_LIGHTS_VERTEX，返回input.fogFactorAndVertexLight.yzw，否者（0，0，0）
        bakedGI(烘焙的灯光颜色), 
        normalizedScreenSpaceUV, 
        shadowMask， 1.烘焙情况下。去烘焙贴图取，2. 没有烘焙的话 返回 unity_ProbesOcclusion，3.否和返回（1，1，1，1）
    }
    SurfaceData: InitializeSimpleLitSurfaceData
    {
        albedo, specular, metallic, smoothness, normalTS, emission, occlusion, alpha, clearCoatMask, clearCoatSmoothness
    }

    struct Light
    {
        half3   direction;
        half3   color;
        float   distanceAttenuation; // full-float precision required on some platforms， 如果光线被裁剪，变成0，否者是1. unity_LightData.z
        half    shadowAttenuation; // shadow衰减。
        uint    layerMask;
    };  

    _ALPHAPREMULTIPLY_ON：blinnphong中，alpha可以乘以 albedo。
    _SPECGLOSSMAP 或者 _SPECULAR_COLOR： 是否需要计算高光。
    
    使用InputData和SurfaceData来进行光照计算。
    我看看他们如何使用这些材料进行计算。
    1. 计算ssao如果有的话，没有的话就默认。需要开启 _SCREEN_SPACE_OCCLUSION， !_SURFACE_TYPE_TRANSPARENT. 才可以，去使用normalizedScreenSpaceUV去_ScreenSpaceOcclusionTexture 采样。 获得一个
        AmbientOcclusionFactor{ 
            indirectAmbientOcclusion, // 跟需要比surfacedata中的occlusion小。(simple lit 中，occlusion = 1)
            directAmbientOcclusion   // 通过 lerp(half(1.0), ssao, _AmbientOcclusionParam.w); 获得。 _AmbientOcclusionParam 应该是程序赋值。
        }
    2. GetMainLight() 获取主光源。
        正常主光源就是从方向光来的，unity那里设置的，distanceAttenuation 在forward中，如果光被剔除掉落，值是0，否者是1.
        shadowAttenuation 1.0 ， layermask 用来干嘛？
        通过 MainLightShadow() 去计算 shadowAttenuation， 阴影的衰减把。 
            实时阴影：
                1. （未开启阴影）没有定义 MAIN_LIGHT_CALCULATE_SHADOWS 就 返回1.0，
                2. （光照贴图）定义_MAIN_LIGHT_SHADOWS_SCREEN 和 !_SURFACE_TYPE_TRANSPARENT,就 SampleScreenSpaceShadowmap(), 去采样 shadowmap，_ScreenSpaceShadowmapTexture
                3. （实时阴影）否者去采样 _MainLightShadowmapTexture， 是主灯源的shadowmap。
            bake的阴影
            mainlight shadow。？？
            最后三个阴影混合，如果 LIGHTMAP_SHADOW_MIXING，就进行混合，不然就直接lerp？
        
    3. CalculateBlinnPhong() 通过blinnphong计算， 通过lightcolor， light.distanceAttenuation * light.shadowAttenuation, 来计算最后的lightcolor。
        计算主灯光的blinnphong颜色（mainLightColor），然后一次计算所有的额外灯光，（additionalLightsColor），然后计算顶点光，gibaked颜色是烘焙的灯光。

        组装成一个LightingData 对象，然后通过lightingData对象生成最后的颜色。
            1. giColor， mainLightColor， additionalLightsColor， vertexLightingColor， emissionColor 都加起来。
        emissionColor 是直接发射的光的颜色。 不需要跟光照计算的。

    4. 计算fog 通过fogCoord来判断雾的值。
    5. 输出alpha。（通过_Surface选项）

    总结流程。
    1. vertex 阶段，先去计算 Varyings 对象，
         normal, uv, ws, cs, fogFactor, shadowCoord 如果需要的话。
    2. fragment阶段。
        1. 生成InputData（顶点输入来的。）， 
            计算normal， shadowCoord， fogCoord，vertexLighting, bakedGI(通过normal在烘焙的光照贴图中获得？)， shadowMask（通过烘焙的阴影）， 
        2. surfaceData（材质输入）
        3. 计算总的BlinnPhong颜色UniversalFragmentBlinnPhong
            1. 计算aoFactor信息。 indirectAmbientOcclusion, directAmbientOcclusion, 如果有贴图就采样，没有就1
            2. 获取光源信息。Light:direction,distanceAttenuation, shadowAttenuation, color, layerMask.
            3.  主光源 计算 CalculateBlinnPhong
                1. 通过各种衰减算出最后的灯光颜色
                2. Lambert输出漫反射颜色。
                3. 如果有定义高光，就计算高光颜色。
                4. 漫反射*albedo + specular
            4. 多光源遍历遍历上面操作。
            5. 混合所有灯光颜色：giColor， mainLightColor， additionalLightsColor， vertexLightingColor， emissionColor 
            6. 它没有计算ambient，但是它有计算bakedgi。光线探针。
        4. fog计算，
        5. alpha计算。
        

    了解overdraw的那个view是怎么实现的，要怎么做我们才能看到我们的overdraw。
    
    环境光遮蔽： ambient occlusion， 氛围直接遮蔽和间接遮蔽？ directAmbientOcclusion indirectAmbientOcclusion， ssao 是不是screen space ambient occlusion。_AmbientOcclusionParam

    _SCREEN_SPACE_OCCLUSION _SURFACE_TYPE_TRANSPARENT
    _ScreenSpaceOcclusionTexture
    

