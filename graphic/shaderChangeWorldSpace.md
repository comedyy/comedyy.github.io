# 世界空间的shader。基于球形状的。

```
    // 基于世界坐标的球性弧度弯曲函数
    float3 ApplyCurvature(float3 positionWS)
    {
        float zDistance = positionWS.z;
        float radius = 100 ; // 弧度的半径
        float angle = zDistance / radius; // 计算角度（弧度制）
        float zzz = radius * sin(angle);
        float yyy = radius * (cos(angle) - 1);
        float3 floorPos = float3(positionWS.x, yyy, zzz);

        positionWS = lerp(float3(0, -radius, 0), floorPos, 1 + positionWS.y / radius);

        return positionWS;
    }
```

Shader "Custom/SphericalUnlitShader"
{
    Properties
    {
        _BaseColor ("Base Color", Color) = (1, 1, 1, 1)
        _MainTex ("Main Texture", 2D) = "white" {}
        _Curvature ("Curvature Strength", Float) = 0.1 // 弯曲强度
    }
    SubShader
    {
        Tags { "RenderPipeline"="UniversalRenderPipeline" }
        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            struct Attributes
            {
                float4 positionOS : POSITION; // 物体空间顶点坐标
                float2 uv : TEXCOORD0; // UV 坐标
            };
            struct Varyings
            {
                float4 positionHCS : SV_POSITION; // 裁剪空间顶点坐标
                float2 uv : TEXCOORD0; // UV 坐标
            };
            TEXTURE2D(_MainTex);
            SAMPLER(sampler_MainTex);
            CBUFFER_START(UnityPerMaterial)
            float4 _BaseColor;
            float4 _MainTex_ST;
            float _Curvature;
            CBUFFER_END
            // 基于视图空间 Z 轴距离的弯曲函数
            float3 ApplyCurvature(float3 positionVS)
            {
                // 获取视图空间中的 Z 轴距离（深度）
                float zDistance = positionVS.z;
                // 应用弯曲公式
                float bendFactor = _Curvature * zDistance * zDistance;
                // 弯曲顶点坐标
                positionVS.y -= bendFactor;
                return positionVS;
            }
            Varyings vert(Attributes IN)
            {
                Varyings OUT;
                // 获取物体空间顶点坐标
                float3 positionOS = IN.positionOS.xyz;
                // 转换为世界坐标
                float3 positionWS = TransformObjectToWorld(positionOS);
                // 转换为视图空间坐标
                float3 positionVS = TransformWorldToView(positionWS);
                // 应用基于视图空间 Z 轴距离的弯曲
                positionVS = ApplyCurvature(positionVS);
                // 转换为裁剪空间坐标
                OUT.positionHCS = TransformWViewToHClip(positionVS);
                // 传递 UV 坐标
                OUT.uv = TRANSFORM_TEX(IN.uv, _MainTex);
                return OUT;
            }
            half4 frag(Varyings IN) : SV_Target
            {
                // 采样纹理
                half4 color = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, IN.uv);
                // 应用基础颜色
                return color * _BaseColor;
            }
            ENDHLSL
        }
    }
}
