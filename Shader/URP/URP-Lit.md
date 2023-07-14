# URP-Lit
Lit Shader是URP管线内置的用于渲染写实效果的光照着色器，该着色器使用URP中计算量最大的着色模型。  
Lit Shader要正常渲染至少需要保证有ForwardLit、DepthNormals两个Pass，如果要渲染阴影则还需ShadowCaster Pass。
## 一、ForwardLit Pass
Lit Shader的前向渲染Pass，控制着色器的渲染的流程，结构和BuildIn着色器的结构是相似的。这个Pass包含了大量的shader_feature与multi_compile变体。  

>shader_feature与multi_compile的区别：  
两者的区别在于Unity在最终的版本中不会包括shader_feature着色器的未使用的变体。shader_feature更适合处理从material中设置的关键字，而multi_compile则更适合用来处理从全局代码中设置的关键字。

### 1、ForwardLit Pass的结构
ForwardLit的代码都包含在以下两个hsls文件中，`LitInput.hlsl`定义了Shader所需要的输入数据变量，`LitForwardPass.hlsl`则负责Shader的渲染流程。  
```hlsl
#include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
#include "Packages/com.unity.render-pipelines.universal/Shaders/LitForwardPass.hlsl"
```
ForwardLit Pass的结构如下图所示：
![LitShader_1](https://github.com/raincoco/Unity/blob/main/Shader/URP/MdImages/URP-Lit/LitShader_01.png)  

### 2、Attributes And Varyings 顶点着色器输入/输出结构体
#### 2.1 Attributes 顶点着色器输入结构体
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
#### 2.2 Varyings 顶点着色器输出结构体
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

### 3、LitInput 输入数据
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

### 4、 Varyings LitPassVertex 顶点着色器
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

#### 4.1 vertexInput 顶点输入
VertexPositionInputs<br>
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

GetVertexPositionInputs<br>
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
#### 4.2 normalInput 法线输入
VertexNormalInputs<br>
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
GetVertexNormalInputs<br>
获取顶点法线数据的函数，
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

#### 4.3 vertexLight 顶点光照
只有多光源的情况下才输出顶点光，当定义`_ADDITIONAL_LIGHTS_VERTEX`，vertexLight和fogFactor存入output.fogFactorAndVertexLight中。

顶点光函数 VertexLighting<br>
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
当使用多光源时`_ADDITIONAL_LIGHTS_VERTEX`，fogFactor和顶点光vertexLight存入output.fogFactorAndVertexLight中。
GetOddNegativeScale函数
函数声明位置：SpaceTransforms.hlsl
```hlsl
real GetOddNegativeScale()
{
    return unity_WorldTransformParams.w >= 0.0 ? 1.0 : -1.0;
}
```
`unity_WorldTransformParams`声明在`UnityInput.hlsl`中，其w通常为1.0或 -1.0用于奇负尺度变换。

#### 4.4 fogFactor 雾效因子
如果未定义宏`_FOG_FRAGMENT`，则计算顶点雾效因子。
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
当使用多光源时`_ADDITIONAL_LIGHTS_VERTEX`，fogFactor和顶点光vertexLight存入output.fogFactorAndVertexLight中；如未使用多光源，fogFactor存入output.fogFactor。

#### 4.5 tangentWS And viewDirTS 世界空间切线和切线空间观察矢量
```hlsl
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR) || defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    real sign = input.tangentOS.w * GetOddNegativeScale();
    half4 tangentWS = half4(normalInput.tangentWS.xyz, sign);
#endif
```
`REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR`表示需要一个世界空间的切线插值器；<br>
`REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR`表示需要一个切线空间的观察矢量插值器；<br>
当定义这两个宏任意一个时，会计算世界空间切线tangentWS。

4.5.1 tangentWS
```hlsl
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR)
    output.tangentWS = tangentWS;
#endif
```
当定义有`REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR`时，tangentWS会被存入output.tangentWS。

4.5.2 viewDirTS 
```hlsl
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(vertexInput.positionWS);
    half3 viewDirTS = GetViewDirectionTangentSpace(tangentWS, output.normalWS, viewDirWS);
    output.viewDirTS = viewDirTS;
#endif
```
`GetWorldSpaceNormalizeViewDir`声明在`ShaderVariablesFunctions.hlsl`中。<br>
`GetViewDirectionTangentSpace`声明在`ParallaxMapping.hlsl`中。<br>

当定义有`REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR`时，先调用`GetWorldSpaceNormalizeViewDir`计算出世界空间的观察矢量viewDirWS，再调用`GetViewDirectionTangentSpace`计算出viewDirTS 存入output.viewDirTS。

#### 4.6 LIGHTMAP_UV And SH 光照贴图UV和球谐光
```hlsl
OUTPUT_LIGHTMAP_UV(input.staticLightmapUV, unity_LightmapST, output.staticLightmapUV);
OUTPUT_SH(output.normalWS.xyz, output.vertexSH);
```
`OUTPUT_LIGHTMAP_UV`和`OUTPUT_SH`声明在`Lighting.hlsl`中:
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
当启用LIGHTMAP时，不计算球谐光SH，LIGHTMAP的UV如下：
```hlsl
OUT.xy = lightmapUV.xy * lightmapScaleOffset.xy + lightmapScaleOffset.zw;
```
当不启用LIGHTMAP时，计算球谐光SH，球谐光SH如下：
```hlsl
OUT.xyz = SampleSHVertex(normalWS)
```
球谐光SH采样函数`SampleSHVertex`声明在`GlobalIllumination.hlsl`中。

#### 4.7 DYNAMICLIGHTMAP UV 动态光照贴图UV
```hlsl
output.dynamicLightmapUV = input.dynamicLightmapUV.xy * unity_DynamicLightmapST.xy + unity_DynamicLightmapST.zw;
```
`unity_DynamicLightmapST`声明在'UnityInput.hlsl'中。

#### 4.8 shadowCoord 阴影纹理
GetShadowCoord
函数声明位置：Shadow.hlsl
```hlsl
output.shadowCoord = GetShadowCoord(vertexInput);
```

### 6、 LitPassFragment片元着色器
```hlsl
half4 LitPassFragment(Varyings input) : SV_Target
{
    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input); //XR项目使用

#if defined(_PARALLAXMAP)
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirTS = input.viewDirTS;
#else
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);
    half3 viewDirTS = GetViewDirectionTangentSpace(input.tangentWS, input.normalWS, viewDirWS);
#endif
    ApplyPerPixelDisplacement(viewDirTS, input.uv);
#endif

    SurfaceData surfaceData;
    InitializeStandardLitSurfaceData(input.uv, surfaceData);

    InputData inputData;
    InitializeInputData(input, surfaceData.normalTS, inputData);
    SETUP_DEBUG_TEXTURE_DATA(inputData, input.uv, _BaseMap);

#ifdef _DBUFFER
    ApplyDecalToSurfaceData(input.positionCS, surfaceData, inputData);
#endif

    half4 color = UniversalFragmentPBR(inputData, surfaceData);

    color.rgb = MixFog(color.rgb, inputData.fogCoord);
    color.a = OutputAlpha(color.a, _Surface);

    return color;
}
```

#### 6.1 UNITY_SETUP_INSTANCE_ID 实例化ID
UNITY_SETUP_INSTANCE_ID是用于记录不同实例属性ID的方法，UNITY_SETUP_INSTANCE_ID(input)可以用来访问全局unity_InstanceID，需放在顶点和片元着色器起始第一行。
如果需要将实例化ID传到片段着色器，则需在顶点着色器中增加UNITY_TRANSFER_INSTANCE_ID(v, o);

#### 6.2 PARALLAXMA 视差图
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
URP预设宏_PARALLAXMAP来进行视差映射，其中ApplyPerPixelDisplacement负责计算视差图，如下。
```hlsl
void ApplyPerPixelDisplacement(half3 viewDirTS, inout float2 uv)
{
#if defined(_PARALLAXMAP)
    uv += ParallaxMapping(TEXTURE2D_ARGS(_ParallaxMap, sampler_ParallaxMap), viewDirTS, _Parallax, uv);
#endif
}
```
```hlsl
float2 ParallaxMapping(TEXTURE2D_PARAM(heightMap, sampler_heightMap), half3 viewDirTS, half scale, float2 uv)
{
    half h = SAMPLE_TEXTURE2D(heightMap, sampler_heightMap, uv).g;
    float2 offset = ParallaxOffset1Step(h, scale, viewDirTS);
    return offset;
}
```
```hlsl
half2 ParallaxOffset1Step(half height, half amplitude, half3 viewDirTS)
{
    height = height * amplitude - amplitude / 2.0;
    half3 v = normalize(viewDirTS);
    v.z += 0.42;
    return height * (v.xy / v.z);
}
```

#### 6.3 数据初始化
初始化模型表面数据和外部输入数据。
```hlsl
SurfaceData surfaceData;
InitializeStandardLitSurfaceData(input.uv, surfaceData);

InputData inputData;
InitializeInputData(input, surfaceData.normalTS, inputData);
```
##### 6.3.1 初始化表面数据
SurfaceData声明在SurfaceData.hlsl中，如下。

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
InitializeStandardLitSurfaceData声明在LitInput.hlsl中，如下。
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
##### 6.3.2 初始化输入数据
InputData声明在Input.hlsl中，如下。
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
InitializeInputData声明在LitForwardPass.hlsl中，如下。
```hlsl
void InitializeInputData(Varyings input, half3 normalTS, out InputData inputData)
{
    inputData = (InputData)0;

#if defined(REQUIRES_WORLD_SPACE_POS_INTERPOLATOR)
    inputData.positionWS = input.positionWS;
#endif

    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);
#if defined(_NORMALMAP) || defined(_DETAIL)
    float sgn = input.tangentWS.w;      // should be either +1 or -1
    float3 bitangent = sgn * cross(input.normalWS.xyz, input.tangentWS.xyz);
    half3x3 tangentToWorld = half3x3(input.tangentWS.xyz, bitangent.xyz, input.normalWS.xyz);

    #if defined(_NORMALMAP)
    inputData.tangentToWorld = tangentToWorld;
    #endif
    inputData.normalWS = TransformTangentToWorld(normalTS, tangentToWorld);
#else
    inputData.normalWS = input.normalWS;
#endif

    inputData.normalWS = NormalizeNormalPerPixel(inputData.normalWS);
    inputData.viewDirectionWS = viewDirWS;

#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
    inputData.shadowCoord = input.shadowCoord;
#elif defined(MAIN_LIGHT_CALCULATE_SHADOWS)
    inputData.shadowCoord = TransformWorldToShadowCoord(inputData.positionWS);
#else
    inputData.shadowCoord = float4(0, 0, 0, 0);
#endif
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    inputData.fogCoord = InitializeInputDataFog(float4(input.positionWS, 1.0), input.fogFactorAndVertexLight.x);
    inputData.vertexLighting = input.fogFactorAndVertexLight.yzw;
#else
    inputData.fogCoord = InitializeInputDataFog(float4(input.positionWS, 1.0), input.fogFactor);
#endif

#if defined(DYNAMICLIGHTMAP_ON)
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.dynamicLightmapUV, input.vertexSH, inputData.normalWS);
#else
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.vertexSH, inputData.normalWS);
#endif

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

##### 6.3.3 InitializeStandardLitSurfaceData

