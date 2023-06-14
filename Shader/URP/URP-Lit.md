# URP-Lit
Lit Shader是URP管线内置的用于渲染写实效果的光照着色器，该着色器使用URP中计算量最大的着色模型。  
Lit Shader要正常渲染至少需要保证有ForwardLit、DepthNormals两个Pass，如果要渲染阴影则还需ShadowCaster Pass。
## 一、ForwardLit Pass
Lit Shader的前向渲染Pass，这个Pass包含了大量的shader_feature与multi_compile变体。  

>shader_feature与multi_compile的区别：  
两者的区别在于Unity在最终的版本中不会包括shader_feature着色器的未使用的变体。shader_feature更适合处理从material中设置的关键字，而multi_compile则更适合用来处理从全局代码中设置的关键字。

### 1、ForwardLit Pass的结构
ForwardLit的代码都包含在以下两个hsls文件中，LitInput.hlsl定义了Shader所需要输入的数据变量，LitForwardPass.hlsl则负责Shader的渲染流程。  
```hlsl
#include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
#include "Packages/com.unity.render-pipelines.universal/Shaders/LitForwardPass.hlsl"
```
ForwardLit Pass的结构如下图所示：
![LitShader_1](https://github.com/raincoco/Unity/blob/main/Shader/URP/MdImages/URP-Lit/LitShader_01.png)  

### 2、LitInput.hlsl
(1) LitInput中定义了计算表面光照所需的数据，包含由Properties传入的属性参数和纹理贴图、纹理贴图采样函数、Detail细节添加的相关函数。  
(2) LitInput中还有一个初始化函数InitializeStandardLitSurfaceData，用来初始化这些数据。
```hlsl
inline void InitializeStandardLitSurfaceData(float2 uv, out SurfaceData outSurfaceData)
{
    half4 albedoAlpha = SampleAlbedoAlpha(uv, TEXTURE2D_ARGS(_BaseMap, sampler_BaseMap));
    outSurfaceData.alpha = Alpha(albedoAlpha.a, _BaseColor, _Cutoff);

    half4 specGloss = SampleMetallicSpecGloss(uv, albedoAlpha.a);
    outSurfaceData.albedo = albedoAlpha.rgb * _BaseColor.rgb;

#if _SPECULAR_SETUP
    outSurfaceData.metallic = half(1.0);
    outSurfaceData.specular = specGloss.rgb;
#else
    outSurfaceData.metallic = specGloss.r;
    outSurfaceData.specular = half3(0.0, 0.0, 0.0);
#endif

    outSurfaceData.smoothness = specGloss.a;
    outSurfaceData.normalTS = SampleNormal(uv, TEXTURE2D_ARGS(_BumpMap, sampler_BumpMap), _BumpScale);
    outSurfaceData.occlusion = SampleOcclusion(uv);
    outSurfaceData.emission = SampleEmission(uv, _EmissionColor.rgb, TEXTURE2D_ARGS(_EmissionMap, sampler_EmissionMap));

#if defined(_CLEARCOAT) || defined(_CLEARCOATMAP)
    half2 clearCoat = SampleClearCoat(uv);
    outSurfaceData.clearCoatMask       = clearCoat.r;
    outSurfaceData.clearCoatSmoothness = clearCoat.g;
#else
    outSurfaceData.clearCoatMask       = half(0.0);
    outSurfaceData.clearCoatSmoothness = half(0.0);
#endif

#if defined(_DETAIL)
    half detailMask = SAMPLE_TEXTURE2D(_DetailMask, sampler_DetailMask, uv).a;
    float2 detailUv = uv * _DetailAlbedoMap_ST.xy + _DetailAlbedoMap_ST.zw;
    outSurfaceData.albedo = ApplyDetailAlbedo(detailUv, outSurfaceData.albedo, detailMask);
    outSurfaceData.normalTS = ApplyDetailNormal(detailUv, outSurfaceData.normalTS, detailMask);
#endif
}
```
### 3、LitForwardPass.hlsl
LitForwardPass控制着色器的渲染的流程，这里的结构和BuildIn着色器的结构是相似的。
#### 3.1 struct Attributes 顶点着色器输入结构体
```hlsl
struct Attributes
{
    float4 positionOS   : POSITION;        // 模型空间中的顶点位置
    float3 normalOS     : NORMAL;          // 模型空间中的法线
    float4 tangentOS    : TANGENT;         // 模型空间中的切线
    float2 texcoord     : TEXCOORD0;
    float2 staticLightmapUV   : TEXCOORD1; // 静态光照贴图
    float2 dynamicLightmapUV  : TEXCOORD2; // 动态光照贴图
    UNITY_VERTEX_INPUT_INSTANCE_ID         // GPU实例化时，顶点属性的索引
};
```
#### 3.2 struct Varyings 顶点着色器输出结构体
```hlsl
struct Varyings
{
    float2 uv                       : TEXCOORD0;

#if defined(REQUIRES_WORLD_SPACE_POS_INTERPOLATOR)
    float3 positionWS               : TEXCOORD1;    // 世界空间中的顶点位置
#endif

    float3 normalWS                 : TEXCOORD2;    // 世界空间中的法线
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR)
    half4 tangentWS                : TEXCOORD3;     // xyz: tangent, w: sign
#endif
    float3 viewDirWS                : TEXCOORD4;    // 世界空间中的观察方向

#ifdef _ADDITIONAL_LIGHTS_VERTEX
    half4 fogFactorAndVertexLight   : TEXCOORD5; // x: fogFactor, yzw: vertex light //雾效衰减因子和顶点光计算
#else
    half  fogFactor                 : TEXCOORD5;    // 雾效衰减因子
#endif

#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
    float4 shadowCoord              : TEXCOORD6;    // 应该是存放提取阴影贴图的uv坐标的
#endif

#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirTS                : TEXCOORD7;    
#endif

    DECLARE_LIGHTMAP_OR_SH(staticLightmapUV, vertexSH, 8);    // 选择使用LightMap还是SH
#ifdef DYNAMICLIGHTMAP_ON
    float2  dynamicLightmapUV : TEXCOORD9;            // Dynamic lightmap UVs
#endif

    float4 positionCS               : SV_POSITION;    // 裁剪空间中的顶点位置
    UNITY_VERTEX_INPUT_INSTANCE_ID
    UNITY_VERTEX_OUTPUT_STEREO
};
```

#### 3.3 Varyings LitPassVertex 顶点着色器

