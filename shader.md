## unity shader

SubShader
{
    <optional: LOD>
    <optional: tags>
    <optional: commands>
    <One or more Pass definitions>
}

Tags: 
    1. 你可以自定义tag的keyvalue，它可以被c#代码访问到。
    2. unity定义了一些keyvalue，用来判断当前shader是否执行。（跟pass里面的tag似乎一样）
    3. 预定义值： RenderPipeline ， Queue ，RenderType ，DisableBatching ，ForceNoShadowCasting ，CanUseSpriteAtlas ，PreviewType 
        1. RenderPipeline： [UniversalPipeline] urp only, [HDRenderPipeline] hrp only, (any other value, or not declared) not support urp or hrp.
        2. Queue: [Background]在背景时候画，[Geometry][AlphaTest][Transparent][Overlay] + offset
        3. RenderType: 在build-in pipeline的replacement shader中。
        4. ForceNoShadowCasting: [True]  [False]
        5. DisableBatching: [True] [False] [LODFading]
        6. IgnoreProjector：[True] [False]
        7. PreviewType: [Sphere] [Plane] [Skybox]
        8. CanUseSpriteAtlas: [True] [False] 主要针对 Legacy Sprite Packer

Lod: 一个数值，用来程序自定义设置的，可以对某个shader，也可以针对所有的shader进行设置。
    使用shader.maximumLOD 或者  Shader.globalMaximumLOD 来进行设置。

Pass: 
    1. Name "xxxx" 定义一个pass的名字
    2. Tag:
        buildin
            1. LightMode 
            2. PassFlags:
            3. RequireOptions 
    3. shader program
        1. HLSLPROGRAM ENDHLSL  最新使用，所有pipeline都能使用
        2. CGINCLUDE ENDCG，只能buildin pipeline使用
        3. HLSLINCLUDE ENDHLSL； 定义一些常量或者宏， 比如 #define EPSILON （1e-10）
PackageRequirements: 需要有这个包名才起作用

Command： 这些commond同样可以用在pass底下，这样就控制不同的粒度。
    1. AlphaToMask: 主要用与打开msaa的情况下，[On] [Off], 没打开msaa不能开启这个选项
    2. Blend：用于当前颜色如何与背景颜色混合。[Off](默认)， blend会去掉early-z，导致性能问题
    3. BlendOp: 决定Blend的操作，默认是[Add]
    4. ColorMask: 0-7，可以是RGBA各种组合。 作用可以是禁止写入colorbuffer，但是写入stencil buffer。
    5. Conservative：[True] [False]让gpu在判断是否需要渲染一个像素的时候，更加保守，直接都渲染。正常情况gpu会根据图形在像素上的占比来判断是否要渲染当前像素，如果开启，那么只要有占一点，也会渲染这个像素。
    6. Cull: [Back] [Front] [Off] 默认[Back]
    7. Offset: 主要是为了避免zfighting，Offset -1, -1, 针对这个东西离摄像机更近了。
    8. Stencil：模板测试，
    9. UsePass: 可以直接使用其它shader中的一个pass，pass名字要全大写。
    10. GrabPass：只在buildin中起作用，urp中不行。只推荐在原型阶段做，
    11. ZClip：[True] [False] default True
    12. ZTest: [Less] default
    13. ZWrite：[On] [Off] default On

