# 一、SpaceTransforms
Unity常用的空间转换函数，这些函数都定义在以下文件中（URP 12.1.10）：
```
com.unity.render-pipelines.core@12.1.10\ShaderLibrary\SpaceTransforms.hlsl
```

```HLSL
// 转换顶点坐标的空间
float4 TransformWViewToHClip  (float3 positionVS) // VS --> CS 
float4 TransformObjectToHClip (float3 positionOS) // OS --> CS 
float3 TransformObjectToWorld (float3 positionOS) // OS --> WS 
float4 TransformWorldToHClip  (float3 positionWS) // WS --> CS 
float3 TransformWorldToView   (float3 positionWS) // WS --> VS 
float3 TransformWorldToObject (float3 positionWS) // WS --> OS 

// 转换向量的空间
float3 TransformObjectToWorldDir (float3 dirOS, bool doNormalize = true)       // OS --> WS 
float3 TransformWorldToObjectDir (float3 dirWS, bool doNormalize = true)       // WS --> OS 
real3  TransformWorldToViewDir   (real3 dirWS, bool doNormalize = false)       // WS --> VS 
real3  TransformWorldToHClipDir  (real3 directionWS, bool doNormalize = false) // WS --> CS 
real3  TransformTangentToWorld   (float3 dirTS, real3x3 tangentToWorld)        // TS --> WS
real3  TransformWorldToTangent   (real3 dirWS, real3x3 tangentToWorld)         // WS --> TS
real3  TransformTangentToObject  (real3 dirTS, real3x3 tangentToWorld)         // TS --> OS
real3  TransformObjectToTangent  (real3 dirOS, real3x3 tangentToWorld)         // OS --> TS

// 转换法线的空间
float3 TransformObjectToWorldNormal(float3 normalOS, bool doNormalize = true) // OS --> WS 
float3 TransformWorldToObjectNormal(float3 normalWS, bool doNormalize = true) // WS --> OS 
```

> 关于传入参数doNormalize：
> 有的转换函数的传入参数中有一个bool型参数doNormalize，这个doNormalize负责决定转化的结果是否进行归一化（Normalize）。在调用转换函数时可以不传入doNormalize，此时doNormalize默认为true。

### （1）Unity转换矩阵
Unity 转换矩阵都是 `float4x4` 类型，并且是列主序的。

| 矩阵名称                |                   |
| ------------------- | ----------------- |
| UNITY_MATRIX_MVP    | 当前模型 - 视图 - 投影矩阵。 |
| UNITY_MATRIX_MV     | 当前模型 - 视图矩阵。      |
| UNITY_MATRIX_V      | 当前视图矩阵。           |
| UNITY_MATRIX_P      | 当前投影矩阵。           |
| UNITY_MATRIX_VP     | 当前视图 - 投影矩阵。      |
| UNITY_MATRIX_T_MV   | 模型转置 - 视图矩阵。      |
| UNITY_MATRIX_IT_MV  | 模型逆转置 - 视图矩阵。     |
| unity_ObjectToWorld | 当前模型矩阵。           |
| unity_WorldToObject | 当前世界矩阵的逆矩阵。       |
Unity调用转换矩阵的函数：
```HLSL
float4x4 GetObjectToWorldMatrix()
{
    return UNITY_MATRIX_M;
}

float4x4 GetWorldToObjectMatrix()
{
    return UNITY_MATRIX_I_M;
}

float4x4 GetPrevObjectToWorldMatrix()
{
    return UNITY_PREV_MATRIX_M;
}

float4x4 GetPrevWorldToObjectMatrix()
{
    return UNITY_PREV_MATRIX_I_M;
}

float4x4 GetWorldToViewMatrix()
{
    return UNITY_MATRIX_V;
}

// Transform to homogenous clip space
float4x4 GetWorldToHClipMatrix()
{
    return UNITY_MATRIX_VP;
}

// Transform to homogenous clip space
float4x4 GetViewToHClipMatrix()
{
    return UNITY_MATRIX_P;
}
```
### （2）Unity转换函数
 TransformWViewToHClip
```HLSL
float4 TransformWViewToHClip(float3 positionVS)
{
    return mul(GetViewToHClipMatrix(), float4(positionVS, 1.0));
}
```
 TransformObjectToHClip
```HLSL
float4 TransformObjectToHClip(float3 positionOS)
{
    return mul(GetWorldToHClipMatrix(), mul(GetObjectToWorldMatrix(), float4(positionOS, 1.0)));
}
```
 TransformObjectToWorld
```HLSL
float3 TransformObjectToWorld(float3 positionOS)
{
    #if defined(SHADER_STAGE_RAY_TRACING)
    return mul(ObjectToWorld3x4(), float4(positionOS, 1.0)).xyz;
    #else
    return mul(GetObjectToWorldMatrix(), float4(positionOS, 1.0)).xyz;
    #endif
}
```
 TransformWorldToHClip
```HLSL
float4 TransformWorldToHClip(float3 positionWS)
{
    return mul(GetWorldToHClipMatrix(), float4(positionWS, 1.0));
}
```
 TransformWorldToView
```HLSL
float3 TransformWorldToView(float3 positionWS)
{
    return mul(GetWorldToViewMatrix(), float4(positionWS, 1.0)).xyz;
}
```
 TransformWorldToObject
```HLSL
float3 TransformWorldToObject(float3 positionWS)
{
    #if defined(SHADER_STAGE_RAY_TRACING)
    return mul(WorldToObject3x4(), float4(positionWS, 1.0)).xyz;
    #else
    return mul(GetWorldToObjectMatrix(), float4(positionWS, 1.0)).xyz;
    #endif
}
```
 TransformObjectToWorldDir
```HLSL
float3 TransformObjectToWorldDir(float3 dirOS, bool doNormalize = true)
{
    #ifndef SHADER_STAGE_RAY_TRACING
    float3 dirWS = mul((float3x3)GetObjectToWorldMatrix(), dirOS);
    #else
    float3 dirWS = mul((float3x3)ObjectToWorld3x4(), dirOS);
    #endif
    if (doNormalize)
        return SafeNormalize(dirWS);

    return dirWS;
}
```
 TransformWorldToObjectDir
```HLSL
float3 TransformWorldToObjectDir(float3 dirWS, bool doNormalize = true)
{
    #ifndef SHADER_STAGE_RAY_TRACING
    float3 dirOS = mul((float3x3)GetWorldToObjectMatrix(), dirWS);
    #else
    float3 dirOS = mul((float3x3)WorldToObject3x4(), dirWS);
    #endif
    if (doNormalize)
        return normalize(dirOS);

    return dirOS;
}
```
 TransformWorldToViewDir
```HLSL
real3 TransformWorldToViewDir(real3 dirWS, bool doNormalize = false)
{
    float3 dirVS = mul((real3x3)GetWorldToViewMatrix(), dirWS).xyz;
    if (doNormalize)
        return normalize(dirVS);

    return dirVS;
}
```
 TransformWorldToHClipDir
```HLSL
real3 TransformWorldToHClipDir(real3 directionWS, bool doNormalize = false)
{
    float3 dirHCS = mul((real3x3)GetWorldToHClipMatrix(), directionWS).xyz;
    if (doNormalize)
        return normalize(dirHCS);

    return dirHCS;
}
```
 TransformTangentToWorld
```HLSL
real3 TransformTangentToWorld(float3 dirTS, real3x3 tangentToWorld)
{
    // Note matrix is in row major convention with left multiplication as it is build on the fly
    return mul(dirTS, tangentToWorld);
}
```
 TransformWorldToTangent
