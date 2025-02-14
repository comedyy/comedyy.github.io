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
    

