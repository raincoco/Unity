# URP-Lit
Lit Shader是URP管线内置的用于渲染写实效果的光照着色器，该着色器使用URP中计算量最大的着色模型。  
Lit Shader要正常渲染至少需要保证有ForwardLit、DepthNormals两个Pass，如果要渲染阴影则还需ShadowCaster Pass。
## 1、ForwardLit Pass
Lit Shader的前向渲染Pass，控制着色器的渲染的流程，结构和BuildIn着色器的结构是相似的。这个Pass包含了大量的shader_feature与multi_compile变体。  

>shader_feature与multi_compile的区别：  
两者的区别在于Unity在最终的版本中不会包括shader_feature着色器的未使用的变体。shader_feature更适合处理从material中设置的关键字，而multi_compile则更适合用来处理从全局代码中设置的关键字。

## 2、ForwardLit Pass的结构
ForwardLit的代码都包含在以下两个hsls文件中，`LitInput.hlsl`定义了Shader所需要的输入数据变量，`LitForwardPass.hlsl`则负责Shader的渲染流程。  
```hlsl
#include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
#include "Packages/com.unity.render-pipelines.universal/Shaders/LitForwardPass.hlsl"
```
ForwardLit Pass的结构如下图所示：
![LitShader_1](https://github.com/raincoco/Unity/blob/main/Shader/URP/MdImages/URP-Lit/LitShader_01.png)  

## 3、Attributes And Varyings 顶点着色器输入/输出结构体
### 3.1 Attributes 顶点着色器输入结构体
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
### 3.2 Varyings 顶点着色器输出结构体
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

## 4、LitInput 输入数据
`LitInput.hlsl`内声明了外部输入变量，包含由Properties传入的属性参数和纹理贴图、纹理贴图采样函数、Detail细节添加的相关函数，以及用来初始化模型表面数据的初始化函`数InitializeStandardLitSurfaceData`。

`_SPECULAR_SETUP`宏决定着色器使用的工作流，当`_SPECULAR_SETUP`被定义时使用反射流，未被定义时使用金属流。
```hlsl
#ifdef _SPECULAR_SETUP
    #define SAMPLE_METALLICSPECULAR(uv) SAMPLE_TEXTURE2D(_SpecGlossMap, sampler_SpecGlossMap, uv)
#else
    #define SAMPLE_METALLICSPECULAR(uv) SAMPLE_TEXTURE2D(_MetallicGlossMap, sampler_MetallicGlossMap, uv)
#endif
```

LitInput.hlsl声明的变量：
```hlsl
CBUFFER_START(UnityPerMaterial)
float4 _BaseMap_ST;
float4 _DetailAlbedoMap_ST;
half4 _BaseColor;
half4 _SpecColor;
half4 _EmissionColor;
half _Cutoff;
half _Smoothness;
half _Metallic;
half _BumpScale;
half _Parallax;
half _OcclusionStrength;
half _ClearCoatMask;
half _ClearCoatSmoothness;
half _DetailAlbedoMapScale;
half _DetailNormalMapScale;
half _Surface;
CBUFFER_END
```

LitInput.hlsl声明的采样器：
```hlsl
TEXTURE2D(_ParallaxMap);        SAMPLER(sampler_ParallaxMap);
TEXTURE2D(_OcclusionMap);       SAMPLER(sampler_OcclusionMap);
TEXTURE2D(_DetailMask);         SAMPLER(sampler_DetailMask);
TEXTURE2D(_DetailAlbedoMap);    SAMPLER(sampler_DetailAlbedoMap);
TEXTURE2D(_DetailNormalMap);    SAMPLER(sampler_DetailNormalMap);
TEXTURE2D(_MetallicGlossMap);   SAMPLER(sampler_MetallicGlossMap);
TEXTURE2D(_SpecGlossMap);       SAMPLER(sampler_SpecGlossMap);
TEXTURE2D(_ClearCoatMap);       SAMPLER(sampler_ClearCoatMap);
```

LitInput.hlsl声明的函数：
```hlsl
half4 SampleMetallicSpecGloss(float2 uv, half albedoAlpha)  //采样金属光泽贴图              
half  SampleOcclusion(float2 uv)                            //采样AO贴图
half2 SampleClearCoat(float2 uv)                            //采样ClearCoat透明涂层贴图
void  ApplyPerPixelDisplacement(half3 viewDirTS, inout float2 uv)          //使用逐像素位移，使用视差图
half3 ScaleDetailAlbedo(half3 detailAlbedo, half scale)                    //细节图缩放比
half3 ApplyDetailAlbedo(float2 detailUv, half3 albedo, half detailMask)    //使用基础色细节图
half3 ApplyDetailNormal(float2 detailUv, half3 normalTS, half detailMask)  //使用法线细节图   
inline void InitializeStandardLitSurfaceData(float2 uv, out SurfaceData outSurfaceData)    //初始化表面基础光照数据
```

## 5、Varyings LitPassVertex 顶点着色器
![LitShader_Varyings](https://github.com/raincoco/Unity/blob/main/Shader/URP/MdImages/URP-Lit/LitShader_Varyings.png)
```hlsl
Varyings LitPassVertex(Attributes input)
{
    Varyings output = (Varyings)0;

    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_TRANSFER_INSTANCE_ID(input, output);
    UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);

    VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
    VertexNormalInputs normalInput = GetVertexNormalInputs(input.normalOS, input.tangentOS);

    half3 vertexLight = VertexLighting(vertexInput.positionWS, normalInput.normalWS);

    half fogFactor = 0;
    #if !defined(_FOG_FRAGMENT)
        fogFactor = ComputeFogFactor(vertexInput.positionCS.z);
    #endif

    output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);

    output.normalWS = normalInput.normalWS;
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR) || defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    real sign = input.tangentOS.w * GetOddNegativeScale();
    half4 tangentWS = half4(normalInput.tangentWS.xyz, sign);
#endif
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR)
    output.tangentWS = tangentWS;
#endif

#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(vertexInput.positionWS);
    half3 viewDirTS = GetViewDirectionTangentSpace(tangentWS, output.normalWS, viewDirWS);
    output.viewDirTS = viewDirTS;
#endif

    OUTPUT_LIGHTMAP_UV(input.staticLightmapUV, unity_LightmapST, output.staticLightmapUV);
#ifdef DYNAMICLIGHTMAP_ON
    output.dynamicLightmapUV = input.dynamicLightmapUV.xy * unity_DynamicLightmapST.xy + unity_DynamicLightmapST.zw;
#endif
    OUTPUT_SH(output.normalWS.xyz, output.vertexSH);
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    output.fogFactorAndVertexLight = half4(fogFactor, vertexLight);
#else
    output.fogFactor = fogFactor;
#endif

#if defined(REQUIRES_WORLD_SPACE_POS_INTERPOLATOR)
    output.positionWS = vertexInput.positionWS;
#endif

#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
    output.shadowCoord = GetShadowCoord(vertexInput);
#endif

    output.positionCS = vertexInput.positionCS;

    return output;
}
```

### 5.1 vertexInput 顶点输入
**<VertexPositionInputs>** <br>
顶点空间位置输入结构体，声明了顶点的世界空间位置、观察空间位置、裁剪空间位置，以及（NDC）标准化设备坐标。<br>
结构体声明位置：Core.hlsl
```hlsl
struct VertexPositionInputs
{
    float3 positionWS; // World space position
    float3 positionVS; // View space position
    float4 positionCS; // Homogeneous clip space position
    float4 positionNDC;// Homogeneous normalized device coordinates
};
```

**<GetVertexPositionInputs>** <br>
获取顶点空间位置数据的函数，使用模型空间的顶点位置计算出世界空间、观察空间、裁剪空间、NDC中的顶点位置。<br>
函数声明位置：ShaderVariablesFunctions.hlsl
```hlsl
VertexPositionInputs GetVertexPositionInputs(float3 positionOS)
{
    VertexPositionInputs input;
    input.positionWS = TransformObjectToWorld(positionOS);
    input.positionVS = TransformWorldToView(input.positionWS);
    input.positionCS = TransformWorldToHClip(input.positionWS);

    float4 ndc = input.positionCS * 0.5f;
    input.positionNDC.xy = float2(ndc.x, ndc.y * _ProjectionParams.x) + ndc.w;
    input.positionNDC.zw = input.positionCS.zw;

    return input;
}
```
### 5.2 normalInput 法线输入
**<VertexNormalInputs>** <br>
顶点法线输入结构体，声明了顶点世界空间下的法线、切线、副切线。<br>
结构体声明位置：Core.hlsl
```hlsl
struct VertexNormalInputs
{
    real3 tangentWS;
    real3 bitangentWS;
    float3 normalWS;
};
```
**<GetVertexNormalInputs>** <br>
计算顶点世界空间下的法线、切线、副切线，组成TBN矩阵。<br>
函数声明位置：ShaderVariablesFunctions.hlsl
```hlsl 
VertexNormalInputs GetVertexNormalInputs(float3 normalOS, float4 tangentOS)
{
    VertexNormalInputs tbn;

    // mikkts space compliant. only normalize when extracting normal at frag.
    real sign = real(tangentOS.w) * GetOddNegativeScale();
    tbn.normalWS = TransformObjectToWorldNormal(normalOS);
    tbn.tangentWS = real3(TransformObjectToWorldDir(tangentOS.xyz));
    tbn.bitangentWS = real3(cross(tbn.normalWS, float3(tbn.tangentWS))) * sign;
    return tbn;
}
```

### 5.3 vertexLight 顶点光照
**<VertexLighting>**<br>
顶点光函数，计算主灯光外的其他光照，通常不在顶点着色器中计算多光源光照，所有可以不使用vertexLight。
函数声明位置：Lighting.hlsl
```hlsl
half3 VertexLighting(float3 positionWS, half3 normalWS)
{
    half3 vertexLightColor = half3(0.0, 0.0, 0.0);

#ifdef _ADDITIONAL_LIGHTS_VERTEX
    uint lightsCount = GetAdditionalLightsCount();
    LIGHT_LOOP_BEGIN(lightsCount)
        Light light = GetAdditionalLight(lightIndex, positionWS);
        half3 lightColor = light.color * light.distanceAttenuation;
        vertexLightColor += LightingLambert(lightColor, light.direction, normalWS);
    LIGHT_LOOP_END
#endif

    return vertexLightColor;
}
```
当使用多光源时`_ADDITIONAL_LIGHTS_VERTEX`，fogFactor 和 vertexLight 存入 output.fogFactorAndVertexLight 中。
**GetOddNegativeScale>**<br>
函数声明位置：SpaceTransforms.hlsl
```hlsl
real GetOddNegativeScale()
{
    return unity_WorldTransformParams.w >= 0.0 ? 1.0 : -1.0;
}
```
`unity_WorldTransformParams`声明在`UnityInput.hlsl`中，其w通常为1.0或 -1.0用于奇负尺度变换。

### 5.4 fogFactor 雾效因子
当着色器使用宏 _FOG_FRAGMENT 时，雾效将在片元着色器中计算，fogFactor为0；若未使用宏 _FOG_FRAGMENT，则在顶点着色器中计算顶点雾效因子。
```hlsl
half fogFactor = 0;
#if !defined(_FOG_FRAGMENT)
    fogFactor = ComputeFogFactor(vertexInput.positionCS.z);
#endif
```
ComputeFogFactor函数计算简单的深度。<br>
函数声明位置：ShaderVariablesFunctions.hlsl
```hlsl
real ComputeFogFactorZ0ToFar(float z)
{
    #if defined(FOG_LINEAR)
    // factor = (end-z)/(end-start) = z * (-1/(end-start)) + (end/(end-start))
    float fogFactor = saturate(z * unity_FogParams.z + unity_FogParams.w);
    return real(fogFactor);
    #elif defined(FOG_EXP) || defined(FOG_EXP2)
    // factor = exp(-(density*z)^2)
    // -density * z computed at vertex
    return real(unity_FogParams.x * z);
    #else
        return real(0.0);
    #endif
}

real ComputeFogFactor(float zPositionCS)
{
    float clipZ_0Far = UNITY_Z_0_FAR_FROM_CLIPSPACE(zPositionCS);
    return ComputeFogFactorZ0ToFar(clipZ_0Far);
}
```
如果着色器使用多光源（使用宏_ADDITIONAL_LIGHTS_VERTEX）时，fogFactor 和顶点光 vertexLight 存入 output.fogFactorAndVertexLight 中；<br>
如果着色器未使用多光源，fogFactor 存入 output.fogFactor。
```hlsl
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    output.fogFactorAndVertexLight = half4(fogFactor, vertexLight);
#else
    output.fogFactor = fogFactor;
#endif
```
### 5.5 TRANSFORM_TEX
TRANSFORM_TEX 声明在 Macros.hls 中，用于变换2D UV。
```hlsl
#define TRANSFORM_TEX(tex, name) ((tex.xy) * name##_ST.xy + name##_ST.zw)
```
```hlsl
output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);
----------------------------等价于------------------------------
output.uv = input.texcoord.xy * _BaseMap_ST.xy + _BaseMap_ST.zw;
```

### 5.6 tangentWS And viewDirTS 世界空间切线和切线空间观察矢量
```hlsl
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR) || defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    real sign = input.tangentOS.w * GetOddNegativeScale();      //获取当前平台的切线方向
    half4 tangentWS = half4(normalInput.tangentWS.xyz, sign);   //将获取的切线方向存入切线的w通道
#endif
```
`REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR`    表示需要一个世界空间的切线插值器；<br>
`REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR` 表示需要一个切线空间的观察矢量插值器；<br>
当使用这两个宏中的任意一个时，着色器计算世界空间切线tangentWS。tangentOS.w表用于判断不同平台的切线方向，sign计算了在当前平台切线的方向。

4.6.1 tangentWS <br>
需要世界空间切线时，将tangentWS存入output.tangentWS。
```hlsl
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR)
    output.tangentWS = tangentWS;
#endif
```

4.6.2 viewDirTS 
```hlsl
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(vertexInput.positionWS);
    half3 viewDirTS = GetViewDirectionTangentSpace(tangentWS, output.normalWS, viewDirWS);
    output.viewDirTS = viewDirTS;
#endif
```
`GetWorldSpaceNormalizeViewDir`  获得世界空间观察矢量，函数声明在 ShaderVariablesFunctions.hlsl 中。<br>
`GetViewDirectionTangentSpace`   获得切线空间观察矢量，函数声明在 ParallaxMapping.hlsl 中。<br>

需要切线空间观察矢量时，先调用 `GetWorldSpaceNormalizeViewDir` 计算出世界空间的观察矢量viewDirWS，再调用
 `GetViewDirectionTangentSpace` 计算出 viewDirTS 存入 output.viewDirTS。

### 5.7 LIGHTMAP_UV And SH 光照贴图UV和球谐光
```hlsl
OUTPUT_LIGHTMAP_UV(input.staticLightmapUV, unity_LightmapST, output.staticLightmapUV);
OUTPUT_SH(output.normalWS.xyz, output.vertexSH);
```
OUTPUT_LIGHTMAP_UV 和 OUTPUT_SH 宏声明在 Lighting.hlsl 中:
```hlsl
#if defined(LIGHTMAP_ON)
    #define DECLARE_LIGHTMAP_OR_SH(lmName, shName, index) float2 lmName : TEXCOORD##index
    #define OUTPUT_LIGHTMAP_UV(lightmapUV, lightmapScaleOffset, OUT) OUT.xy = lightmapUV.xy * lightmapScaleOffset.xy + lightmapScaleOffset.zw;
    #define OUTPUT_SH(normalWS, OUT)
#else
    #define DECLARE_LIGHTMAP_OR_SH(lmName, shName, index) half3 shName : TEXCOORD##index
    #define OUTPUT_LIGHTMAP_UV(lightmapUV, lightmapScaleOffset, OUT)
    #define OUTPUT_SH(normalWS, OUT) OUT.xyz = SampleSHVertex(normalWS)
#endif
```
当启用LIGHTMAP时，不计算球谐光SH，OUT输出LIGHTMAP的UV，如下：
```hlsl
OUT.xy = lightmapUV.xy * lightmapScaleOffset.xy + lightmapScaleOffset.zw;
```
当不启用LIGHTMAP时，计算球谐光SH，OUT输出球谐光SH，如下：
```hlsl
OUT.xyz = SampleSHVertex(normalWS)
```
球谐光SH采样函数`SampleSHVertex`声明在 GlobalIllumination.hlsl 中。当启用宏 EVALUATE_SH_VERTEX 时，使用的是完整的球谐光照，计算了L0L1L2；当启用宏 EVALUATE_SH_MIXED 时，球谐光照仅计算了L2。在启用LIGHTMAP的情况下，球谐光为0。
```hlsl
half3 SampleSHVertex(half3 normalWS)
{
#if defined(EVALUATE_SH_VERTEX)
    return SampleSH(normalWS);
#elif defined(EVALUATE_SH_MIXED)
    // no max since this is only L2 contribution
    return SHEvalLinearL2(normalWS, unity_SHBr, unity_SHBg, unity_SHBb, unity_SHC);
#endif

    // Fully per-pixel. Nothing to compute.
    return half3(0.0, 0.0, 0.0);
}
```
### 5.8 DYNAMICLIGHTMAP UV 动态光照贴图UV
```hlsl
output.dynamicLightmapUV = input.dynamicLightmapUV.xy * unity_DynamicLightmapST.xy + unity_DynamicLightmapST.zw;
```
`unity_DynamicLightmapST`声明在'UnityInput.hlsl'中。

### 5.9 shadowCoord 阴影纹理
GetShadowCoord 函数，获取阴影纹理坐标。
函数声明位置：Shadow.hlsl
```hlsl
float4 GetShadowCoord(VertexPositionInputs vertexInput)
{
#if defined(_MAIN_LIGHT_SHADOWS_SCREEN) && !defined(_SURFACE_TYPE_TRANSPARENT)
    return ComputeScreenPos(vertexInput.positionCS);
#else
    return TransformWorldToShadowCoord(vertexInput.positionWS);
#endif
}
```

## 6、LitPassFragment片元着色器
LitForwardPass.hlsl内Lit.shader片元着色器代码：
```hlsl
half4 LitPassFragment(Varyings input) : SV_Target
{
    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input); //XR项目使用

    //视差图计算
    #if defined(_PARALLAXMAP)
    #if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
        half3 viewDirTS = input.viewDirTS;
    #else
        half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);
        half3 viewDirTS = GetViewDirectionTangentSpace(input.tangentWS, input.normalWS, viewDirWS);
    #endif
        ApplyPerPixelDisplacement(viewDirTS, input.uv);
    #endif

    //初始化渲染表面的数据
    SurfaceData surfaceData;
    InitializeStandardLitSurfaceData(input.uv, surfaceData);

    //初始化空间位置和矢量数据
    InputData inputData;
    InitializeInputData(input, surfaceData.normalTS, inputData);
    //设置测试贴图数据(官方Debug的宏)
    SETUP_DEBUG_TEXTURE_DATA(inputData, input.uv, _BaseMap);

    //URP的贴花系统
    #ifdef _DBUFFER
        ApplyDecalToSurfaceData(input.positionCS, surfaceData, inputData);
    #endif

    //PBR渲染
    half4 color = UniversalFragmentPBR(inputData, surfaceData);

    //混合雾效
    color.rgb = MixFog(color.rgb, inputData.fogCoord);
    //输出Alpha
    color.a = OutputAlpha(color.a, _Surface);

    return color;
}
```

### 6.1 UNITY_SETUP_INSTANCE_ID 实例化ID
UNITY_SETUP_INSTANCE_ID是用于记录不同实例属性ID的方法，UNITY_SETUP_INSTANCE_ID(input)可以用来访问全局unity_InstanceID，需放在顶点和片元着色器起始第一行。<br>
如果需要将实例化ID传到片段着色器，则需在顶点着色器中增加UNITY_TRANSFER_INSTANCE_ID(v, o);

### 6.2 viewDir 观察矢量
```hlsl
#if defined(_PARALLAXMAP)
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirTS = input.viewDirTS;
#else
    //世界空间观察矢量
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);
    //切线空间观察矢量
    half3 viewDirTS = GetViewDirectionTangentSpace(input.tangentWS, input.normalWS, viewDirWS);
#endif
    ApplyPerPixelDisplacement(viewDirTS, input.uv);
#endif
```
在声明`_PARALLAXMAP`启用视差图的情况下，根据`REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR`选择是否需要计算切线空间的观察向量。<br>
`REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR`声明：
```hlsl
#if defined(_PARALLAXMAP) && !defined(SHADER_API_GLES)
#define REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR
#endif
```
1）计算世界空间观察矢量viewDirWS
`GetWorldSpaceNormalizeViewDir`计算viewDirWS，该函数定义在：<br>
\Library\PackageCache\com.unity.render-pipelines.universal@12.1.10\ShaderLibrary\ShaderVariablesFunctions.hlsl
```hlsl
float3 GetWorldSpaceNormalizeViewDir(float3 positionWS)
{
    if (IsPerspectiveProjection())
    {
        // Perspective 透视空间
        float3 V = GetCurrentViewPosition() - positionWS;
        return normalize(V);
    }
    else
    {
        // Orthographic 正交空间
        return -GetViewForwardDir();
    }
}
```
2）计算切线空间观察矢量viewDirTS
`GetViewDirectionTangentSpace`计算viewDirTS，该函数定义在：<br>
\Library\PackageCache\com.unity.render-pipelines.core@12.1.10\ShaderLibrary\ParallaxMapping.hlsl
```hlsl
half3 GetViewDirectionTangentSpace(half4 tangentWS, half3 normalWS, half3 viewDirWS)
{
    // must use interpolated tangent, bitangent and normal before they are normalized in the pixel shader.
    half3 unnormalizedNormalWS = normalWS;
    const half renormFactor = 1.0 / length(unnormalizedNormalWS);

    // use bitangent on the fly like in hdrp
    // IMPORTANT! If we ever support Flip on double sided materials ensure bitangent and tangent are NOT flipped.
    half crossSign = (tangentWS.w > 0.0 ? 1.0 : -1.0); // we do not need to multiple GetOddNegativeScale() here, as it is done in vertex shader
    half3 bitang = crossSign * cross(normalWS.xyz, tangentWS.xyz);

    half3 WorldSpaceNormal = renormFactor * normalWS.xyz;       // we want a unit length Normal Vector node in shader graph

    // to preserve mikktspace compliance we use same scale renormFactor as was used on the normal.
    // This is explained in section 2.2 in "surface gradient based bump mapping framework"
    half3 WorldSpaceTangent = renormFactor * tangentWS.xyz;
    half3 WorldSpaceBiTangent = renormFactor * bitang;

    half3x3 tangentSpaceTransform = half3x3(WorldSpaceTangent, WorldSpaceBiTangent, WorldSpaceNormal);
    half3 viewDirTS = mul(tangentSpaceTransform, viewDirWS);

    return viewDirTS;
}
```
### 6.3 PARALLAXMAP 视差图
```hlsl
#if defined(_PARALLAXMAP)
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)  
    half3 viewDirTS = input.viewDirTS;
#else
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);
    half3 viewDirTS = GetViewDirectionTangentSpace(input.tangentWS, input.normalWS, viewDirWS);
#endif
    ApplyPerPixelDisplacement(viewDirTS, input.uv);
#endif
```
URP预设宏`_PARALLAXMAP`启用视差映射，用`ApplyPerPixelDisplacement`函数计算视差图。<br>
* `ApplyPerPixelDisplacement`函数定义：<br>
函数位置：\Library\PackageCache\com.unity.render-pipelines.universal@12.1.10\Shaders\LitInput.hlsl
```hlsl
void ApplyPerPixelDisplacement(half3 viewDirTS, inout float2 uv)
{
#if defined(_PARALLAXMAP)
    uv += ParallaxMapping(TEXTURE2D_ARGS(_ParallaxMap, sampler_ParallaxMap), viewDirTS, _Parallax, uv);
#endif
}
```
* `ParallaxMapping`函数定义：<br>
函数位置：\Library\PackageCache\com.unity.render-pipelines.core@12.1.10\ShaderLibrary\ParallaxMapping.hlsl
```hlsl
float2 ParallaxMapping(TEXTURE2D_PARAM(heightMap, sampler_heightMap), half3 viewDirTS, half scale, float2 uv)
{
    half h = SAMPLE_TEXTURE2D(heightMap, sampler_heightMap, uv).g;
    float2 offset = ParallaxOffset1Step(h, scale, viewDirTS);
    return offset;
}
```
* `ParallaxOffset1Step`函数：<br>
Unity使用的最简单的ParallaxMapping算法。<br>
函数位置：\Library\PackageCache\com.unity.render-pipelines.core@12.1.10\ShaderLibrary\ParallaxMapping.hlsl
```hlsl
half2 ParallaxOffset1Step(half height, half amplitude, half3 viewDirTS)
{
    height = height * amplitude - amplitude / 2.0;
    half3 v = normalize(viewDirTS);
    v.z += 0.42;
    return height * (v.xy / v.z);
}
```

### 6.4 初始化表面数据
初始化模型表面数据和外部输入数据。
```hlsl
SurfaceData surfaceData;
InitializeStandardLitSurfaceData(input.uv, surfaceData);
```
SurfaceData 结构体声明在 SurfaceData.hlsl 中，声明模型如基础色、金属度、法线等基础表面信息变量，如下。
```hlsl
struct SurfaceData
{
    half3 albedo;
    half3 specular;
    half  metallic;
    half  smoothness;
    half3 normalTS;
    half3 emission;
    half  occlusion;
    half  alpha;
    half  clearCoatMask;
    half  clearCoatSmoothness;
};
```
InitializeStandardLitSurfaceData 初始化模型表面的数据信息，定义在 LitInput.hlsl 中，如下。
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
### 6.5 初始化输入数据
InputData 结构体定义在 Input.hlsl 中，定义模型的空间信息和其他输入变量，如下。
```hlsl
struct InputData
{
    float3  positionWS;
    float4  positionCS;
    float3   normalWS;
    half3   viewDirectionWS;
    float4  shadowCoord;
    half    fogCoord;
    half3   vertexLighting;
    half3   bakedGI;
    float2  normalizedScreenSpaceUV;
    half4   shadowMask;
    half3x3 tangentToWorld;

    #if defined(DEBUG_DISPLAY)
    half2   dynamicLightmapUV;
    half2   staticLightmapUV;
    float3  vertexSH;

    half3 brdfDiffuse;
    half3 brdfSpecular;
    float2 uv;
    uint mipCount;

    // texelSize :
    // x = 1 / width
    // y = 1 / height
    // z = width
    // w = height
    float4 texelSize;

    // mipInfo :
    // x = quality settings minStreamingMipLevel
    // y = original mip count for texture
    // z = desired on screen mip level
    // w = loaded mip level
    float4 mipInfo;
    #endif
};
```
InitializeInputData 函数定义在 LitForwardPass.hlsl 中，声明模型的空间信息和其他输入变量，如下。
```hlsl
void InitializeInputData(Varyings input, half3 normalTS, out InputData inputData)
{
    inputData = (InputData)0;

#if defined(REQUIRES_WORLD_SPACE_POS_INTERPOLATOR)
    inputData.positionWS = input.positionWS;
#endif

    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);

//TBN矩阵（切线空间转换到模型空间矩阵）
#if defined(_NORMALMAP) || defined(_DETAIL)
    float sgn = input.tangentWS.w;      // should be either +1 or -1
    float3 bitangent = sgn * cross(input.normalWS.xyz, input.tangentWS.xyz);
    half3x3 tangentToWorld = half3x3(input.tangentWS.xyz, bitangent.xyz, input.normalWS.xyz);

    #if defined(_NORMALMAP)
    inputData.tangentToWorld = tangentToWorld;
    #endif
    //法线图法线转换（切线到世界空间）
    inputData.normalWS = TransformTangentToWorld(normalTS, tangentToWorld);
#else
    inputData.normalWS = input.normalWS;
#endif

    //Normalize世界法线
    inputData.normalWS = NormalizeNormalPerPixel(inputData.normalWS);
    //世界空间观察矢量
    inputData.viewDirectionWS = viewDirWS;

//阴影坐标
#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
    inputData.shadowCoord = input.shadowCoord;
#elif defined(MAIN_LIGHT_CALCULATE_SHADOWS)
    inputData.shadowCoord = TransformWorldToShadowCoord(inputData.positionWS);
#else
    inputData.shadowCoord = float4(0, 0, 0, 0);
#endif

//多光源时的Fog坐标和顶点光
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    inputData.fogCoord = InitializeInputDataFog(float4(input.positionWS, 1.0), input.fogFactorAndVertexLight.x);
    inputData.vertexLighting = input.fogFactorAndVertexLight.yzw;
#else
    inputData.fogCoord = InitializeInputDataFog(float4(input.positionWS, 1.0), input.fogFactor);
#endif

//光照烘焙
#if defined(DYNAMICLIGHTMAP_ON)
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.dynamicLightmapUV, input.vertexSH, inputData.normalWS);
#else
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.vertexSH, inputData.normalWS);
#endif

    //屏幕空间UV（normalize）
    inputData.normalizedScreenSpaceUV = GetNormalizedScreenSpaceUV(input.positionCS);
    inputData.shadowMask = SAMPLE_SHADOWMASK(input.staticLightmapUV);

    #if defined(DEBUG_DISPLAY)
    #if defined(DYNAMICLIGHTMAP_ON)
    inputData.dynamicLightmapUV = input.dynamicLightmapUV;
    #endif
    #if defined(LIGHTMAP_ON)
    inputData.staticLightmapUV = input.staticLightmapUV;
    #else
    inputData.vertexSH = input.vertexSH;
    #endif
    #endif
}
```

### 6.6 PBR光照计算 
Lit的PBR光照计算由函数 UniversalFragmentPBR 完成，该函数定义在 Lighting.hlsl 中，如下：
```hlsl
half4 UniversalFragmentPBR(InputData inputData, SurfaceData surfaceData)
{
    #if defined(_SPECULARHIGHLIGHTS_OFF)
    bool specularHighlightsOff = true;
    #else
    bool specularHighlightsOff = false;
    #endif

    //BRDF相关数据初始化
    BRDFData brdfData;
    InitializeBRDFData(surfaceData, brdfData);

    #if defined(DEBUG_DISPLAY)
    half4 debugColor;

    if (CanDebugOverrideOutputColor(inputData, surfaceData, brdfData, debugColor))
    {
        return debugColor;
    }
    #endif

    // Clear-coat calculation...
    BRDFData brdfDataClearCoat = CreateClearCoatBRDFData(surfaceData, brdfData);
    half4 shadowMask = CalculateShadowMask(inputData);
    AmbientOcclusionFactor aoFactor = CreateAmbientOcclusionFactor(inputData, surfaceData);
    uint meshRenderingLayers = GetMeshRenderingLightLayer();
    Light mainLight = GetMainLight(inputData, shadowMask, aoFactor);

    // NOTE: We don't apply AO to the GI here because it's done in the lighting calculation below...
    MixRealtimeAndBakedGI(mainLight, inputData.normalWS, inputData.bakedGI);

    LightingData lightingData = CreateLightingData(inputData, surfaceData);

    lightingData.giColor = GlobalIllumination(brdfData, brdfDataClearCoat, surfaceData.clearCoatMask,
                                              inputData.bakedGI, aoFactor.indirectAmbientOcclusion, inputData.positionWS,
                                              inputData.normalWS, inputData.viewDirectionWS);

    if (IsMatchingLightLayer(mainLight.layerMask, meshRenderingLayers))
    {
        lightingData.mainLightColor = LightingPhysicallyBased(brdfData, brdfDataClearCoat,
                                                              mainLight,
                                                              inputData.normalWS, inputData.viewDirectionWS,
                                                              surfaceData.clearCoatMask, specularHighlightsOff);
    }

    //获取多光源灯光数量
    #if defined(_ADDITIONAL_LIGHTS)
    uint pixelLightCount = GetAdditionalLightsCount();

    #if USE_CLUSTERED_LIGHTING
    for (uint lightIndex = 0; lightIndex < min(_AdditionalLightsDirectionalCount, MAX_VISIBLE_LIGHTS); lightIndex++)
    {
        Light light = GetAdditionalLight(lightIndex, inputData, shadowMask, aoFactor);

        if (IsMatchingLightLayer(light.layerMask, meshRenderingLayers))
        {
            lightingData.additionalLightsColor += LightingPhysicallyBased(brdfData, brdfDataClearCoat, light,
                                                                          inputData.normalWS, inputData.viewDirectionWS,
                                                                          surfaceData.clearCoatMask, specularHighlightsOff);
        }
    }
    #endif

    //计算多光源时的其他光照
    LIGHT_LOOP_BEGIN(pixelLightCount)
        Light light = GetAdditionalLight(lightIndex, inputData, shadowMask, aoFactor);

        if (IsMatchingLightLayer(light.layerMask, meshRenderingLayers))
        {
            lightingData.additionalLightsColor += LightingPhysicallyBased(brdfData, brdfDataClearCoat, light,
                                                                          inputData.normalWS, inputData.viewDirectionWS,
                                                                          surfaceData.clearCoatMask, specularHighlightsOff);
        }
    LIGHT_LOOP_END
    #endif

    #if defined(_ADDITIONAL_LIGHTS_VERTEX)
    lightingData.vertexLightingColor += inputData.vertexLighting * brdfData.diffuse;
    #endif

    return CalculateFinalColor(lightingData, surfaceData.alpha);
}

half4 UniversalFragmentPBR(InputData inputData, half3 albedo, half metallic, half3 specular,
    half smoothness, half occlusion, half3 emission, half alpha)
{
    SurfaceData surfaceData;

    surfaceData.albedo = albedo;
    surfaceData.specular = specular;
    surfaceData.metallic = metallic;
    surfaceData.smoothness = smoothness;
    surfaceData.normalTS = half3(0, 0, 1);
    surfaceData.emission = emission;
    surfaceData.occlusion = occlusion;
    surfaceData.alpha = alpha;
    surfaceData.clearCoatMask = 0;
    surfaceData.clearCoatSmoothness = 1;

    return UniversalFragmentPBR(inputData, surfaceData);
}
```

### 6.6.1 BRDF相关数据初始化
BRDF相关数据由 InitializeBRDFData 函数和 InitializeBRDFDataDirect 函数完成初始化计算，这两个函数定义在 BRDF.hlsl 中，如下：
```hlsl
inline void InitializeBRDFDataDirect(half3 albedo, half3 diffuse, half3 specular, half reflectivity, half oneMinusReflectivity, half smoothness, inout half alpha, out BRDFData outBRDFData)
{
    outBRDFData = (BRDFData)0;
    outBRDFData.albedo = albedo;
    outBRDFData.diffuse = diffuse;
    outBRDFData.specular = specular;
    outBRDFData.reflectivity = reflectivity;

    outBRDFData.perceptualRoughness = PerceptualSmoothnessToPerceptualRoughness(smoothness);
    outBRDFData.roughness           = max(PerceptualRoughnessToRoughness(outBRDFData.perceptualRoughness), HALF_MIN_SQRT);
    outBRDFData.roughness2          = max(outBRDFData.roughness * outBRDFData.roughness, HALF_MIN);
    outBRDFData.grazingTerm         = saturate(smoothness + reflectivity);
    outBRDFData.normalizationTerm   = outBRDFData.roughness * half(4.0) + half(2.0);
    outBRDFData.roughness2MinusOne  = outBRDFData.roughness2 - half(1.0);

#ifdef _ALPHAPREMULTIPLY_ON
    outBRDFData.diffuse *= alpha;
    alpha = alpha * oneMinusReflectivity + reflectivity; // NOTE: alpha modified and propagated up.
#endif
}

inline void InitializeBRDFData(half3 albedo, half metallic, half3 specular, half smoothness, inout half alpha, out BRDFData outBRDFData)
{
#ifdef _SPECULAR_SETUP
    half reflectivity = ReflectivitySpecular(specular);
    half oneMinusReflectivity = half(1.0) - reflectivity;
    half3 brdfDiffuse = albedo * oneMinusReflectivity;
    half3 brdfSpecular = specular;
#else
    half oneMinusReflectivity = OneMinusReflectivityMetallic(metallic);
    half reflectivity = half(1.0) - oneMinusReflectivity;
    half3 brdfDiffuse = albedo * oneMinusReflectivity;
    half3 brdfSpecular = lerp(kDieletricSpec.rgb, albedo, metallic);
#endif

    InitializeBRDFDataDirect(albedo, brdfDiffuse, brdfSpecular, reflectivity, oneMinusReflectivity, smoothness, alpha, outBRDFData);
}

inline void InitializeBRDFData(inout SurfaceData surfaceData, out BRDFData brdfData)
{
    InitializeBRDFData(surfaceData.albedo, surfaceData.metallic, surfaceData.specular, surfaceData.smoothness, surfaceData.alpha, brdfData);
}
```
### 6.6.2 主灯光数据
结构体 Light 定义了灯光的四个参数，GetMainLight 函数用于获取主灯光数据，该函数定义在 Realtimelights.hlsl 中，如下：
```hlsl
struct Light
{
    half3   direction;            // 灯光方向
    half3   color;                // 灯光颜色
    half    distanceAttenuation;  // 距离衰减
    half    shadowAttenuation;    // 阴影衰减
    uint    layerMask;
};

Light GetMainLight()
{
    Light light;
    light.direction = half3(_MainLightPosition.xyz); // 灯光方向
#if USE_CLUSTERED_LIGHTING
    light.distanceAttenuation = 1.0;                 // 灯光阴影衰减
#else
    light.distanceAttenuation = unity_LightData.z;   // 灯光距离衰减。当没有被剔除掩码剔除时，unity_LightData.z 为 1，否则为 0。
#endif
    light.shadowAttenuation = 1.0;
    light.color = _MainLightColor.rgb;               // 灯光颜色

#ifdef _LIGHT_LAYERS
    light.layerMask = _MainLightLayerMask;
#else
    light.layerMask = DEFAULT_LIGHT_LAYERS;
#endif

    return light;
}

Light GetMainLight(float4 shadowCoord, float3 positionWS, half4 shadowMask)
{
    Light light = GetMainLight();
    // MainLightShadow 混合实时光照和烘焙阴影
    light.shadowAttenuation = MainLightShadow(shadowCoord, positionWS, shadowMask, _MainLightOcclusionProbes); 

    #if defined(_LIGHT_COOKIES)
        real3 cookieColor = SampleMainLightCookie(positionWS);
        light.color *= cookieColor;
    #endif

    return light;
}

Light GetMainLight(InputData inputData, half4 shadowMask, AmbientOcclusionFactor aoFactor)
{
    Light light = GetMainLight(inputData.shadowCoord, inputData.positionWS, shadowMask);

    #if defined(_SCREEN_SPACE_OCCLUSION) && !defined(_SURFACE_TYPE_TRANSPARENT)
    if (IsLightingFeatureEnabled(DEBUGLIGHTINGFEATUREFLAGS_AMBIENT_OCCLUSION))
    {
        light.color *= aoFactor.directAmbientOcclusion;
    }
    #endif

    return light;
}
```