```HLSL
real3 TransformWorldToTangent(real3 dirWS, real3x3 tangentToWorld)
{
    // Note matrix is in row major convention with left multiplication as it is build on the fly
    float3 row0 = tangentToWorld[0];
    float3 row1 = tangentToWorld[1];
    float3 row2 = tangentToWorld[2];

    // these are the columns of the inverse matrix but scaled by the determinant
    float3 col0 = cross(row1, row2);
    float3 col1 = cross(row2, row0);
    float3 col2 = cross(row0, row1);

    float determinant = dot(row0, col0);
    float sgn = determinant<0.0 ? (-1.0) : 1.0;

    // inverse transposed but scaled by determinant
    // Will remove transpose part by using matrix as the first arg in the mul() below
    // this makes it the exact inverse of what TransformTangentToWorld() does.
    real3x3 matTBN_I_T = real3x3(col0, col1, col2);

    return SafeNormalize( sgn * mul(matTBN_I_T, dirWS) );
}
```
 TransformTangentToObject
```HLSL
real3 TransformTangentToObject(real3 dirTS, real3x3 tangentToWorld)
{
    // Note matrix is in row major convention with left multiplication as it is build on the fly
    real3 normalWS = TransformTangentToWorld(dirTS, tangentToWorld);
    return TransformWorldToObjectNormal(normalWS);
}
```
 TransformObjectToTangent
```HLSL
real3 TransformObjectToTangent(real3 dirOS, real3x3 tangentToWorld)
{
    // Note matrix is in row major convention with left multiplication as it is build on the fly

    // don't normalize, as normalWS will be normalized after TransformWorldToTangent
    float3 normalWS = TransformObjectToWorldNormal(dirOS, false);

    // transform from world to tangent
    return TransformWorldToTangent(normalWS, tangentToWorld);
}
```
 TransformObjectToWorldNormal
```HLSL
float3 TransformObjectToWorldNormal(float3 normalOS, bool doNormalize = true)
{
#ifdef UNITY_ASSUME_UNIFORM_SCALING
    return TransformObjectToWorldDir(normalOS, doNormalize);
#else
    // Normal need to be multiply by inverse transpose
    float3 normalWS = mul(normalOS, (float3x3)GetWorldToObjectMatrix());
    if (doNormalize)
        return SafeNormalize(normalWS);

    return normalWS;
#endif
}
```
 TransformWorldToObjectNormal
```HLSL
float3 TransformWorldToObjectNormal(float3 normalWS, bool doNormalize = true)
{
#ifdef UNITY_ASSUME_UNIFORM_SCALING
    return TransformWorldToObjectDir(normalWS, doNormalize);
#else
    // Normal need to be multiply by inverse transpose
    float3 normalOS = mul(normalWS, (float3x3)GetObjectToWorldMatrix());
    if (doNormalize)
        return SafeNormalize(normalOS);

    return normalOS;
#endif
}
```
# 二、Light
## 1、Light 结构体
```hlsl
struct Light
{
    half3   direction;            // 灯光方向
    half3   color;                // 灯光颜色
    half    distanceAttenuation;  // 距离衰减
    half    shadowAttenuation;    // 阴影衰减
    uint    layerMask;            // 遮罩层级
};
```
## 2、GetMainLight 获取主灯光
`GetMainLight`用于获取主灯光数据，该函数进行了两次重载，以下按重载顺序分析。
### （1）GetMainLight 函数定义
```hlsl
Light GetMainLight(InputData inputData, half4 shadowMask, AmbientOcclusionFactor aoFactor)
{
    Light light = GetMainLight(inputData.shadowCoord, inputData.positionWS, shadowMask);
    
	// 直接光照的环境遮蔽Debug，需启用屏幕空间遮挡 且 没用启用表面透明。
    #if defined(_SCREEN_SPACE_OCCLUSION) && !defined(_SURFACE_TYPE_TRANSPARENT)
    if (IsLightingFeatureEnabled(DEBUGLIGHTINGFEATUREFLAGS_AMBIENT_OCCLUSION))
    {
        light.color *= aoFactor.directAmbientOcclusion;
    }
    #endif

    return light;
}
```
`GetMainLight`函数定义里就两个部分，一是实现直接光照的环境遮蔽（AMBIENT_OCCLUSION）的Debug功能，二是重载`GetMainLight`。

`GetMainLight`传入的三个参数具体使用了四个变量：
1. InputData 结构体的阴影纹理坐标 inputData.shadowCoord；
2. InputData 结构体的世界空间顶点位置 inputData.positionWS；
3. shadowMask 阴影遮罩（与烘焙光照贴图相关）；
4. aoFactor 遮蔽因子。
### （2）GetMainLight 第一次重载
第一次重载`GetMainLight`，主要完成两个计算：
1. shadowAttenuation：主灯光阴影衰减，调用函数`MainLightShadow`完成计算；
2. Cookie：根据`_LIGHT_COOKIES`判断是否启用Cookie，计算乘以Cookie后的灯光颜色。
```hlsl
Light GetMainLight(float4 shadowCoord, float3 positionWS, half4 shadowMask)
{
    Light light = GetMainLight();
    // 获取灯光的阴影衰减（或者应该表述为衰减的阴影）
    light.shadowAttenuation = MainLightShadow(shadowCoord, positionWS, 
									    shadowMask, _MainLightOcclusionProbes);
	// Cookie
    #if defined(_LIGHT_COOKIES)
        real3 cookieColor = SampleMainLightCookie(positionWS);
        light.color *= cookieColor;
    #endif

    return light;
}
```
`_MainLightOcclusionProbes`是 Input.hlsl 中定义的一个常量缓冲区的变量，当着色器使用光照贴图时，这个变量才会被使用。（不清楚里面存储的数据是哪里来的）
### （3）GetMainLight 第二次重载
第二次重载是为了获取灯光方向、衰减、颜色等数据。
```hlsl
Light GetMainLight()
{
    Light light;
    // 灯光方向
    light.direction = half3(_MainLightPosition.xyz);
    // 灯光衰减
#if USE_CLUSTERED_LIGHTING
	// 使用集群照明，灯光距离衰减为1.0；
    light.distanceAttenuation = 1.0;
#else
	// 使用非集群照明，灯光距离衰减为unity_LightData.z，这个值与是否剔除相关
    light.distanceAttenuation = unity_LightData.z; 
    // unity_LightData.z is 1 when not culled by the culling mask, otherwise 0.
    // 官方注释：当没有被剔除掩码剔除时，unity_LightData.z 为 1，否则为 0。
#endif
	// 灯光阴影衰减
    light.shadowAttenuation = 1.0;
    // 灯光颜色
    light.color = _MainLightColor.rgb;

	// 灯光层级
#ifdef _LIGHT_LAYERS
    light.layerMask = _MainLightLayerMask;
#else
    light.layerMask = DEFAULT_LIGHT_LAYERS;
#endif

    return light;
}
```
## 3、Cookie 纹理
Cookie 是一张 alpha 纹理贴图，通过灯光照射投影纹理，放置在光源前面并随光源移动。Cookie的原理是让光线通过纹理 alpha 分量为 1 的位置，并阻挡 alpha 分量为 0 的光线通过，直接与计算的光照相乘即可。

在Unity中，当选择光源时，可以在 Inspector View 中为每个光源指定一个 Cookie。Lit根据预编译的变体关键字`_LIGHT_COOKIES`判断是否启用Cookie。
```HLSL
#if defined(_LIGHT_COOKIES)
	real3 cookieColor = SampleMainLightCookie(positionWS);
	light.color *= cookieColor;
#endif
```
## 4、混合实时光照和烘焙GI
MixRealtimeAndBakedGI 实现实时光照和烘焙GI混合。

