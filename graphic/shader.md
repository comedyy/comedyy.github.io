## shader

0. 语言可以使用 HLSLPROGRAM 或者 CGPROGRAM， unity以后都是使用HLSLPROGRAM,但是兼容CGPROGRAM，

HLSLPROGRAM语法：
```
#pragma vertex vert
#pragma fragment frag  // 定义入口
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl" // 核心库
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"  // include文件
#define kDieletricSpec half4(0.04, 0.04, 0.04, 1.0 - 0.04)  // 定义常量
```
申明：
1. texture: TEXTURE2D(_BaseMap); SAMPLER(sampler_BaseMap); 对应的是一个贴图跟一个贴图的采样器。 为啥需要采样器，因为图片在unity导入的时候都会定义采样方式。
2. texture_ST: float4 _BaseMap_ST; 申明这个sprite对应于texture的相对位置，和TRANSFORM_TEX配置使用
3. 各种面板属性：
    float _BumpScale;
    float _Cutoff;
    float _Metallic;
    float _Smoothness
4. 申明appdata，以及vertex输出结构体。
    // 这个一般是
    struct appdata
    {
        float4 positionOS : POSITION;   // 顶点位置
        float2 texcoord : TEXCOORD0;    // 顶点uv
        float3 normalOS     : NORMAL;   // 顶点normal
        float4 tangentOS : TANGENT;     // 顶点tangent
        float2 lightmapUV   : TEXCOORD1; // lightMapUv
    };

    输出结构体：
    
    struct Varyings
    {
        float4 positionCS : SV_POSITION;  // Clip 空间的坐标，必须的, 通过TransformObjectToHClip()函数得到的，就是mvp函数。得到的结果 是在一个[-1]
        float2 uv : TEXCOORD0; // 顶点的真实uv，用来采样用
        // 下面这些就随意了，根据自己的逻辑添加。
        DECLARE_LIGHTMAP_OR_SH(lightmapUV, vertexSH, 1);
        float3 normalWS : TEXCOORD2;
        float4 tangentWS : TEXCOORD3;
        //loat3 bitangentWS : TEXCOORD3;
        float3 positionWS : TEXCOORD4;
        float3 viewDirWS                : TEXCOORD5;
    };
5. TEXTURE2D(_BaseMap);SAMPLER(sampler_BaseMap); 对应于贴图的申明和采样。

1. TRANSFORM_TEX 这个是实现什么逻辑。
    通常我们使用一张图片`Tex`，那么我们就可以声明另外一个变量，`float4 Tex_ST`，它是个float4，定义了一个sprite位于图片的位置。可以用来做图集，或者只显示图片的某个区域。
    然后我们通常传入uv通常是对应着这个sprite的uv，我们需要转成这个`Tex`的uv。
    就是用 TRANSFORM_TEX 来进行一次转化，它实现的逻辑是 uv = Tex_ST.xy * uv + Tex_ST.zw.

2. SRP中，可以使用 CBUFFER_START(UnityPerMaterial) 来定义一些material 共有的常量： ❓

3. frag shader往往最复杂。

4. 对比一下直接normal 和 normal贴图是怎么处理的。

5. srp中很多宏，c语言的类型的，头大，不同平台实现方式都不一样。
    比如：TEXTURE2D() SAMPLER() SAMPLE_TEXTURE2D() 



1. 开始阅读一下源码，ugui的源码开始，从shader sample开始。