HLSL 如何在unity中使用。
    1. 预处理标签
        1. #include， #include_with_pragmas
        2. #pragma
            1. #pragma vertex, fragment, geometry, hull, domain， 定义一些入口函数。
            2. #pragma multi_compile, shader_feature 定义keyword， 同时还有一些 shader_feature_local之类的。 _local 的话只针对当前的shader的关键字，而不是全局的，shader_feature是全局的。
            3. #pragma target, require ， 默认是 target 2.5， require derivatives. shader需要机器的shader 支持什么版本。
            4. #pragma only_renderers, exclude_renderers 
            5. #pragma instancing_options, 
            6. #pragma enable_d3d11_debug_symbols 生产debug信息。
            7. #pragma skip_optimizations 跳过优化
            8. #pragma disable_fastmath meta平台需要，更好处理nan
            9. #pragma editor_sync_compilation 需要是编辑器上sync compile
            10.#pragma enable_cbuffer ： enable cbuffer 就算平台不支持cbuffer
            11. 一些预定义快捷group。multi_compile_fog之类的，使用#pragma multi_compile_fog 等同于 #pragma FOG_LINEAR, FOG_EXP, FOG_EXP2
        3. shader semantics
            0. fragment输出的时候，可以是个 SV_Target, 也可以是个结构体，这个结构体可以包含 SV_Target1,SV_Target2，之类的，看输出到哪个target上，（ Multiple Render Targets rendering technique）， 同时还可以输出 SV_Depth，用处是： 正常情况depth信息是在光栅化的时候写入的，但是如果你想像素着色的时候写入，也可以。（造成early-z优化的失效）
            1. vertex的输出跟fragment的输入的结构体，不同的shader model可以使用的插值器不同，
                OpenGL ES 2.0 8个
                openGL ES 3.0 16个
                model 4 以上32个。
                为了性能考虑，也不应该传递太多的变量，因为插值也需要性能。
            2. 屏幕空间的输入。VPOS， 需要在shadermodel >= 3.0 才有。这个看起来不错。可以在屏幕中绘制方格。或者可以直接判断在屏幕左边就clip掉？（传入fragment中）
            3. VFACE， 判断是mesh的正面还是mesh的反面。（传入fragment中）
            4. SV_VertexID, 顶点id，传入到顶点着色器中。
            5. shader property； hlsl中会定义很多的property。 需要从其它地方给他赋值。
                1. 通过materialpropertyblock给他赋值。
                2. shader的property会自动给他赋值。 通过修改material暴露出去的值给他修改。
                3. shader global value。
                优先级从上到下。注意的是 有些参数是无法暴露在 material中的，比如array或者materix。需要程序动态赋值。
                shader的property跟hlsl的对应关系是：
                    1. Color 对应的是 float4， half4， fixed4.
                    2. Float 对应的是float，half， fixed
                    3. Texture 对应的是 Sampler2D， ; Cubemaps => samplerCUBE; 3D textures => sampler3D.
                shader中的图片还有其它的property。
                    1. _MainTex_ST ==> 图片的tile跟offset
                    2. _MainTex_TexelSize => 图片大小。 x=1.0/width y=1.0/height z=width w=height
                    3. _MainTex_HDR => 图片hdr信息
                正常情况下如果如果 你是线性颜色空间，而且property中定义的是color，那么传入到hlsl中之后会给你转成线性空间的颜色。因为材质上的颜色都是srgb的颜色。
            6. 顶点着色器的输入。
                1. POSITION, COLOR, NORMAL, TEXCOORD0, TEXCOORDN(1-N), TANGENT, 
                2. TEXCOORD0 可以是float2， 或者float4， 那么如果是float4的时候，对应于2d texture的话是（x, y, 0, 1)
                3. buildin shder 的include 目录。
                    1. UnityGC.cginc，通用的头文件，定义了一些常用函数，以及vertex的输入。appdata_base， appdata_tan，appdata_full，appdata_img
                    2. HLSLSupport.cginc， hlsl的通用文件，自动包含，不需要手动引用。
                    3. UnityShaderVariables.cginc 同样自动包含，不需要手动引用，一些常量
            7. Built-in macro： 
                1. Platform：
                    1. SHADER_API_D3D11， SHADER_API_GLES， SHADER_API_GLES3， SHADER_API_METAL， SHADER_API_VULKAN 之类的。
                    2. SHADER_TARGET，shader的target， 使用#if SHADER_TARGET  > 30  来操作。
                    3. UNITY_VERSION， 202030 表示2022.3
                    4. shader阶段。主要是用来公用一些代码，给vert跟frag通用的代码。
                        1. SHADER_STAGE_VERTEX
                        2. SHADER_STAGE_FRAGMENT
                    5. 平台限制宏
                        1. UNITY_NO_SCREENSPACE_SHADOWS ：那些不支持屏幕shadow的平台，主要是移动平台。
                        2. UNITY_NO_LINEAR_COLORSPACE： 不支持线性空间的平台。
                        3. ...
                    6. shadow map 宏： `UNITY_DECLARE_SHADOWMAP()` `UNITY_SAMPLE_SHADOW()` `UNITY_SAMPLE_SHADOW_PROJ()`
                    7. CBUFFER_START CBUFFER_END, const buffer。
                    8. Texture sampler： UNITY_DECLARE_TEX2D， UNITY_DECLARE_TEX2D
                    9. Surface shader需要处理光照，有下面一些定义：UNITY_PASS_FORWARDBASE， UNITY_PASS_FORWARDADD， UNITY_PASS_DEFERRED， UNITY_PASS_SHADOWCASTER
                    10. UNITY_SHADER_NO_UPGRADE ，不要自动升级。
                    11. 深度贴图的帮助宏：COMPUTE_EYEDEPTH， DECODE_EYEDEPTH， Linear01Depth， UNITY_TRANSFER_DEPTH，UNITY_OUTPUT_DEPTH
            8. Built-in help function,都定义在了UnityCG.cginc
                1. UnityObjectToClipPos,UnityObjectToViewPos 
                2. 通用的函数： WorldSpaceViewDir, ObjSpaceViewDir， ParallaxOffset，Luminance（计算亮度） DecodeLightmap EncodeFloatRGBA（把浮点数弄成rgba），DecodeFloatRGBA（把rgba弄成float） 用途是高精度数据，比如光线追帧数据。
                3. forward渲染的函数：WorldSpaceLightDir ObjSpaceLightDir Shade4PointLights 
                4. screen space的函数 ComputeScreenPos  ComputeGrabScreenPos 
                5. 顶点着色器的：ShadeVertexLights 
            9. Built-in shader variable
                1. UNITY_MATRIX_MVP, UNITY_MATRIX_MV,UNITY_MATRIX_V,UNITY_MATRIX_P,UNITY_MATRIX_VP,UNITY_MATRIX_T_MV, UNITY_MATRIX_IT_MV,unity_ObjectToWorld,unity_WorldToObject
                2. _WorldSpaceCameraPos(相机的世界空间位置)， _ProjectionParams（投影参数yz代表近品面z代表远平面）， _ScreenParams（屏幕参数，xy代表屏幕像素大小，zw代表    1 + 1/width, 1+1/height）
                    1. _ZBufferParams（深度空间）
                    2. unity_OrthoParams，正交相机的参数，x相机的width，y相机的height。
                    3. unity_CameraProjection 投影矩阵
                    4. unity_CameraInvProjection： 投影矩阵逆
                    5. unity_CameraWorldClipPlanes： 数组，裁切平面。
                3. Time:
                    1. _Time 从场景加载好了的时间。（t/20, t, t*2, t*3)
                    2. _SinTime Sine of time: (t/8, t/4, t/2, t).
                    3. _CosTime Cosine of time: (t/8, t/4, t/2, t).
                    4. unity_DeltaTime Delta time: (dt, 1/dt, smoothDt, 1/smoothDt).
                4. Lighting：_LightColor0 ，_WorldSpaceLightPos0，等
                5. Lightmaps：unity_Lightmap unity_LightmapST
                6. Fog and Ambient：unity_AmbientSky unity_AmbientEquator unity_AmbientGround unity_IndirectSpecColor，unity_FogColor， unity_FogParams
                7. Various： _TextureSampleAdd（说是使用alpha8贴图的时候会用到，如果没使用的话不用关心） unity_LODFade（LODGroup的使用）
            10. 各种异常：
                1. unity不支持 2.0h这种代表half的形式，还是会把它当作float
                2. 除法的异常： 除以0 在桌面端gpu中一般跟cpu一样，正无穷，在手机上情况不一致，nan，0，或者其它值。
                3. fixed 精度支持。只有opengles支持，其它的最少已经是half了。
            11. 使用half代替float来提高性能。   Texture2D<half4> _MainTex; 这样可以提高采样的速度可能。
            12. sampler的使用方式：
                1. 正常情况是一个sampler一个texture一一对应的，sampler是从texture的配置读出来的，定义sampler2D _MainTex;就可以直接使用tex2d(_MainTex)来sample了。
                2. 另外一种是，定义一个Texture2D，然后定义对应的sampler。
                    Texture2D _MainTex; 
                    SamplerState sampler_MainTex;
                    _MainTex.Sample(sampler_MainTex, uv);
                    当然其它的贴图你也可以使用sampler_MainTex来sample
                    _SecondTex.Sample(sampler_MainTex, uv);
                3. UNITY_DECLARE_TEX2D(_MainTex); UNITY_DECLARE_TEX2D_NOSAMPLER(_SecondTex); 
                    然后使用 UNITY_SAMPLE_TEX2D(_MainTex, uv);  UNITY_SAMPLE_TEX2D_SAMPLER(_SecondTex, _MainTex, uv); 来sample。
                4. 我们可以同时指定一个sampler的类型。override texture的设定。
                    SamplerState my_point_clamp_sampler; 这个就定义了 sampler类型是 point，并且是clamp的。
                    然后使用 _MainTex.Sample(my_point_clamp_sampler, uv);
        4. 不同平台的一些不同。
            1. renderTexture坐标： opengl跟dx有uv的垂直方向的起点不同，opengl是从下面开始，dx是从上面开始，unity使用opengl的，在dx里面自动翻转
            2. image effects： 
            3. ？？？
            4. half在桌面端都会被转成float
        5. 


    