烘焙的光照贴图（LightingMap）存放的是主光源的光照和阴影信息，这里的“混合”实际就是从光照贴图中减去直接主光源的光照和阴影（LightingMap 光照的信息），以避免与实时光照再叠加，最终得到的阴影是这个差值和实时阴影中更暗的那个。

要使用 MixRealtimeAndBakedGI 需要同时满足两个条件：
- Lit 使用光照贴图LightMap；
- 混合光照模式为Subtractive（Linghting>Scene>Mixed Lighting>Lighting Mode选项为Subtractive）。

**注意**：如果注释掉`MixRealtimeAndBakedGI`，使用LightMap的静态物体将无法接受来自动态物体的阴影。

**● MixRealtimeAndBakedGI**
判断是否调用`SubtractDirectMainLightFromLightmap`函数。
```hlsl
void MixRealtimeAndBakedGI(inout Light light, half3 normalWS, inout half3 bakedGI)
{
#if defined(LIGHTMAP_ON) && defined(_MIXED_LIGHTING_SUBTRACTIVE)
    bakedGI = SubtractDirectMainLightFromLightmap(light, normalWS, bakedGI);
#endif
}
```

**● _MIXED_LIGHTING_SUBTRACTIVE 关键字**
```hlsl
#if !defined(_MIXED_LIGHTING_SUBTRACTIVE) && defined(LIGHTMAP_SHADOW_MIXING) && !defined(SHADOWS_SHADOWMASK)
    #define _MIXED_LIGHTING_SUBTRACTIVE
#endif
```

**● SubtractDirectMainLightFromLightmap**
计算烘焙光和实时光的混合阴影。
>Step1：实时光阴影估算值 = 烘焙全局光（bakedGI）阴影中 - 主光源（一个简单的Lambert光照）阴影；
>Step2：限制实时光阴影估算值差值的最小值；
>Step3：输出烘焙光和实时光中阴影更暗的一方。
```hlsl
half3 SubtractDirectMainLightFromLightmap(Light mainLight, half3 normalWS, half3 bakedGI)
{
    // Let's try to make realtime shadows work on a surface, which already contains
    // baked lighting and shadowing from the main sun light.
    // Summary:
    // 1) Calculate possible value in the shadow by subtracting estimated light contribution from the places occluded by realtime shadow:
    //      a) preserves other baked lights and light bounces
    //      b) eliminates shadows on the geometry facing away from the light
    // 2) Clamp against user defined ShadowColor.
    // 3) Pick original lightmap value, if it is the darkest one.


    // 1) Gives good estimate of illumination as if light would've been shadowed during the bake.
    // We only subtract the main direction light. This is accounted in the contribution term below.

	// 获取主灯光阴影强度
    half shadowStrength = GetMainLightShadowStrength();
    // 计算了一个Lambert光照
    half contributionTerm = saturate(dot(mainLight.direction, normalWS));
    half3 lambert = mainLight.color * contributionTerm;
	// 计算出主灯光在阴影中的贡献值
    half3 estimatedLightContributionMaskedByInverseOfShadow = lambert * (1.0 - mainLight.shadowAttenuation);
	// 实时光阴影估算值 = 烘焙全局光（光照贴图） - 主灯光（实时光）的阴影贡献值
    half3 subtractedLightmap = bakedGI - estimatedLightContributionMaskedByInverseOfShadow;

    // 2) Allows user to define overall ambient of the scene and control situation when realtime shadow becomes too dark.

	// 限制实时光阴影估算值差值的最小值
    half3 realtimeShadow = max(subtractedLightmap, _SubtractiveShadowColor.xyz);
    // shadowStrength控制实时阴影强度
    realtimeShadow = lerp(bakedGI, realtimeShadow, shadowStrength);

    // 3) Pick darkest color
    // 输出烘焙光和实时光中阴影更暗的一方
    return min(bakedGI, realtimeShadow);
}
```
\_SubtractiveShadowColor 是Unity设置的的固定值：`float3(0.42,0.478,0.627)`
# 三、Shadow
## 1、MainLightShadow 主灯光阴影
`GetMainLight`中调用的`MainLightShadow`计算阴影的衰减：
```hlsl
light.shadowAttenuation = MainLightShadow(shadowCoord, positionWS, 
									shadowMask, _MainLightOcclusionProbes);
```
`_MainLightOcclusionProbes`是 Input.hlsl 中定义的一个常量缓冲区的变量，当着色器使用光照贴图时，这个变量才会被使用。（不清楚里面存储的数据是哪里来的）。
### （1）MainLightShadow
计算主灯光阴影。由 灯光实时阴影、烘焙阴影、阴影渐变系数、混合实时阴影烘焙阴影 四个部分的计算组成。
```hlsl
//Include：PackageCache\com.unity.render-pipelines.universal@12.1.10\ShaderLibrary\Shadows.hlsl
//-------------------------------------------------------------------------------
half MainLightShadow(float4 shadowCoord, float3 positionWS, half4 shadowMask, half4 occlusionProbeChannels)
{
	// 灯光实时阴影
    half realtimeShadow = MainLightRealtimeShadow(shadowCoord);

	// 烘焙阴影
#ifdef CALCULATE_BAKED_SHADOWS
    half bakedShadow = BakedShadow(shadowMask, occlusionProbeChannels);
#else
    half bakedShadow = half(1.0);
#endif

	// 阴影渐变系数
	// 如果要计算主灯光阴影，则需要通过GetMainLightShadowFade获取渐变系数；如果不计算，渐变系数则为1。
#ifdef MAIN_LIGHT_CALCULATE_SHADOWS
    half shadowFade = GetMainLightShadowFade(positionWS);
#else
    half shadowFade = half(1.0);
#endif
	
	// 混合实时阴影和烘焙阴影
    return MixRealtimeAndBakedShadows(realtimeShadow, bakedShadow, shadowFade);
}
```
--------------------------------------------------------------------------
**CALCULATE_BAKED_SHADOWS 关键字**
该关键字表示着色器会计算烘焙光照贴图的阴影，其定义如下：
```hlsl
#if defined(LIGHTMAP_ON) || defined(LIGHTMAP_SHADOW_MIXING) || defined(SHADOWS_SHADOWMASK)
#define CALCULATE_BAKED_SHADOWS
#endif
```
Pass 需要具有以下任一 Unity 所定义的预编译关键字，`CALCULATE_BAKED_SHADOWS`关键字才会被定义。
```hlsl
// Unity defined keywords
#pragma multi_compile _ LIGHTMAP_SHADOW_MIXING // 混合光照贴图阴影
#pragma multi_compile _ SHADOWS_SHADOWMASK     // 启用阴影遮罩
#pragma multi_compile _ LIGHTMAP_ON            // 启用光照贴图
```
--------------------------------------------------------------------------
**MAIN_LIGHT_CALCULATE_SHADOWS 关键字**
该关键字表示着色器会计算主灯光阴影，其定义如下：
```hlsl
//Include：PackageCache\com.unity.render-pipelines.universal@12.1.10\ShaderLibrary\Shadows.hlsl
//-------------------------------------------------------------------------------
#if !defined(_RECEIVE_SHADOWS_OFF)
    #if defined(_MAIN_LIGHT_SHADOWS) || defined(_MAIN_LIGHT_SHADOWS_CASCADE) || defined(_MAIN_LIGHT_SHADOWS_SCREEN)
        #define MAIN_LIGHT_CALCULATE_SHADOWS
		......
    #endif
	......
#endif
```
在Lit中，`MAIN_LIGHT_CALCULATE_SHADOWS`关键字需要在物体开启阴影接收，且shader具有以下预编译关键字的情况下才会被定义。
```hlsl
// Universal Pipeline keywords
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _MAIN_LIGHT_SHADOWS_SCREEN
```
### （2）GetMainLightShadowFade
获取主灯光阴影渐变系数。
```hlsl
half GetMainLightShadowFade(float3 positionWS)
{
    float3 camToPixel = positionWS - _WorldSpaceCameraPos;
    float distanceCamToPixel2 = dot(camToPixel, camToPixel);

    float fade = saturate(distanceCamToPixel2 * float(_MainLightShadowParams.z) + float(_MainLightShadowParams.w));
    return half(fade);
}
```
### （3）BakedShadow
烘焙阴影。
```hlsl
half BakedShadow(half4 shadowMask, half4 occlusionProbeChannels)
{
    // Here occlusionProbeChannels used as mask selector to select shadows in shadowMask
    // If occlusionProbeChannels all components are zero we use default baked shadow value 1.0
    // This code is optimized for mobile platforms:
    // half bakedShadow = any(occlusionProbeChannels) ? dot(shadowMask, occlusionProbeChannels) : 1.0h;
    half bakedShadow = half(1.0) + dot(shadowMask - half(1.0), occlusionProbeChannels);
    return bakedShadow;
}
```
### （4）MixRealtimeAndBakedShadows
混合实时阴影和烘焙阴影，根据关键字`LIGHTMAP_SHADOW_MIXING`判断是否使用LightMap Shadow进行混合。
```hlsl
half MixRealtimeAndBakedShadows(half realtimeShadow, half bakedShadow, half shadowFade)
{
#if defined(LIGHTMAP_SHADOW_MIXING)
    return min(lerp(realtimeShadow, 1, shadowFade), bakedShadow);
#else
    return lerp(realtimeShadow, bakedShadow, shadowFade);
#endif
}
```
**定义`LIGHTMAP_SHADOW_MIXING`：**
> 根据渐变系数 shadowFade 使阴影在实时阴影和1之间平滑过渡，且阴影的值不会低于烘焙阴影 bakedShadow 的值。

**未定义`LIGHTMAP_SHADOW_MIXING`：**
> 根据渐变系数 shadowFade 使阴影在实时阴影和烘焙阴影之间平滑过渡。

**LIGHTMAP_SHADOW_MIXING 关键字** 
Lit.shader的ForwardLit、GBuffer两个Pass中有使用这个关键字，该关键字定义了一个混合光照贴图阴影的变体。
```hlsl
// Unity defined keywords
#pragma multi_compile _ LIGHTMAP_SHADOW_MIXING
```
## 2、shadowMask 阴影遮罩
**（1）unity_ProbesOcclusion**
Unity将`shadowMask`阴影遮罩数据烘焙到了光探针中，这个探针就是`unity_ProbesOcclusion`遮挡探针，或者我们可以将它称之为阴影探针。

`unity_ProbesOcclusion`的定义如下：
```hlsl
#define unity_ProbesOcclusion       UNITY_ACCESS_DOTS_INSTANCED_PROP_FROM_MACRO(float4,Metadataunity_ProbesOcclusion)
```

**（2）UNITY_ACCESS_DOTS_INSTANCED_PROP_FROM_MACRO**
官方注释解释`UNITY_DOTS_INSTANCED_METADATA_NAME_FROM_MACRO`这类型的宏可用于让着色器代码从DOTS实例中加载常量，如同普通常量一样。
```hlsl
#define UNITY_ACCESS_DOTS_INSTANCED_PROP_FROM_MACRO(type,metadata_underscore_var)
LoadDOTSInstancedData_##type(
	UNITY_DOTS_INSTANCED_METADATA_NAME_FROM_MACRO(type,metadata_underscore_var)
)
```

**（3）LoadDOTSInstancedData_##type**
```hlsl
#define DEFINE_DOTS_LOAD_INSTANCE_SCALAR(type, conv, sizeof_type) \
type LoadDOTSInstancedData_##type(uint metadata) \
{ \
    uint address = ComputeDOTSInstanceDataAddress(metadata, sizeof_type); \
    return conv(unity_DOTSInstanceData.Load(address)); \
}
```

（4）DOTS
Unity DOTS（Data-Oriented Technology Stack 多线程式数据导向型技术堆栈 / 面向数据的技术堆栈）是Unity引擎的最新技术栈，其核心技术是实体组件系统 [Entity Component System](https://docs.unity3d.com/Packages/com.unity.entities@0.0/manual/index.html?q=Data-Oriented%20Tech)（ECS）。库文件UniversalDOTSInstancing.hlsl中存放的是一些普遍使用的DOTS实例。

# 5、ConvertF0ForClearCoat15
该函数用于计算清漆涂层的高光`baseBRDFData.specular`。
```hlsl
half3 ConvertF0ForClearCoat15(half3 f0)
{
#if defined(SHADER_API_MOBILE)
	// 移动端
    return ConvertF0ForAirInterfaceToF0ForClearCoat15Fast(f0);
#else
	// 非移动端
    return ConvertF0ForAirInterfaceToF0ForClearCoat15(f0);
#endif
}
```
`ConvertF0ForAirInterfaceToF0ForClearCoat15Fast`和`ConvertF0ForAirInterfaceToF0ForClearCoat15`两个函数是由`TEMPLATE_1_REAL`定义的模版函数，官方源码如下：
```hlsl
/*
官方注释：
// This function is a coarse approximation of computing fresnel0 for a different top than air (here clear coat of IOR 1.5) when we only have fresnel0 with air interface
// This function is equivalent to IorToFresnel0(Fresnel0ToIor(fresnel0), 1.5)
// mean
// real sqrtF0 = sqrt(fresnel0);
// return Sq(1.0 - 5.0 * sqrtF0) / Sq(5.0 - sqrtF0);
// Optimization: Fit of the function (3 mad) for range [0.04 (should return 0), 1 (should return 1)]
当只有fresnel0和空气介面时，此函数计算fresnel0的粗略近似值，用于空气不同的顶部，即不同于空气的其他介质（此处为IOR的透明涂层）。

空气介面应该是涂层表面和空气这两种不同介质间的表面。
*/

// 非移动端
TEMPLATE_1_REAL(ConvertF0ForAirInterfaceToF0ForClearCoat15, fresnel0, return saturate(-0.0256868 + fresnel0 * (0.326846 + (0.978946 - 0.283835 * fresnel0) * fresnel0)))

/*
官方注释：
// Even coarser approximation of ConvertF0ForAirInterfaceToF0ForClearCoat15 (above) for mobile (2 mad)
对于移动设备，ConvertF0ForAirInterfaceToF0ForClearCoat15的更粗略近似（2 mad）
*/

// 移动端
TEMPLATE_1_REAL(ConvertF0ForAirInterfaceToF0ForClearCoat15Fast, fresnel0, return saturate(fresnel0 * (fresnel0 * 0.526868 + 0.529324) - 0.0482256))
```
计算清漆涂层高光的两个函数如下：
```hlsl
// ConvertF0ForAirInterfaceToF0ForClearCoat15
TEMPLATE_1_REAL(ConvertF0ForAirInterfaceToF0ForClearCoat15, fresnel0, return saturate(-0.0256868 + fresnel0 * (0.326846 + (0.978946 - 0.283835 * fresnel0) * fresnel0)))

// ConvertF0ForAirInterfaceToF0ForClearCoat15Fast
TEMPLATE_1_REAL(ConvertF0ForAirInterfaceToF0ForClearCoat15Fast, fresnel0, return saturate(fresnel0 * (fresnel0 * 0.526868 + 0.529324) - 0.0482256))
```
相当于
```hlsl
half3 ConvertF0ForAirInterfaceToF0ForClearCoat15(half3 fresnel0)
{
	return saturate(-0.0256868 + fresnel0 * (0.326846 + 
					(0.978946 - 0.283835 * fresnel0) * fresnel0)))
}
half3 ConvertF0ForAirInterfaceToF0ForClearCoat15Fast(half3 fresnel0)
{
	 return saturate(fresnel0 * (fresnel0 * 0.526868 + 0.529324) - 0.0482256)
}
```

# 四、雾效
Unity 的雾分为线性雾（FOG_LINEAR）和指数雾（FOG_EXP）两种，其中指数雾根据浓度计算方式的不同又分为 FOG_EXP 和 FOG_EXP2 两种。

## 1、雾效因子
### ComputeFogFactor
这个函数有两个作用，一是抽象化平台差异，统一全局的Z深度；二是调用`ComputeFogFactorZ0ToFar`计算雾效因子。`ComputeFogFactor`函数如下：
```hlsl
real ComputeFogFactor(float zPositionCS)
{
    float clipZ_0Far = UNITY_Z_0_FAR_FROM_CLIPSPACE(zPositionCS);
    return ComputeFogFactorZ0ToFar(clipZ_0Far);
}
```

**● UNITY_Z_0_FAR_FROM_CLIPSPACE**
根据官方文档说明，如果要使用裁剪空间 (Z) 深度，需要使用宏`UNITY_Z_0_FAR_FROM_CLIPSPACE`来抽象化平台差异，即统一全局的Z深度：
```hlsl
float clipSpaceRange01 = UNITY_Z_0_FAR_FROM_CLIPSPACE(rawClipSpace);
```
**注意**：这个宏不会改变 OpenGL 或 OpenGL ES 平台上的裁剪空间，只是返回“-near”1（近平面）到 far（远平面）之间的值。

在源码中，UNITY_Z_0_FAR_FROM_CLIPSPACE 针对不同平台的不同情况都有相应处理：
```
//D3d with reversed Z => z clip range is [near, 0] -> remapping to [0, far]
//max is required to protect ourselves from near plane not being corr
//D3d平台 Z值反转 => Z裁剪范围从[near, 0]重映射到[0, far]

//GL with reversed z => z clip range is [near, -far] -> should remap in theory but dont do it in practice to save some perf (range is close enough)
//GL平台 Z值反转 => Z裁剪范围为[near, 0]，理论上应该重映射，但为了节约性能所有没有重映射（其范围足够接近）
       
//D3d without reversed z => z clip range is [0, far] -> nothing to do
//D3d平台 未反转Z值 => Z裁剪范围为[0, far]

//Opengl => z clip range is [-near, far] -> should remap in theory but dont do it in practice to save some perf (range is close enough)
//OpenGL平台 => Z裁剪范围为[-near, far]，理论上应该重映射，但为了节约性能所有没有重映射（其范围足够接近）
```

### ComputeFogFactorZ0ToFar
可以根据着色器使用的雾的类型计算相应的雾效因子。如果着色器使用的是指数雾，该函数输出的仅为雾效因子公式中的`unity_FogParams.x * z`部分。
```hlsl
real ComputeFogFactorZ0ToFar(float z)
{
	// 线性雾
    #if defined(FOG_LINEAR)
    // factor = (end-z)/(end-start) = z * (-1/(end-start)) + (end/(end-start))
    float fogFactor = saturate(z * unity_FogParams.z + unity_FogParams.w);
    return real(fogFactor);
    
    // 指数雾EXP或EXP2
    #elif defined(FOG_EXP) || defined(FOG_EXP2)
    // factor = exp(-(density*z)^2)
    // -density * z computed at vertex
    return real(unity_FogParams.x * z);

    #else
        return real(0.0);
    #endif
}
```

#### unity_FogParams
在计算雾效因子前我们需要先了解变量 unity_FogParams。unity_FogParams 的xy分量表示指数雾两种不同模式的浓度（Density），用指数雾的因子计算；zw分量表示了因子计算公式的一部分，用于线性雾的因子计算。
```hlsl
// x = density / sqrt(ln(2)), useful for Exp2 mode
// y = density / ln(2), useful for Exp mode
// z = -1/(end-start), useful for Linear mode
// w = end/(end-start), useful for Linear mode
float4 unity_FogParams;
```

#### 线性雾 雾效因子
线性雾雾效因子公式：
```hlsl
factor = (end-z)/(end-start) = z * (-1/(end-start)) + (end/(end-start))
```
代入 unity_FogParams 的zw分量，得到 fogFactor：
```hlsl
float fogFactor = saturate(z * unity_FogParams.z + unity_FogParams.w);
```
#### 指数雾 雾效因子
指数雾雾效因子公式，其中 density 表示雾的浓度值，z 表示物体的深度值：
```hlsl
factor = exp(-(density*z)^2)
```
`ComputeFogFactorZ0ToFar`只计算了指数雾因子 factor 的`density*z`部分，将 unity_FogParams.x 作为 density 代入公式得到`density*z`：
```
（density * z) = (unity_FogParams.x * z)
```

# 五、Global IIIumination 全局光照
全局光照 (Global Illumination，简称 GI)，也可称为间接光照（Indirect Illumination）或环境光照，是指既考虑场景中直接来自光源的光照（Direct Light）又考虑经过场景中其他物体反射后的光照（Indirect Light）的一种渲染技术。

全局光照的构成我们可以理解为 全局光照(GI) = 直接光照(Direct Light) + 间接光照(Indirect Light)。

在 Untiy 的 Shader 中，GI 部分主要计算的是间接光照，当渲染实时光时需要单独计算直接光照。Unity 在计算间接光的高光时，函数名中出现较多的描述词是“Environment”（环境），所以在接下来的描述中出现的“环境光”指的也是“间接光”。

Unity 使用`GlobalIllumination`函数计算全局光照，如下：
```hlsl
half3 GlobalIllumination(BRDFData brdfData, BRDFData brdfDataClearCoat, float clearCoatMask,
    half3 bakedGI, half occlusion, float3 positionWS,
    half3 normalWS, half3 viewDirectionWS)
{
	// 反射向量
    half3 reflectVector = reflect(-viewDirectionWS, normalWS);
    // NdotV
    half NoV = saturate(dot(normalWS, viewDirectionWS));
    // Fresnel
    half fresnelTerm = Pow4(1.0 - NoV);

	// 间接光漫反射
    half3 indirectDiffuse = bakedGI;
    // 间接光高光色（环境反射）
    half3 indirectSpecular = GlossyEnvironmentReflection(reflectVector, positionWS, brdfData.perceptualRoughness, 1.0h);
	// 计算间接光高光，再加上间接光漫反射
    half3 color = EnvironmentBRDF(brdfData, indirectDiffuse, indirectSpecular, fresnelTerm);

    if (IsOnlyAOLightingFeatureEnabled())
    {
        color = half3(1,1,1); // "Base white" for AO debug lighting mode
    }

	/*
		此处省略透明图层的GI计算......
	*/

    return color * occlusion;
}
```
`GlobalIllumination`函数的计算分了三步：先计算间接光漫反射，在计算间接光高光色（环境反射），最后代入计算出完整的间接光照。接下来我们将间接光分为漫反射和高光两个部分来分析，此外我们需要先了解 bakedGI（Baked Global Illumination） 的计算过程。
## 1、烘焙全局光（bakedGI）
Unity 的烘焙全局光照系统（Baked Global Illumination System）由光照贴图（Lightmappers）、光照探针（Light Probes）和反射探针（Reflection Probes）组成。

在着色器中（以 Lit Shader 为例），烘焙全局光（Baked Global Illumination，bakedGI）特指的是着色器采样的光照贴图和球谐光照。

以下是着色器如何采样光照贴图和计算的球谐光照的过程：
```hlsl
struct Attributes
{
	......
	float2 staticLightmapUV   : TEXCOORD1;  // 静态LightmapUV
	float2 dynamicLightmapUV  : TEXCOORD2;  // 动态LightmapUV
	......
};

struct Varyings
{
	......
	// 为staticLightmap或vertexSH声明纹理坐标
    DECLARE_LIGHTMAP_OR_SH(staticLightmapUV, vertexSH, 8);   
	// 启用动态光照贴图时，为dynamicLightmapUV声明纹理坐标
#ifdef DYNAMICLIGHTMAP_ON
    float2  dynamicLightmapUV : TEXCOORD9; // Dynamic lightmap UVs
#endif
	......
};

Varyings LitPassVertex(Attributes input)
{
	......
	// 输出静态光照贴图UV
    OUTPUT_LIGHTMAP_UV(input.staticLightmapUV, unity_LightmapST, output.staticLightmapUV);
    // 输出动态光照贴图UV
#ifdef DYNAMICLIGHTMAP_ON
    output.dynamicLightmapUV = input.dynamicLightmapUV.xy * 
				    unity_DynamicLightmapST.xy + unity_DynamicLightmapST.zw;
#endif
	// 输出球谐光照
	OUTPUT_SH(output.normalWS.xyz, output.vertexSH);
	......
}

half4 LitPassFragment(Varyings input) : SV_Target
{
	......
	#if defined(DYNAMICLIGHTMAP_ON)
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.dynamicLightmapUV, input.vertexSH, inputData.normalWS);
#else
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.vertexSH, inputData.normalWS);
#endif
	......
}
```

Shader 在片元着色器中根据光照贴图的使用情况，使用 `SAMPLE_GI` 宏采样光照贴图和计算球谐光照，共有四个计算分支，源码如下：
```hlsl
// LIGHTMAP_ON：启用光照贴图关键字
// DYNAMICLIGHTMAP_ON：启用动态光照贴图关键字

// 1.启用光照贴图和动态光照贴图
#if defined(LIGHTMAP_ON) && defined(DYNAMICLIGHTMAP_ON)
#define SAMPLE_GI(staticLmName, dynamicLmName, shName, normalWSName) SampleLightmap(staticLmName, dynamicLmName, normalWSName)

// 2.只启用动态光照贴图
#elif defined(DYNAMICLIGHTMAP_ON)
#define SAMPLE_GI(staticLmName, dynamicLmName, shName, normalWSName) SampleLightmap(0, dynamicLmName, normalWSName)

// 3.只启用光照贴图
#elif defined(LIGHTMAP_ON)
#define SAMPLE_GI(staticLmName, shName, normalWSName) SampleLightmap(staticLmName, 0, normalWSName)
#else

// 4.不启用光照贴图，使用球谐光照
#define SAMPLE_GI(staticLmName, shName, normalWSName) SampleSHPixel(shName, normalWSName)
#endif
```
### （1）Lightmap 光照贴图采样
● SampleLightmap
采样光照贴图的函数。
```hlsl
// Sample baked and/or realtime lightmap. Non-Direction and Directional if available.
half3 SampleLightmap(float2 staticLightmapUV, float2 dynamicLightmapUV, half3 normalWS)
{
#ifdef UNITY_LIGHTMAP_FULL_HDR
    bool encodedLightmap = false;
#else
    bool encodedLightmap = true;
#endif

    half4 decodeInstructions = half4(LIGHTMAP_HDR_MULTIPLIER, LIGHTMAP_HDR_EXPONENT, 0.0h, 0.0h);

    // The shader library sample lightmap functions transform the lightmap uv coords to apply bias and scale.
    // However, universal pipeline already transformed those coords in vertex. We pass half4(1, 1, 0, 0) and
    // the compiler will optimize the transform away.
    half4 transformCoords = half4(1, 1, 0, 0);

    float3 diffuseLighting = 0;

#if defined(LIGHTMAP_ON) && defined(DIRLIGHTMAP_COMBINED)
    diffuseLighting = SampleDirectionalLightmap(TEXTURE2D_LIGHTMAP_ARGS(LIGHTMAP_NAME, LIGHTMAP_SAMPLER_NAME),
        TEXTURE2D_LIGHTMAP_ARGS(LIGHTMAP_INDIRECTION_NAME, LIGHTMAP_SAMPLER_NAME),
        LIGHTMAP_SAMPLE_EXTRA_ARGS, transformCoords, normalWS, encodedLightmap, decodeInstructions);
#elif defined(LIGHTMAP_ON)
    diffuseLighting = SampleSingleLightmap(TEXTURE2D_LIGHTMAP_ARGS(LIGHTMAP_NAME, LIGHTMAP_SAMPLER_NAME), LIGHTMAP_SAMPLE_EXTRA_ARGS, transformCoords, encodedLightmap, decodeInstructions);
#endif

#if defined(DYNAMICLIGHTMAP_ON) && defined(DIRLIGHTMAP_COMBINED)
    diffuseLighting += SampleDirectionalLightmap(TEXTURE2D_ARGS(unity_DynamicLightmap, samplerunity_DynamicLightmap),
        TEXTURE2D_ARGS(unity_DynamicDirectionality, samplerunity_DynamicLightmap),
        dynamicLightmapUV, transformCoords, normalWS, false, decodeInstructions);
#elif defined(DYNAMICLIGHTMAP_ON)
    diffuseLighting += SampleSingleLightmap(TEXTURE2D_ARGS(unity_DynamicLightmap, samplerunity_DynamicLightmap),
        dynamicLightmapUV, transformCoords, false, decodeInstructions);
#endif

    return diffuseLighting;
}

// Legacy version of SampleLightmap where Realtime GI is not supported.
half3 SampleLightmap(float2 staticLightmapUV, half3 normalWS)
{
    float2 dummyDynamicLightmapUV = float2(0,0);
    half3 result = SampleLightmap(staticLightmapUV, dummyDynamicLightmapUV, normalWS);
    return result;
}
```

### （2）Spherical Harmonic 球谐光照
● OUTPUT_SH
当着色器使用 SH球谐光照时，输出 SH 的顶点采样：
```HLSL
#define OUTPUT_SH(normalWS, OUT) OUT.xyz = SampleSHVertex(normalWS)
```
如果着色器使用 LightMap，输出为空：
```HLSL
#define OUTPUT_SH(normalWS, OUT)
```
●  SampleSHVertex
逐顶点进行SH采样。
```HLSL
// SH Vertex Evaluation. Depending on target SH sampling might be
// done completely per vertex or mixed with L2 term per vertex and L0, L1
// per pixel. See SampleSHPixel
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
●  SampleSHPixel
进行SH采样，输出球谐光照。
```hlsl
half3 SampleSHPixel(half3 L2Term, half3 normalWS)
{
#if defined(EVALUATE_SH_VERTEX)
    return L2Term;
#elif defined(EVALUATE_SH_MIXED)
    half3 res = SHEvalLinearL0L1(normalWS, unity_SHAr, unity_SHAg, unity_SHAb);
#ifdef UNITY_COLORSPACE_GAMMA
    res = LinearToSRGB(res);
#endif
    return max(half3(0, 0, 0), res);
#endif

    // Default: Evaluate SH fully per-pixel
    return SampleSH(normalWS);
}
```
●  SampleSH
```HLSL
// Samples SH L0, L1 and L2 terms
half3 SampleSH(half3 normalWS)
{
    // LPPV is not supported in Ligthweight Pipeline
    real4 SHCoefficients[7];
    SHCoefficients[0] = unity_SHAr;
    SHCoefficients[1] = unity_SHAg;
    SHCoefficients[2] = unity_SHAb;
    SHCoefficients[3] = unity_SHBr;
    SHCoefficients[4] = unity_SHBg;
    SHCoefficients[5] = unity_SHBb;
    SHCoefficients[6] = unity_SHC;

    return max(half3(0, 0, 0), SampleSH9(SHCoefficients, normalWS));
}
```

```HLSL
real3 SHEvalLinearL2(real3 N, real4 shBr, real4 shBg, real4 shBb, real4 shC)
{
    real3 x2;
    // 4 of the quadratic (L2) polynomials
    real4 vB = N.xyzz * N.yzzx;
    x2.r = dot(shBr, vB);
    x2.g = dot(shBg, vB);
    x2.b = dot(shBb, vB);

    // Final (5th) quadratic (L2) polynomial
    real vC = N.x * N.x - N.y * N.y;
    real3 x3 = shC.rgb * vC;

    return x2 + x3;
}
```
## 2、间接光漫反射（Indirect Diffuse）
```hlsl
half3 indirectDiffuse = bakedGI;
```
Unity 的间接光漫反射使用的`bakedGI`就是烘焙光照（LightMap）和球谐光照（SH）。
## 3、间接光高光（Indirect Specular）
Untiy 
**● GlossyEnvironmentReflection**
计算环境反射，间接光照的漫反射部分。
```hlsl
half3 GlossyEnvironmentReflection(half3 reflectVector, float3 positionWS, half perceptualRoughness, half occlusion)
{
#if !defined(_ENVIRONMENTREFLECTIONS_OFF)
    half3 irradiance;

#ifdef _REFLECTION_PROBE_BLENDING // 启用反射探针混合
    irradiance = CalculateIrradianceFromReflectionProbes(reflectVector, positionWS, perceptualRoughness);
#else // 不启用反射探针混合，采样天空盒（Skybox Material）
	// 使用反射探针的盒体投影
	#ifdef _REFLECTION_PROBE_BOX_PROJECTION
	    reflectVector = BoxProjectedCubemapDirection(reflectVector, positionWS, unity_SpecCube0_ProbePosition, unity_SpecCube0_BoxMin, unity_SpecCube0_BoxMax);
	#endif // _REFLECTION_PROBE_BOX_PROJECTION

	// 采样Environment设置的天空盒（Skybox Material）
    half mip = PerceptualRoughnessToMipmapLevel(perceptualRoughness);
    half4 encodedIrradiance = half4(SAMPLE_TEXTURECUBE_LOD(unity_SpecCube0, samplerunity_SpecCube0, reflectVector, mip));

	#if defined(UNITY_USE_NATIVE_HDR)  // Unity使用原生的HDR，官方文档没有这个宏，不清楚具体作用。
	    irradiance = encodedIrradiance.rgb;
	#else
		// 如果Unity未使用原生的HDR，则需要进行HDR解码
	    irradiance = DecodeHDREnvironment(encodedIrradiance, unity_SpecCube0_HDR);
	#endif // UNITY_USE_NATIVE_HDR
#endif // _REFLECTION_PROBE_BLENDING
    return irradiance * occlusion;
#else
    return _GlossyEnvironmentColor.rgb * occlusion;
#endif // _ENVIRONMENTREFLECTIONS_OFF
}
```
URP 计算环境的反射光照主要看是否启用反射探针混合。如果启用混合，

**● EnvironmentBRDFSpecular**
计算间接光的高光（环境高光），再加上间接光照的漫反射（环境反射），得到完整的间接光照。
```hlsl
// Computes the specular term for EnvironmentBRDF
half3 EnvironmentBRDFSpecular(BRDFData brdfData, half fresnelTerm)
{
    float surfaceReduction = 1.0 / (brdfData.roughness2 + 1.0);
    return half3(surfaceReduction * lerp(brdfData.specular, brdfData.grazingTerm, fresnelTerm));
}

half3 EnvironmentBRDF(BRDFData brdfData, half3 indirectDiffuse, half3 indirectSpecular, half fresnelTerm)
{
	// brdfData.diffuse = albedo * oneMinusReflectivity
    half3 c = indirectDiffuse * brdfData.diffuse;
    // BRDF高光
    c += indirectSpecular * EnvironmentBRDFSpecular(brdfData, fresnelTerm);
    return c;
}
```
## 4、探针混合
URP 支持探针混合，但每个物体至多受到两个反射探针影响。
使用 URP 的`CalculateIrradianceFromReflectionProbes`函数可以计算两个反射探针混合的辐照度（Irradiance）。

# 六、空间坐标
## 1、坐标空间变换
坐标变换核心阶段：模型空间 OS -> 世界空间 WS -> 观察空间 VS -> 裁剪空间 CS -> NDC空间 -> 屏幕空间 SS
此处指分析前三个阶段，NDC空间和屏幕空间变换在后续分析。
### （1）模型空间 OS -> 世界空间 WS
通过`UNITY_MATRIX_M`矩阵完成模型空间到世界空间的变换：
```hlsl
float3 positionWS = mul(UNITY_MATRIX_M, float4(positionOS, 1.0)).xyz;
```

在Unity中可以调用`TransformObjectToWorld`函数完成变换：
```hlsl
float3 TransformObjectToWorld(float3 positionOS)
{
	// 判断着色器是否启用光线追踪DXR
    #if defined(SHADER_STAGE_RAY_TRACING)
    return mul(ObjectToWorld3x4(), float4(positionOS, 1.0)).xyz;
    #else
    return mul(GetObjectToWorldMatrix(), float4(positionOS, 1.0)).xyz;
    #endif
}
```
`GetObjectToWorldMatrix`返回内置矩阵 UNITY_MATRIX_M；

### （2）世界空间 WS -> 观察空间VS
通过`UNITY_MATRIX_V`矩阵完成世界空间到观察空间的变换：
```hlsl
float3 positionVS = mul(UNITY_MATRIX_V, float4(positionWS, 1.0)).xyz;
```
在Unity中可以调用`TransformWorldToView`函数完成变换：
```hlsl
float3 TransformWorldToView(float3 positionWS)
{
    return mul(GetWorldToViewMatrix(), float4(positionWS, 1.0)).xyz;
}
```
`TransformWorldToView`返回内置矩阵 UNITY_MATRIX_V；

### （3）观察空间 VS -> 裁剪空间CS
通过`UNITY_MATRIX_VP`矩阵完成观察空间到裁剪空间的变换：
```hlsl
float3 positionCS = mul(UNITY_MATRIX_VP, float4(positionWS, 1.0)).xyz;
```
在Unity中可以调用`TransformWorldToHClip`函数完成变换：
```hlsl
float4 TransformWorldToHClip(float3 positionWS)
{
    return mul(GetWorldToHClipMatrix(), float4(positionWS, 1.0));
}
```
`GetWorldToHClipMatrix`返回内置矩阵 UNITY_MATRIX_VP；
## 2、PositionCS
顶点着色器中，裁剪空间坐标（PositionCS）是顶点经过投影矩阵变换后的空间坐标，其分量含义和范围如表（w为齐次坐标分量）：

| 分量  | 含义       | 值/范围                        |
| --- | -------- | --------------------------- |
| xy  | 裁剪空间坐标位置 | \[-w, w]                    |
| z   | 深度值      | \[-w, w]（正交投影时为\[near,far]） |
| w   | 存储原始深度数据 | -positionVS.z               |

在顶点输出结构体中定义 PositionCS 时，通常都会使用 SV_POSITION 语义：
```hlsl
float4 positionCS : SV_POSITION;
```
而关于 PositionCS 的语义定义，有一个很容易被忽视的问题：使用 SV_POSITION 语义和使用 TEXCOORD 语义的 PositionCS 在传入片元着色器后会出现明显的差异。
SV_POSITION 是一个非常特殊的系统值语义，当 positionCS 传入片元着色器时，Unity 会对 SV_POSITION 进行一系列操作，与使用 TEXCOORD 语义手动传递的 positionCS 差异如下：

| 特性     | 手动传递 PositionCS | 使用SV_POSITION语义 |
| ------ | --------------- | --------------- |
| 插值方式   | 线性插值            | 透视矫正插值          |
| 平台Y轴方向 | 保持OpenGL标准      | 自动适配            |
| 深度优化   | 保留原始Z值          | 应用REVERSED_Z策略  |
| 屏幕映射   | 需要手动计算透视除法和视口映射 | 隐式转换到屏幕空间       |
| 视椎体裁剪  | 保留被裁剪顶点的原始数据    | 自动剔除不可见区域坐标     |
经过 Unity 的操作后，片元着色器接收的 SV_POSITION 的各个分量实际上是：

| 分量  |                                      |
| --- | ------------------------------------ |
| xy  | 片元在屏幕上的像素位置                          |
| z   | 非线性z深度                               |
| w   | 透视：相机空间深度（即观察空间 PositionVS.w）；正交：1.0 |
**UNITY_REVERSED_Z** 
UNITY_REVERSED_Z 是Unity引擎定义的平台深度方向标识宏，用于处理不同图形API的深度缓冲区差异。

| 平台                          | 深度缓冲范围        |
| --------------------------- | ------------- |
| DX12、Metal、Vulkan、PS5等      | 近裁面：1 / 远裁面：0 |
| OpenGL、OpenGL ES、WebGL 1.0等 | 近裁面：0 / 远裁面：1 |

## 3、PositionNDC
**（1）NDC概念**
标准化设备坐标（Normalized Device Coordinates，NDC）是一个计算机图形学概念，指将三维模型通过投影变换映射到二维屏幕空间后得到的坐标系统。NDC的主要目的是为了标准化不同设备（如不同的显示器）之间的坐标表示，使得图形渲染可以在统一的坐标系下进行。

在标准NDC中，-x和y坐标的范围通常在\[-1, 1]之间， - z坐标的范围在\[0, 1]之间（DX：\[0, 1]；OpenGL：\[-1, 1]）。而在透视投影下，还需要考虑顶点的齐次坐标分量w，-x和y坐标的范围通常在\[-w, w]之间， - z坐标的范围在\[0, w]之间。

**（2）PositionNDC计算**
裁剪空间坐标（PositionCS）除以裁剪空间坐标的w分量：
```hlsl
float4 positionNDC = positionCS / positionCS.w; 
```

在 URP 管线中，Unity 使用了一个`GetVertexPositionInputs`函数计算空间坐标变化，而其中有一个名为`input.positionNDC`的变量，如下：
```hlsl
VertexPositionInputs GetVertexPositionInputs(float3 positionOS)
{
	......
	
    float4 ndc = input.positionCS * 0.5f;
    input.positionNDC.xy = float2(ndc.x, ndc.y * _ProjectionParams.x) + ndc.w;
    input.positionNDC.zw = input.positionCS.zw;

    return input;
}
```
注意：此处的 NDC 计算就是已经弃用的函数`ComputeScreenPos`，两者的计算公式完全相同。
虽然 positionNDC 名字里带有NDC，但它严格意义上并非表示标准化设备坐标NDC，而是“Homogeneous Normalized Device Coordinates”（近似NDC）。标准NDC的xy分量的范围是\[-1, 1]，考虑w分量则应该是\[-w, w]，而这个近似NDC的范围经过计算后实际为\[0, w]，只需除以自身w分量即可得到屏幕空间坐标，这个近似应该可以称为屏幕空间坐标（ScreenPos）。
## 4、ScreenPos
在Unity中可以使用`GetVertexPositionInputs`函数和`ComputeScreenPos`函数获取屏幕空间坐标：
（1）GetVertexPositionInputs
根据上一点对 PositionNDC 的分析，在URP管线中可以在顶点着色器中将`vertexInput.positionNDC`作为屏幕空间坐标（ScreenPos）输出：
```hlsl
output.screenPos = vertexInput.positionNDC;
```
其计算过程如下：
```hlsl
// 1.将positionCS坐标从[-w,w]映射到[-0.5w,0.5w]
float4 positionNDC = input.positionCS * 0.5f; 
// 2.平移positionCS坐标到[0,w]
positionNDC.xy = float2(positionNDC.x, positionNDC.y * _ProjectionParams.x) + positionNDC.w; 
// 3.保留positionCS的zw分量
positionNDC.zw = positionCS.zw;
```
**\_ProjectionParams**：Unity内置的投影参数，用来处理投影矩阵的翻转。\_ProjectionParams.x 为1，如果投影式是翻转的则为-1。

（2）ComputeScreenPos
```hlsl
output.screenPos = ComputeScreenPos(vertexInput.positionCS);
```
PS：在URP中Unity已经弃用`ComputeScreenPos`，该函数被整合到了`GetVertexPositionInputs`中。虽然官方HLSL文件显示弃用`ComputeScreenPos`，但实际上依旧可以调用。
## 5、ScreenUV
在Unity中有两种计算屏幕UV（ScreenUV）的方法：
- 【方法1】使用PositionNDC计算：
```hlsl
half4 screenPos = input.screenPos / input.screenPos.w;
```
- 【方法2】用PositionCS除以屏幕分辨率：
```hlsl
half2 normalizedScreenUV = input.positionCS.xy * rcp(_ScaledScreenParams.xy);    
```
**rcp**：表示取参数的倒数。
或者直接调用`GetNormalizedScreenSpaceUV`函数：
```hlsl
float2 GetNormalizedScreenSpaceUV(float2 positionCS)
{
    float2 normalizedScreenSpaceUV = positionCS.xy * rcp(GetScaledScreenParams().xy);
    TransformNormalizedScreenUV(normalizedScreenSpaceUV);
    return normalizedScreenSpaceUV;
}
```

Q：为什么 positionCS 除以 \_ScaledScreenParams（屏幕分辨率）可以得到ScreenUV呢？
A：传入片元且由输出结构体定义的 input.positionCS 使用的是 SV_POSITION 语义（详见PositionCS）。根据 SV_POSITION 的语义作用，input.positionCS.xy 的实际值是片元的像素位置，因此 input.positionCS.xy 除以屏幕分辨率即为 ScreenUV（0到1区间）。

如果只需要屏幕空间UV（ScreenUV），可以跳过顶点着色器中的屏幕空间坐标（ScreenPos）计算，绕开手动 PositionNDC。

# 内置着色器变量
## ● \_ProjectionParams
Unity内置的投影参数，用来处理投影矩阵的翻转。
```HLSL
// x = 1 or -1 (-1 if projection is flipped) 
// y = near plane 
// z = far plane 
// w = 1/far plane 

float4 _ProjectionParams;
```
- x：如果投影式是翻转的，x=-1；
- y：近裁平面在view空间（相机空间）的z值，数值上等于相机设置中的近裁平面的值；
- z：远裁平面在view空间（相机空间）的z值，数值上等于相机设置中的远裁平面的值；
- w：1/z；
