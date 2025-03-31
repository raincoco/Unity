URP Lit Shader / URP版本：12.1.10
# 一、ShaderLab
援引《Unity Documentation》
## 1、ShaderLab 语法结构
Unity 中的所有的着色器文件都是用名为“ShaderLab”的声明性语言编写的。在文件中，通过嵌套括号 语法声明着色器的各个描述，例如，Material Inspector 中显示的着色器属性，要执行哪些类型的硬件回退，以及要使用哪些类型的混合模式。实际着色器代码 编写在同一着色器文件中的 `CGPROGRAM` 代码片段中。

**ShaderLab 语法：**
```hlsl
Shader "name" { [Properties] Subshaders [Fallback] [CustomEditor] }
```

```hlsl
Shader "name" 
{ 
	Properties
	{
		...
	} 
	Subshaders
	{
		...
	} 
	Fallback "FallbackShader"
	CustomEditor "ShaderGUI"
}
```
- Properties：着色器可以具有属性列表。着色器中声明的所有属性都会显示在 Unity 中的材质检视面板中。典型的属性包括对象颜色、纹理或者着色器要使用的任意值。
- Subshaders：子着色器。每个着色器均包含一组子着色器且必须至少包含一个子着色器。
- Fallback：回退着色器。加载着色器时，Unity 将检查子着色器的列表，并选取最终用户的机器支持的第一个子着色器。如果不支持任何子着色器，Unity 将尝试使用回退着色器。
- CustomEditor：自定义编辑界面。 
## 2、ShaderLab：Properties
**（1）Properties 代码块可定义属性**
- **数字和滑动条**
```hlsl
name ("display name", Range (min, max)) = number
name ("display name", Float) = number
name ("display name", Int) = number
```
- **颜色和矢量**
```hlsl
name ("display name", Color) = (number,number,number,number)
name ("display name", Vector) = (number,number,number,number)
```
- **纹理**
```hlsl
name ("display name", 2D) = "defaulttexture" {}
name ("display name", Cube) = "defaulttexture" {}
name ("display name", 3D) = "defaulttexture" {}
```

**（2）属性特性与检查器（Inspector）**
在属性前面可指定可选的特性，这些Unity可识别的特性可以指示您自己的 MaterialPropertyDrawer 类 来控制它们在材质检视面板中的呈现方式。Unity 可以识别的特性如下：

| 特性                 | 作用                                                                                                                                                                                                                  |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| \[HideInInspector] | 在材质球检查器（Material Inspector）隐藏该属性。                                                                                                                                                                                   |
| \[NoScaleOffset]   | 不显示该纹理属性的 Scale 和 Offset                                                                                                                                                                                            |
| \[Normal]          | 表示该纹理属性需要是 Normal Map。                                                                                                                                                                                              |
| \[HDR]             | 表示该纹理属性需要高动态范围（HDR）纹理。                                                                                                                                                                                              |
| \[Gamma]           | 表示在 UI 中将浮点/矢量属性指定为 sRGB 值（就像颜色一样），并且可能需要根据使用的颜色空间进行转换。                                                                                                                                                             |
| \[PerRendererData] | 表示属性将以 MaterialPropertyBlock 的形式来自表示每个渲染器数据。材质检查器将这些属性显示为只读。                                                                                                                                                        |
| \[MainTexture]     | 表示属性是材质的主要纹理。默认情况下，Unity 将属性名为_MainTex 的纹理视为主纹理。                                                                                                                                                                    |
| \[MainColor]       | 表示一个属性 (property) 是材质的主色。默认情况下，Unity 将属性 (property) 名称为  *_Color* 的颜色视为主色。如果您的颜色具有其他属性 (property) 名称，但您希望 Unity 将这个颜色视为主色，请使用此属性 (attribute)。如果您多次使用此属性 (attribute)，则 Unity 会使用第一个属性 (property)，而忽略后续属性 (property)。 |
## 3、ShaderLab：SubShader
Unity 中的每个着色器都包含一个子着色器列表，且列表中至少包含一个子着色器（SubShader）。Unity 在需要渲染 Mesh 时会选择第一个子着色器。

**ShaderLab 语法：**
```hlsl
Subshader { [Tags] [CommonState] Passdef [Passdef ...]}
```
子着色器可以定义多个可选标签`Tags`、多种通用状态`CommonState`和通道定义列表`Passdef`。子着色器定义渲染通道的列表，并且可选择性地设置所有通道共同的任意状态。此外，还可以设置子着色器专用的标签。
## 4、ShaderLab：SubShader Tags
SubShader Tags（子着色器标签）就是数据的键/值对。Unity 使用预定义键和值确定如何以及何时使用给定子着色器。

语法：
```hlsl
Tags { "TagName1" = "Value1" "TagName2" = "Value2" }
```
注意，子着色器和通道都使用 `Tags` 代码块，但其工作方式不同，不能使用对方的标签：
- 子着色器标签（SubShader Tags）只能在着色器中使用，要定义子着色器标签，请将 `Tags` 代码块置于 `SubShader` 代码块内部，但是在 `Pass` 代码块外部。
- 通道标签（Pass Tags）仅能在通道中使用，要定义通道标签，请将 `Tags` 代码块置于 `Pass` 代码块内部。

### （1）RenderPipeline 标签
`RenderPipeline` 标签向 Unity 告知子着色器是否与通用渲染管线 (URP) 或高清渲染管线 (HDRP) 兼容。

**语法和有效值**
```hlsl
SubShader {
	Tags { "RenderPipeline" = "UniversalRenderPipeline" }
}
```

| 签名                           | 功能                                |
| ---------------------------- | --------------------------------- |
| “RenderPipeline” = “\[name]” | 向 Unity 告知此子着色器是否与 URP 或 HDRP 兼容。 |

| 参数      | 值                            | 功能                     |
| ------- | ---------------------------- | ---------------------- |
| \[name] | UniversalRenderPipeline      | 此子着色器仅与 URP 兼容。        |
|         | HighDefinitionRenderPipeline | 此子着色器仅与 HDRP 兼容。       |
|         | （任何其他值，或未声明）                 | 此子着色器与 URP 和 HDRP 不兼容。 |
### （2）Queue 标签
Queue 标签向 Unity 告知将被它渲染的几何体的渲染队列。渲染队列是确定 Unity 渲染几何体的顺序的因素之一。


**语法和有效值**
```hlsl
SubShader {
	Tags { "Queue" = "Transparent" }
}
```

| 签名                                    | 功能                                                                        |
| ------------------------------------- | ------------------------------------------------------------------------- |
| “Queue” = “\[queue name]”             | 向 Unity 告知此子着色器是否与 URP 或 HDRP 兼容。                                         |
| “Queue” = “\[queue name] + \[offset]” | 在相对于命名队列的给定偏移处使用未命名队列。<br>这种用法十分有用的一种示例情况是透明的水，它应该在不透明对象之后绘制，但是在透明对象之前绘制。 |

| 参数            | 值           | 功能                             |
| ------------- | ----------- | ------------------------------ |
| \[queue name] | Background  | 指定背景渲染队列。                      |
|               | Geometry    | 指定几何体渲染队列。                     |
|               | AlphaTest   | 指定 AlphaTest 渲染队列。             |
|               | Transparent | 指定透明渲染队列。                      |
|               | Overlay     | 指定覆盖渲染队列。                      |
| \[offset]     | 整数          | 指定 Unity 渲染未命名队列处的索引（相对于命名队列）。 |
### （3）RenderType 标签
使用 RenderType 标签可将着色器标识为可替换子着色器。RenderType 标签涉及到“光照着色器替换”的问题，在这里我们可以还是可以将它直接解释为着色器的渲染类型。


**语法和有效值**
```hlsl
SubShader {
	Tags { "Queue" = "Transparent" }
}
```

| 签名            | 标签值                   | 描述                             |
| ------------- | --------------------- | ------------------------------ |
| \[renderType] | Opaque                | 大部分着色器（法线、自发光、反射和地形着色器）。       |
|               | Transparent           | 大部分半透明着色器（透明、粒子、字体和地形附加通道着色器）。 |
|               | TransparentCutout     | 遮罩透明度着色器（透明、镂空两个通道植被着色器）。      |
|               | Background            | 天空盒着色器。                        |
|               | Overlay               | 光环、光晕着色器。                      |
|               | TreeOpaque            | 地形引擎树皮。                        |
|               | TreeTransparentCutout | 地形引擎树叶。                        |
|               | TreeBillboard         | 地形引擎公告牌树。                      |
|               | Grass                 | 地形引擎草。                         |
|               | GrassBillboard        | 地形引擎公告牌草。                      |
### （4）UniversalMaterialType 标签
Unity 在延迟渲染路径中使用此标签，指示着色器类型。

**语法和有效值**
```hlsl
SubShader {
	Tags { "UniversalMaterialType" = "Lit" }
}
```

| 签名        | 功能                                                                                                  |
| --------- | --------------------------------------------------------------------------------------------------- |
| Lit       | 此值指示着色器类型为光照 (Lit)。在 G 缓冲区通道期间，Unity 使用模板来标记使用光照着色器类型（镜面反射模型为 PBR）的像素。<br>如果未在通道中设置标签，Unity 默认使用此值。 |
| SimpleLit | 此值指示着色器类型为简单光照 (SimpleLit)。在 G 缓冲区通道期间，Unity 使用模板来标记使用简单光照着色器类型（镜面反射模型为 Blinn-Phong）的像素。            |
### （5）IgnoreProjector 标签
`IgnoreProjector` 子着色器标签向 Unity 告知几何体是否受投影器影响。这对于排除投影器不兼容的半透明几何体多半很有用。
根据《Unity Documentation 2022.3》中的说明，IgnoreProjector 只在内置渲染管线中生效，但 URP 管线的 Lit Shader 仍使用了该标签。

**语法和有效值**
```hlsl
SubShader {
	Tags { "IgnoreProjector" = "True" }
}
```

| 签名                                    | 功能                                                                        |
| ------------------------------------- | ------------------------------------------------------------------------- |
| “IgnoreProjector” = “\[state]”        | Unity 在渲染此几何体时是否忽略投影器。                                                    |

| 签名       | 值     | 功能                           |
| -------- | ----- | ---------------------------- |
| \[state] | True  | Unity 在渲染此几何体时忽略投影器。<br>     |
|          | False | Unity 在渲染此几何体时不会忽略投影器。这是默认值。 |
|          |       |                              |
### （6）ShaderModel 标签
指示子着色器的优化渲染引擎模式。Shader Model 的等级越高，Shader 能使用的运算指令数目、寄存器个数等特性和能力越强。在 URP 中通常我们通常将 Shader Model 设置为4.5。

**语法和有效值**
```hlsl
SubShader {
	Tags { "ShaderModel" = "4.5" }
}
```
## 5、ShaderLab：Pass 
子着色器渲染通道，每个 Pass 都会将所有的游戏对象（GameObject）几何体渲染一次。Pass 有三种定义，分别是常规 Pass、Use Pass 和 Grab Pass。
### 5 .1、Pass
基本渲染通道，一个Pass可以定义一个名字（Name）和任意数量的标签（Tags），设置图形硬件的各种状态（RenderSetup），以及一个 HLSL 代码片段。

**ShaderLab 语法：**
```hlsl
Pass { [Name and Tags] [RenderSetup] }
```

```hlsl
Pass
{
	// 名字
	Name "Name"
	
	// 标签
	Tags{}
	
	// 渲染状态设置
	[RenderSetup]

	// HLSL代码片段
	HLSLPROGRAM
	......
	ENDHLSL
}
```
- Name：通道名称。
- Tags：标签。
- RenderSetup：渲染状态设置。
#### （1）Tags 标签
通道（Pass）使用标签来告知它们期望何时以何种方式被渲染到渲染引擎。

语法：
```hlsl
Tags { "TagName1" = "Value1" "TagName2" = "Value2" }
```
标签是数据的键/值对。在渲染通道中，Unity 使用预定义的标签和值来确定如何以及何时渲染给定的通道。还可以使用自定义值创建自己的自定义通道标签，并从 C# 代码访问它们。
特别注意，以下由 Unity 识别的标签必须位于 Pass 部分中，不能在 SubShader 中使用。

● **LightMode 标签**
Pass 的 LightMode 标签可以让管线确定在执行渲染管线的不同部分时使用哪个通道。如果没有在管线中设置 LightMode 标签，URP 将使用该通道的 SRPDefaultUnlit 标签值。

| 属性                       | 作用                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **UniversalForward**<br> | 该通道会渲染对象几何体并评估所有光源影响。URP 在前向渲染路径中使用此标签值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **UniversalGBuffer**     | 该通道会渲染对象几何体，但不评估任何光源影响。在 Unity 必须在延迟渲染路径中执行的通道中使用此标签值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **UniversalForwardOnly** | 该通道会渲染对象几何体并评估所有光源影响，类似于 **LightMode** 具有 **UniversalForward** 值的情况。但与 **UniversalForward** 的区别在于，URP 可以将该通道用于前向渲染路径和延迟渲染路径两者。<br><br>● 如果在 URP 使用延迟渲染路径时某个通道必须使用前向渲染路径来渲染对象，请使用此值。例如，如果 URP 使用延迟渲染路径来渲染某个场景，并且该场景包含的某些对象具有不适合 G 缓冲区的着色器数据（例如透明涂层法线），则应使用此标签。<br>● 如果着色器必须在前向渲染路径和延迟渲染路径两者中进行渲染，请使用 `UniversalForward` 和 `UniversalGBuffer` 标签值声明两个通道。                                    <br>● 如果着色器必须使用前向渲染路径进行渲染，而不管 URP 渲染器使用的渲染路径如何，请仅声明一个 `LightMode` 标签设置为 `UniversalForwardOnly` 的通道。<br>● 如果使用了 SSAO 渲染器功能，请添加一个 `LightMode` 标签设置为 `DepthNormalsOnly` 的通道。 |
| **DepthNormalsOnly**     | 将此值与延迟渲染路径中的 `UniversalForwardOnly` 结合使用。此标签允许 Unity 在深度和法线预通道中渲染着色器。在延迟渲染路径中，如果缺少具有 `DepthNormalsOnly` 标签值的通道，则 Unity 不会在网格周围生成环境光遮挡。                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **Universal2D**          | 该通道会渲染对象并评估 2D 光源影响。URP 在 2D 渲染器中使用此标签值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **ShadowCaster**         | 该通道从光源的角度将对象深度渲染到阴影贴图或深度纹理中。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| **DepthOnly**            | 该通道仅从摄像机的角度将深度信息渲染到深度纹理中。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| **Meta**                 | Unity 仅在 Unity 编辑器中烘焙光照贴图时执行该通道。Unity 在构建播放器时会从着色器中剥离该通道。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| **SRPDefaultUnlit**      | 渲染对象时，使用此 `LightMode` 标签值绘制额外的通道。<br>应用示例：绘制对象轮廓。此标签值对前向渲染路径和延迟渲染路径均有效。当通道没有 `LightMode` 标签时，URP 使用此标签值作为默认值。                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

● **PassFlags 标签**
使用 PassFlags 标签可以更改渲染管线向通道传递数据的方式，该标签的值为空格分隔的标志名称。
当前 Unity 仅支持标志名称 OnlyDirectional。当在 ForwardBase 通道类型中使用时，此标志的作用是仅允许主方向光和环境光/光照探针数据传递到着色器。这意味着非重要光源的数据将不会传递到顶点光源或球谐函数着色器变量。

● **RequireOptions 标签**
表示仅在满足某些外部条件时才渲染该通道，该标签的值为空格分隔的选项字符串。当前 Unity 仅支持 SoftVegetation，仅当 Quality 窗口中开启了 Soft Vegetation 时才渲染此通道。
#### （2）RenderSetup 渲染状态设置
- **Cull**
```hlsl
Cull Back | Front | Off
```
设置多边形剔除模式。GPU会根据多边形面对摄像机的方向剔除指定方向的多边形，可设置的剔除模式有剔除正面、剔除背面或不剔除三种模式。

- **ZTest**
```hlsl
ZTest (Less | Greater | LEqual | GEqual | Equal | NotEqual | Always)
```
设置深度缓冲区测试模式。

- **ZWrite**
```hlsl
ZWrite On | Off
```
设置深度缓冲区写入模式，确定在渲染过程中是否更新深度缓冲区内容。禁用 ZWrite 会导致不正确的深度排序，通常 ZWrite 对不透明对象启用，对半透明对象禁用。

- **Offset**
```hlsl
Offset OffsetFactor, OffsetUnits
```
设置 Z 缓冲区深度偏移。

- **Blend**
```hlsl
Blend sourceBlendMode destBlendMode
Blend sourceBlendMode destBlendMode, alphaSourceBlendMode alphaDestBlendMode
BlendOp colorOp
BlendOp colorOp, alphaOp
AlphaToMask On | Off
```
设置 Alpha 混合、Alpha 操作和 alpha-to-coverage 模式。

- **Conservative Rasterization** 
```hlsl
Conservative True | False
```
打开和关闭保守光栅化。使用前，请通过SystemInfo.supportsConserviveRaster检查硬件支持。

- **ColorMask**
```hlsl
ColorMask RGB | A | 0 | R、G、B、A 的任意组合
```
设置颜色通道写入遮罩。写入 ColorMask 0 可关闭对所有颜色通道的渲染。默认模式是 写入所有通道 (RGBA)，但是对于某些特殊效果，您可能希望不修改某些通道，或完全禁用颜色 写入。
使用多渲染目标 (MRT) 渲染时，可通过在末尾添加索引（0 到 7）来为每个渲染目标设置不同的颜色遮罩。例如，ColorMask RGB 3 将使渲染目标 #3 仅写入到 RGB 通道。
#### （3）HLSL代码段
HLSL 着色器代码需写在 HLSLPROGRAM 和 ENDHLSL 关键字之间。
### 5 .2、Use Pass
UsePass 命令使用来自另一个着色器的指定通道。

**ShaderLab 语法：**
```hlsl
UsePass "ShaderName/PassName"
```
### 5. 3、Grab Pass
GrabPass 是一种特殊通道类型，它把即将绘制对象时的屏幕内容抓取到纹理中，在后续通道中可直接使用此纹理。

**ShaderLab 语法：**
形式一：将当前屏幕内容抓取到某个纹理中。在随后的通道中可通过 \_GrabTexture 名称访问该纹理。这种抓取通道的形式将为使用它的每个对象执行耗时的屏幕抓取操作。
```hlsl
GrabPass { "TextureName" }
```
形式二：可将当前屏幕内容抓取到给定名为 TextureName 的纹理中，但仅为使用 TextureName 名称的第一个对象在每一帧执行一次该操作。在后续通道（Pass）中可通过 TextureName 访问该纹理。场景中有多个对象在使用 GrabPass 时，这种方法更高效。
```hlsl
GrabPass { "TextureName" }
```

## 6、ShaderLab：Fallback
在所有子着色器的后面可定义 Fallback，当没有任何子着色器能在此硬件上运行时，则尝试使用 Fallback 指定的着色器。

**ShaderLab 语法：**
设置回退着色器：
```hlsl
Fallback "name"
```
不设置回退的着色器，即使没有子着色器可以在此硬件上运行，也不会进行回退并且不会显示警告：
```hlsl
Fallback Off
```
## 7、ShaderLab：CustomEditor
自定义着色器 GUI。

**ShaderLab 语法：**
```hlsl
CustomEditor "name"
```
## 8、Shader variants
### （1）Shader variants 概念
在 Shader 的预编译阶段，可以使用`#pragma multi_compile`和`#pragma shader_feature`两条指令来为 Shader 衍生不同的变体版本，并使用 shader keywords（关键字）配置变体，这就是Unity提供的概念 Shader variants（Shader 变体）。

Q：如何控制 Shader 使用哪个变体？
A：我们可以在脚本中通过 Material.EnableKeyWord 和 Shader.EnableKeyword 来使用指定关键字（Keyword）的变体。也可以在 GUI 中通过`CoreUtils.SetKeyword`来设置关键字。
### （2）变体编译数量
Shader 会根据定义 multi_compile 的数量和关键字数量生成不同的变体，变体数 = multi_compile数 * 关键字数。例如：
```hlsl
#pragma multi_compile A B C 
#pragma multi_compile D E
```
生成6中变体组合 (A,D)、(A,E) 、(BD)、 (B,E) 、(C,D)、 (C,E)。如果定义的 multi_compile 和关键字很多，生成的变体组合就会多到恐怖。
### （3）创建着色器变体
Unity 有 program 四条指令可以用来处理着色器变体。

| 语句                             | 功能                                                 |
| ------------------------------ | -------------------------------------------------- |
| `#pragma multi_compile`        | 为给定的关键字创建着色器变体。multi_compile 着色器的未使用变体会包含在游戏构建中。   |
| `#pragma multi_compile_local`  |                                                    |
| `#pragma shader_feature`       | 为给定的关键字创建着色器变体。shader_feature 着色器的未使用变体不会包含在游戏构建中。 |
| `#pragma shader_feature_local` |                                                    |
**● multi_compile 和 shader_feature 的区别**
两者的区别就在于是否编译了所有变体。我们通过一个例子来说明，两者具体有什么区别：
假设一个 Shader 分别用 multi_compile 和 shader_feature 定义两个不同的关键字变体。
```hlsl
#pragma shader_feature _BASEMAP   // 使用基础色贴图的变体
#pragma multi_compile _NORMALMAP  // 使用法线贴图的变体
```
再使用自定义的GUI脚本 CustomEditor 来动态启用变体，且脚本规定只有当材质使用了 BASEMAP 和 NORMALMAP 时，才会为 Shader 定义对应的关键字启用变体。脚本的相关语句如下：
```cs
CoreUtils.SetKeyword(targetMat, "_BASEMAP", targetMat.GetTexture("_BaseMap"));
CoreUtils.SetKeyword(targetMat, "_NORMALMAP", targetMat.GetTexture("_NormalMap"));
```
接下来，我们设定两个不同情景，然后查看 Shader 变体的编译（构建）情况。在 Shader 的 Compiled Code（已编译代码）选项中点击 Show（variants include） 可以查看 Shader 包含的变体。

情景一：在材质球上不使用两张贴图。我们可以在 variants include 看到以下信息：
```hlsl
Keywords stripped away when not used: ... _BASEMAP  // 不使用时去掉的关键字：_BASEMAP
Keywords always included into build: _NORMALMAP     // 始终包含在构建中的关键字：_NORMALMAP

1 keyword variants used in scene:  // 场景中使用的1个关键字变体

_NORMALMAP
```
情景二：在材质球上使用两张贴图。我们可以在 variants include 看到以下信息：
```hlsl
Keywords stripped away when not used: ..._BASEMAP  // 不使用时去掉的关键字：_BASEMAP
Keywords always included into build: _NORMALMAP    // 始终包含在构建中的关键字：_NORMALMAP

1 keyword variants used in scene: // 场景中使用的2个关键字变体

_BASEMAP _NORMALMAP
```

对比两次编译信息，我们可以得到以下结论：
1）multi_compile 定义的关键字变体，无论 CustomEditor 是否定义其关键字（材质球是否使用相应贴图），这个变体始终都会被构建（编译），也就是说它一直都处于启用的状态。
2）shader_feature 定义的关键字变体，只有在 CustomEditor 定义其关键字（材质球使用了该贴图）时，这个变体才会被构建（编译）和启用。

根据上述结论，我们可以来解释以下这两个问题：
Q1：为什么在说明 multi_compile 和 shader_feature 的区别时，最常见的叙述是 multi_compile 和 shader_feature 的区别在于是否打包所有变体？
A：因为打包会将所有已编译（构建）的 Shader 变体全部打进包中，即上述情景中 keyword variants used in scene 中所列举的那些变体，无论是否真的在场景中被使用，它们都会被打包。
从variants include 的信息中我们可以看到， multi_compile 定义的关键字被始终包含在构建（编译）的列表中，无论是否使用，变体都会编译，也就都会被打包；而 shader_feature 定义的关键字在没有被使用时会被去掉，只有在被使用时对应变体才会被编译，出现在 keyword variants used in scene 中，仅有被实际启用的变体才会被打包。
这也可以看出 multi_compile 相比于 shader_feature 的缺点，multi_compile 会导致很多没有被使用的 Shader 变体被打进包体，可能导致打包速度慢。

Q2：如何选择使用 multi_compile 还是 shader_feature 来定义变体？
A：如果一个变体是在打包后的程序动态启用的，且这个变体在打包前从未被使用（编译）过，那么使用 multi_compile 更合适。如果使用 shader_feature，会因为 shader_feature 不会打包没有编译过的 Shader 变体，导致打包后程序通过关键字找不到对应的 Shader 变体。

**● multi_compile_local 和 shader_feature_local的区别**
根据官方文档的说明，shader_feature 和 multi_compile 的主要缺点是，它们中定义的所有关键字都会影响 Unity 的全局关键字数量限制（256 个全局关键字，加上 64 个本地关键字）。为了避免变体过多占用全局关键字变量的数量，官方建议使用shader_feature_local 和 multi_compile_local 指令创建着色器变体。

shader_feature_local 和 multi_compile_local 与 multi_compile 和 shader_feature 相似，但是它们枚举的关键字是本地的。Local（局部）指令将特定于该着色器的已定义关键字保存在着色器下面，而不是将它们应用于整个项目。全局关键字的状态不会影响到本地关键字的状态。
### （4）关键字集合
如果要在同一条 multi_compile 和 shader_feature 指令中定义多个关键字，请在指令后加上一个下划线，如下所示：
```hlsl
#pragma shader_feature _ _BASEMAP _NORMALMAP    
```
如果不加这个下划线，第一个关键字是默认定义的，在不使用任何贴图的情况下，查看 Shader 的 variants include 可以看到以下信息：
```
Keywords stripped away when not used:  _BASEMAP _NORMALMAP // 不使用时去掉的关键字：_BASEMAP _NORMALMAP

1 keyword variants used in scene:  // 场景中使用的1个关键字变体：_BASEMAP

_BASEMAP
```
此时我们并没有为材质球设置BaseMap，但这个变体依然编译了。
### （5）特定阶段关键字
根据官方文档，默认情况下，Unity会为着色器的每个阶段生成关键字变量。例如，如果着色器包含顶点阶段和片段阶段，Unity会为顶点和片段着色器程序的每个关键字组合生成变体。如果一组关键字只在其中一个阶段使用，将会导致另一个阶段出现相同的变体。Unity会自动识别相同的变体并对其进行重复数据删除，因此它们不会增加构建大小，但仍会导致编译时间浪费、着色器加载时间增加以及运行时内存使用增加。

为了避免这一问题，可以在着色器中使用声明关键字时，可以指定在哪一个着色器阶段来编译它们。

关于特定阶段的关键字，Unity的官方文档说明很模糊，在 Unity Shader 中常见的有`multi_compile_vertex`和`shader_feature_local_fragment`。
### （6）Shader 快捷指令
Unity 内置了一些快捷指令，可以快速创建关键字和变量集。常见的快捷指令有：

| 快捷指令                             | Unity 添加的关键字                                                                                                                                                                                 | 作用                           |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| multi_compile_fog                | `FOG_LINEAR`, `FOG_EXP`, `FOG_EXP2`                                                                                                                                                          | Unity内置雾效。                   |
| multi_compile_fwdbase            | `DIRECTIONAL`, `LIGHTMAP_ON`, `DIRLIGHTMAP_COMBINED`,<br>`DYNAMICLIGHTMAP_ON`, `SHADOWS_SCREEN`, <br>`SHADOWS_SHADOWMASK`, `LIGHTMAP_SHADOW_MIXING`, <br>`LIGHTPROBE_SH`                     | （Build-In）前向渲染。              |
| multi_compile_fwdadd             | `POINT`, `DIRECTIONAL`, `SPOT`, `POINT_COOKIE`<br> `DIRECTIONAL_COOKIE`                                                                                                                      | （Build-In）前向渲染的附加光。          |
| multi_compile_fwdadd_fullshadows | `POINT`, `DIRECTIONAL`, `SPOT`, `POINT_COOKIE`,<br>`DIRECTIONAL_COOKIE`, `SHADOWS_DEPTH SHADOWS_SCREEN`<br> `SHADOWS_CUBE SHADOWS_SOFT SHADOWS_SHADOWMASK`, <br>`LIGHTMAP_SHADOW_MIXING`<br> | （Build-In）为前向渲染的附加光添加实时阴影功能。 |
| multi_compile_shadowcaster       | `SHADOWS_DEPTH`, `SHADOWS_CUBE`                                                                                                                                                              | （Build-In）阴影投射和深度纹理着色器通道。    |
| multi_compile_instancing         | `INSTANCING_ON`. 如果项目使用程序实例化，还会添加`PROCEDURAL_ON`                                                                                                                                             | 启用实例化。                       |
### （7）编译平台
Unity在打包时，会把 Shader 编译成不同平台对应的代码，Shader 可以使用`#pragma only_renderers`和`pragma exclude_renderers`指定目标平台和剔除目标平台
- `#pragma only_renderers`指定编译的目标平台。
- `#pragma exclude_renderers`剔除目标平台，不编译该平台代码。

● Unity 编译平台：
```
d3d11    - Direct3D 11/12
glcore   - OpenGL 3.x/4.x
gles     - OpenGL ES 2.0
gles3    - OpenGL ES 3.x
metal    - iOS/Mac Metal
vulkan   - Vulkan
d3d11_9x - Direct3D 11 9.x功能级别，通常在WSA平台上使用
xboxone  - Xbox One
ps4      - PlayStation 4
psp2     - PlayStation Vita
n3ds     - Nintendo 3DS
wiiu     - Nintendo Wii U
```

● Unity在不同目标平台所定义的宏：

| 宏                   | 目标平台                                      |
| ------------------- | ----------------------------------------- |
| SHADER_API_D3D11    | Direct3D 11                               |
| SHADER_API_GLCORE   | 桌面端 OpenGL“核心”(GL 3/4)                    |
| SHADER_API_GLES     | OpenGL ES 2.0                             |
| SHADER_API_GLES3    | OpenGL ES 3.0/3.1                         |
| SHADER_API_METAL    | iOS/Mac Metal                             |
| SHADER_API_VULKAN   | Vulkan                                    |
| SHADER_API_D3D11_9X | 适用于通用 Windows 平台的 Direct3D 11“功能级别 9.x”目标 |
| SHADER_API_MOBILE   | 所有常规移动平台（GLES、GLES3、METAL）                |

# 二、Properties 属性
Lit 着色器设置的相关属性：
```HLSL
Properties
{
	// 设置工作流模式 反射流或金属流
	// Specular vs Metallic workflow
	_WorkflowMode("WorkflowMode", Float) = 1.0

	// 基础色贴图和颜色叠加
	[MainTexture] _BaseMap("Albedo", 2D) = "white" {}
	[MainColor] _BaseColor("Color", Color) = (1,1,1,1)

	// Alpha裁剪值
	_Cutoff("Alpha Cutoff", Range(0.0, 1.0)) = 0.5

	// 光滑度和光滑度贴图设置
	_Smoothness("Smoothness", Range(0.0, 1.0)) = 0.5
	_SmoothnessTextureChannel("Smoothness texture channel", Float) = 0

	// 金属度和金属-光泽度贴图
	_Metallic("Metallic", Range(0.0, 1.0)) = 0.0
	_MetallicGlossMap("Metallic", 2D) = "white" {}

	// 反射颜色和反射-光泽度贴图
	_SpecColor("Specular", Color) = (0.2, 0.2, 0.2)
	_SpecGlossMap("Specular", 2D) = "white" {}

	[ToggleOff] _SpecularHighlights("Specular Highlights", Float) = 1.0
	[ToggleOff] _EnvironmentReflections("Environment Reflections", Float) = 1.0

	// 法线贴图和法线强度
	_BumpScale("Scale", Float) = 1.0
	_BumpMap("Normal Map", 2D) = "bump" {}

	// 视差贴图和视差强度
	_Parallax("Scale", Range(0.005, 0.08)) = 0.005
	_ParallaxMap("Height Map", 2D) = "black" {}

	// 遮蔽贴图和强度
	_OcclusionStrength("Strength", Range(0.0, 1.0)) = 1.0
	_OcclusionMap("Occlusion", 2D) = "white" {}

	// 自发光颜色和自发光贴图
	[HDR] _EmissionColor("Color", Color) = (0,0,0)
	_EmissionMap("Emission", 2D) = "white" {}

	// 细节纹理
	// 细节纹理遮罩
	_DetailMask("Detail Mask", 2D) = "white" {}
	// 细节纹理基础色强度和基础色贴图
	_DetailAlbedoMapScale("Scale", Range(0.0, 2.0)) = 1.0
	_DetailAlbedoMap("Detail Albedo x2", 2D) = "linearGrey" {}
	// 细节纹理法线强度和法线贴图
	_DetailNormalMapScale("Scale", Range(0.0, 2.0)) = 1.0
	[Normal] _DetailNormalMap("Normal Map", 2D) = "bump" {}

	// 透明涂层
	// Lit Shader中没有透明涂层渲染
	// SRP batching compatibility for Clear Coat (Not used in Lit)
	[HideInInspector] _ClearCoatMask("_ClearCoatMask", Float) = 0.0
	[HideInInspector] _ClearCoatSmoothness("_ClearCoatSmoothness", Float) = 0.0

	// Blending state 混合状态
	_Surface("__surface", Float) = 0.0  // 透明模式
	_Blend("__blend", Float) = 0.0      // 混合模式
	_Cull("__cull", Float) = 2.0        // 剔除模式  
	
	[ToggleUI] _AlphaClip("__clip", Float) = 0.0       // Alpha裁剪
	[HideInInspector] _SrcBlend("__src", Float) = 1.0  // 混合源颜色
	[HideInInspector] _DstBlend("__dst", Float) = 0.0  // 混合目标颜色
	[HideInInspector] _ZWrite("__zw", Float) = 1.0     // 深度写入

	
	[ToggleUI] _ReceiveShadows("Receive Shadows", Float) = 1.0  // 接收阴影
	// Editmode props
	_QueueOffset("Queue offset", Float) = 0.0  // 调整队列优先级

	// ObsoleteProperties（过时的属性）
	[HideInInspector] _MainTex("BaseMap", 2D) = "white" {}
	[HideInInspector] _Color("Base Color", Color) = (1, 1, 1, 1)
	[HideInInspector] _GlossMapScale("Smoothness", Float) = 0.0
	[HideInInspector] _Glossiness("Smoothness", Float) = 0.0
	[HideInInspector] _GlossyReflections("EnvironmentReflections", Float) = 0.0

	[HideInInspector][NoScaleOffset]unity_Lightmaps("unity_Lightmaps", 2DArray) = "" {}
	[HideInInspector][NoScaleOffset]unity_LightmapsInd("unity_LightmapsInd", 2DArray) = "" {}
	[HideInInspector][NoScaleOffset]unity_ShadowMasks("unity_ShadowMasks", 2DArray) = "" {}
}
```
## 1、\_Surface 透明模式

`_Surface`表示物体的透明类型，Opaque 不透明或 Transparent 半透明，对应 Inspector 面板上的  Surface Type 选项（BaseShaderGUI 的 alphaClipProp 变量）。BaseShaderGUI 会根据不同透明模式设置队列和`_ZWrite`。
> 详细待更新......

## 2、\_Blend 混合模式
设置GPU将片元着色器的输出与渲染目标进行合并的模式，着色器的颜色混合方式由`_SrcBlend`和`_DstBlend`属性的值决定。当着色器设置为 Transparent 半透明时，Inspector 面板上才会开放 Blending Mode 选项。
```hlsl
Blend[_SrcBlend][_DstBlend]
```

`_SrcBlend`和`_DstBlend`分别表示源颜色（该片元着色器的颜色值）和目标颜色（颜色缓冲区中的颜色值），BaseShaderGUI 中设置四种不同的混合模式：
```cs
BlendMode.Alpha       : Blend SrcAlpha OneMinusSrcAlpha // 传统透明度
BlendMode.Premultiply : Blend One      OneMinusSrcAlpha // 预乘透明度
BlendMode.Additive    : Blend SrcAlpha One              // ???
BlendMode.Multiply    : Blend DstColor Zero             // 乘法    
```

**● 常用的混合类型 (Blend Type)**
```HLSL
Blend SrcAlpha OneMinusSrcAlpha // 传统透明度 
Blend One OneMinusSrcAlpha // 预乘透明度 
Blend One One // 加法 
Blend OneMinusDstColor One // 软加法 
Blend DstColor Zero // 乘法 
Blend DstColor SrcColor // 2x 乘法
```
## 3、\_Cull 剔除模式
`_Cull`表示物体的剔除模式，它有0.0、1.0、2.0三个值，分别表示Off（不剔除）、Front（剔除正面）和Back（剔除背面）三种模式。

Q：Lit 是如何通过检查器（Inspector）面板上的 Render Face 选项设置剔除模式的？
A：
我们知道在 Lit 着色器中`_Cull`属性的 Float 值决定色器的剔除模式，如下：
```hlsl
Cull[_Cull]
```
由于使用了 BaseShaderGUI 编辑 Lit 着色器的GUI，Lit 着色器剔除模式的设置会涉及到以下四个属性/参数：

| 属性/参数       | 定义位置                 | 作用                                                               |
| ----------- | -------------------- | ---------------------------------------------------------------- |
| Render Face | Lit Inspector        | cullingProp 参数在 Lit 着色器检查器面板上的 UI 控件，Render Face 的选项表示的是需要渲染哪个面。 |
| cullingProp | BaseShaderGUI        | cullingProp 在检查器面板上获取的值将传给着色器的 \_Cull 属性。                        |
| \_Cull      | Lit Properties       | 0.0、1.0、2.0分别表示Off、Front、Back。                                   |
| Cull        | Lit Pass RenderSetup | 剔除模式属性                                                           |

要注意，Lit Inspector 界面上的 Render Face 选项表示的是用户希望着色器渲染的面，而不是希望剔除的面，以上剔除设置的四个属性/参数的值的关联如下：

| Render Face（Lit Inspector） | cullingProp（BaseShaderGUI） | \_Cull（Lit Properties） | Cull（Lit Pass RenderSetup） | 剔除面      |
| :------------------------: | :------------------------: | :--------------------: | :------------------------: | -------- |
|            Both            |            0.0             |          0.0           |            Off             | Off 不剔除  |
|            Back            |            1.0             |          1.0           |           Front            | Front 正面 |
|           Front            |            2.0             |          2.0           |            Back            | Back 背面  |
假设要剔除对象的背面（Back），我们需要在 Lit Inspector 的 Render Face 下拉栏中选择“Front”，此时 cullingProp 的值为 2.0， cullingProp 再将值传递给\_Cull，最终由\_Cull 将 Cull 属性设置为 Back。
## 4、\_ZWrite 深度写入
`_ZWrite` 表示是否启用深度写入。Lit 着色器的`_ZWrite` 无需手动更改，BaseShaderGUI 会根据不同透明模式设置`_ZWrite`，Opaque 模式下开启深度写入`_ZWrite`为 on， Transparent 模式下关闭深度写入`_ZWrite`为 off。

## 5、\_Alpha 裁剪

`_AlphaClip` 表示是否启用 Alpha 裁剪，对应 Shader Inspector 面板上的 Alpha Clipping （BaseShaderGUI 中`alphaClipProp`变量）选项。

当在 Inspector 面板上勾选 Alpha Clipping 时，`_AlphaClip`值为1，BaseShaderGUI 会为着色器定义关键字`_ALPHATEST_ON` ，以启用 Alpha 裁剪，BaseShaderGUI 代码如下：
```cs
bool alphaClip = false;
// 如果材质的AlphaClip属性存在，则获取AlphaClip的值进行判断。如果AlphaClip大于0.5，为材质球设置关键字_ALPHATEST_ON
if (material.HasProperty(Property.AlphaClip))  
	alphaClip = material.GetFloat(Property.AlphaClip) >= 0.5;
CoreUtils.SetKeyword(material, ShaderKeywordStrings._ALPHATEST_ON, alphaClip);
```
## 6、Cutoff 裁剪值

`_Cutoff`表示裁剪值，对应 Shader Inspector 面板上的 Threshold （BaseShaderGUI 中`alphaCutoffProp`变量）选项。着色器会在裁剪阶段调用该变参。

只有在 Inspector 面板上勾选 Alpha Clipping 时，Threshold（\_Cutoff） 才会出现在面板上，BaseShaderGUI 代码如下：
```cs
if ((alphaClipProp != null) && (alphaCutoffProp != null) && (alphaClipProp.floatValue == 1))
	materialEditor.ShaderProperty(alphaCutoffProp, Styles.alphaClipThresholdText, 1);
```

## 7、ReceiveShadows 接收阴影
> 详细待更新......
## 8、QueueOffset 调整队列优先级
`_QueueOffset`对队列值进行加减以调整优先级，对应 Shader Inspector 面板上的 Sorting Priority 值（BaseShaderGUI 中`queueOffsetProp`变量）。
以透明队列 Transparent 为例，Transparent 的队列为3000，`_QueueOffset`为50，则当前物体队列为3050。
# 三、SubShader 子着色器
Lit Shader 中有两个子着色器，分别使用了4.5和2.0两种不同的 ShaderModel（优化渲染引擎模式）。以下是 4.5 子着色器的代码，共有7个 Pass，比 2.0 多了一个 GBuffer Pass。后续 Pass 渲染通道的分析以 ShaderModel 4.5 子着色器为样本。

```hlsl
SubShader
{
	Tags{"RenderType" = "Opaque" "RenderPipeline" = "UniversalPipeline" "UniversalMaterialType" = "Lit" "IgnoreProjector" = "True" "ShaderModel"="4.5"}
	LOD 300
	
	Pass
	{
		Name "ForwardLit"
		Tags{"LightMode" = "UniversalForward"}
		
		Blend[_SrcBlend][_DstBlend]
		ZWrite[_ZWrite]
		Cull[_Cull]

		HLSLPROGRAM
		......
		ENDHLSL
	}
	
	Pass
	{
		Name "ShadowCaster"
		Tags{"LightMode" = "ShadowCaster"}
	
		ZWrite On
		ZTest LEqual
		ColorMask 0
		Cull[_Cull]
	
		HLSLPROGRAM
		......
		ENDHLSL
	}
	
	Pass
	{
		Name "GBuffer"
		Tags{"LightMode" = "UniversalGBuffer"}
	
		ZWrite[_ZWrite]
		ZTest LEqual
		Cull[_Cull]
	
		HLSLPROGRAM
		......
		ENDHLSL
	}
	Pass
	{
		Name "DepthOnly"
		Tags{"LightMode" = "DepthOnly"}
	
		ZWrite On
		ColorMask 0
		Cull[_Cull]
	
		HLSLPROGRAM
		......
		ENDHLSL
	}
	
	Pass
	{
		Name "DepthNormals"
		Tags{"LightMode" = "DepthNormals"}
	
		ZWrite On
		Cull[_Cull]
	
		HLSLPROGRAM
		......
		ENDHLSL
	}
	
	Pass
	{
		Name "Meta"
		Tags{"LightMode" = "Meta"}
	
		Cull Off
	
		HLSLPROGRAM
		......
		ENDHLSL
	}
	
	Pass
	{
		Name "Universal2D"
		Tags{ "LightMode" = "Universal2D" }
	
		Blend[_SrcBlend][_DstBlend]
		ZWrite[_ZWrite]
		Cull[_Cull]
	
		HLSLPROGRAM
		......
		ENDHLSL
	}
}
FallBack "Hidden/Universal Render Pipeline/FallbackError"
CustomEditor "UnityEditor.Rendering.Universal.ShaderGUI.LitShader"
```
**Tags 标签**
```hlsl
"RenderType" = "Opaque"                 // 渲染类型 : 渲染不透明
"RenderPipeline" = "UniversalPipeline"  // 渲染管线 : URP管线
"IgnoreProjector" = "True"              // 受投影器影响
"ShaderModel" = "4.5"                   // 优化渲染引擎模式 4.5
```
# 四、ForwardLit Pass 前向渲染通道
ForwardLit Pass 前向渲染通道。
## 1、Tags 标签 和 RenderSetup 渲染状态设置
```HLSL
Pass
{
	// Pass名
	Name "ForwardLit"
	
	// Tags 标签
	Tags{"LightMode" = "UniversalForward"}
	
	// RenderSetup 渲染状态设置       
	Blend[_SrcBlend][_DstBlend]
	ZWrite[_ZWrite]
	Cull[_Cull]

	......
}
```

## 2、#pragma 预编译
Lit 的预编译主要有五个部分，编译平台、优化渲染引擎模式、材质球关键字、URP管线关键字、Unity宏定义关键字。
```HLSL
Pass
{
	......
	
	// 编译平台和优化渲染引擎模式
	#pragma exclude_renderers gles gles3 glcore
	#pragma target 4.5
	
	// -------------------------------------
	// Material Keywords —— 材质球关键字
	#pragma shader_feature_local _NORMALMAP
	#pragma shader_feature_local _PARALLAXMAP
	#pragma shader_feature_local _RECEIVE_SHADOWS_OFF 
	#pragma shader_feature_local _ _DETAIL_MULX2 _DETAIL_SCALED
	#pragma shader_feature_local_fragment _SURFACE_TYPE_TRANSPARENT
	#pragma shader_feature_local_fragment _ALPHATEST_ON
	#pragma shader_feature_local_fragment _ALPHAPREMULTIPLY_ON
	#pragma shader_feature_local_fragment _EMISSION
	#pragma shader_feature_local_fragment _METALLICSPECGLOSSMAP
	#pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A
	#pragma shader_feature_local_fragment _OCCLUSIONMAP
	#pragma shader_feature_local_fragment _SPECULARHIGHLIGHTS_OFF
	#pragma shader_feature_local_fragment _ENVIRONMENTREFLECTIONS_OFF
	#pragma shader_feature_local_fragment _SPECULAR_SETUP
	
	// -------------------------------------
	// Universal Pipeline keywords —— URP管线关键字
	#pragma multi_compile _ _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _MAIN_LIGHT_SHADOWS_SCREEN
	#pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
	#pragma multi_compile_fragment _ _ADDITIONAL_LIGHT_SHADOWS
	#pragma multi_compile_fragment _ _REFLECTION_PROBE_BLENDING
	#pragma multi_compile_fragment _ _REFLECTION_PROBE_BOX_PROJECTION
	#pragma multi_compile_fragment _ _SHADOWS_SOFT
	#pragma multi_compile_fragment _ _SCREEN_SPACE_OCCLUSION
	#pragma multi_compile_fragment _ _DBUFFER_MRT1 _DBUFFER_MRT2 _DBUFFER_MRT3
	#pragma multi_compile_fragment _ _LIGHT_LAYERS
	#pragma multi_compile_fragment _ _LIGHT_COOKIES
	#pragma multi_compile _ _CLUSTERED_RENDERING
	
	// -------------------------------------
	// Unity defined keywords —— Unity宏定义关键字
	#pragma multi_compile _ LIGHTMAP_SHADOW_MIXING
	#pragma multi_compile _ SHADOWS_SHADOWMASK
	#pragma multi_compile _ DIRLIGHTMAP_COMBINED
	#pragma multi_compile _ LIGHTMAP_ON
	#pragma multi_compile _ DYNAMICLIGHTMAP_ON
	#pragma multi_compile_fog
	#pragma multi_compile_fragment _ DEBUG_DISPLAY
	
	......
}
```
### 2. 1、编译平台关键字
```hlsl
#pragma exclude_renderers gles gles3 glcore
```
ForwardLit Pass 默认剔除了 OpenGL ES 2.0、OpenGL ES 3.x、OpenGL 3.x/4.x 三个 OpenGL 平台，即不会在这些平台渲染。编译平台指令详请参考 [[Shader 知识点]]。
### 2. 2、材质球关键字
均使用 shader_feature_local 和 shader_feature_local_fragment 指令定义变体。
```HLSL
_NORMALMAP                           // 启用法线贴图
_PARALLAXMAP                         // 启用视差贴图
_RECEIVE_SHADOWS_OFF                 // 关闭接收阴影 
_ _DETAIL_MULX2 _DETAIL_SCALED       // 启用细节纹理的两个相关关键字
_SURFACE_TYPE_TRANSPARENT            // 物体表面类型，Opaque或Transparent
_ALPHATEST_ON                        // 启用Alpha剔除
_ALPHAPREMULTIPLY_ON                 // 启用Alpha的multiply计算（光照乘alpha值）
_EMISSION                            // 启用自发光
_METALLICSPECGLOSSMAP                // 启用光泽度贴图（金属-光泽度贴图/反射-光泽度贴图）
_SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A // 从BaseMap贴图的Alpha通道读取光滑度贴图
_OCCLUSIONMAP                        // 启用遮蔽贴图
_SPECULARHIGHLIGHTS_OFF              // 启用镜面高光 
_ENVIRONMENTREFLECTIONS_OFF          // 关闭环境反射
_SPECULAR_SETUP                      // 设置高光的工作流模式为反射流
```
#### （1）\_RECEIVE_SHADOWS_OFF
表示是否物体接受阴影，默认OFF不接受阴影。如果物体开启接受阴影功能，`_RECEIVE_SHADOWS_OFF`关键字不会被着色器定义，如下：
```hlsl
#if !defined(_RECEIVE_SHADOWS_OFF)
	
	// 主灯光阴影
    #if defined(_MAIN_LIGHT_SHADOWS) || defined(_MAIN_LIGHT_SHADOWS_CASCADE) || defined(_MAIN_LIGHT_SHADOWS_SCREEN)
        #define MAIN_LIGHT_CALCULATE_SHADOWS

        #if defined(_MAIN_LIGHT_SHADOWS) || (defined(_MAIN_LIGHT_SHADOWS_SCREEN) && !defined(_SURFACE_TYPE_TRANSPARENT))
            #define REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR
        #endif
    #endif
	
	// 附加灯光阴影
    #if defined(_ADDITIONAL_LIGHT_SHADOWS)
        #define ADDITIONAL_LIGHT_CALCULATE_SHADOWS
    #endif
#endif
```

**● 主灯光**
当开启接收阴影时，如果着色器主灯光使用了常规阴影、级联阴影、屏幕空间阴影中的一种，则定义计算阴影坐标的关键字
`MAIN_LIGHT_CALCULATE_SHADOWS`在 InputData 初始化时计算主灯光阴影坐标（[[#（2）shadowCoord 阴影纹理坐标|shadowCoord 阴影纹理坐标]]）。

如果主灯光使用常规阴影或屏幕空间阴影，且物体表面不是半透明时，着色器定义关键字`REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR`直接申请一个阴影纹理坐标插值器，不再单独计算阴影坐标。阴影纹理坐标插值器见 [[#4、shadowCoord |shadowCoord]]。

**● 附加灯光**
待测试......
#### （2）细节纹理
只有当着色器使用`_DETAIL_MULX2`或`_DETAIL_SCALED`关键字时，计算漫射细节的关键字`_DETAIL`才会被定义，如下：
```hlsl
#if defined(_DETAIL_MULX2) || defined(_DETAIL_SCALED)
#define _DETAIL
#endif
```
#### （3）_ADDITIONAL_LIGHTS_VERTEX

表示着色器需要计算附加灯光的顶点光照。

附加灯光计算由 UniversalRenderPipelineAsset 控制的，在Add>Additional Lights中选择Per Vertex即可启用`_ADDITIONAL_LIGHTS_VERTEX`，计算顶点光照；选择Per Vertex，计算片元光照。

#### （4）\_SURFACE_TYPE_TRANSPARENT
表示物体的表面透明类型，Opaque不透明或Transparent半透明。`_SURFACE_TYPE_TRANSPARENT`与 BaseShaderGUI 的属性挂钩，可以通过 Inspector 面板上 Surface Type 选择透明类型。

#### （5）\_ALPHATEST_ON
表示开启 Alpha 剔除，在 Inspector 面板上勾选 Alpha Clipping 选项时被定义。`_ALPHATEST_ON`由 Properties 的 `_AlphaClip`属性定义，详情参考[[#（5） _Alpha 裁剪 |\_Alpha 裁剪]]。
#### （6）\_SPECULAR_SETUP
如果定义`_SPECULAR_SETUP`关键字，着色器使用反射流高光，未定义则使用金属流高光。
```
#if _SPECULAR_SETUP // Specular反射流高光 
	outSurfaceData.metallic = half(1.0); 
	outSurfaceData.specular = specGloss.rgb; 
#else // Metallic金属流高光
```
### 2. 3、URP管线的关键字
```HLSL
//----- 主灯光阴影 -----
_MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _MAIN_LIGHT_SHADOWS_SCREEN // 主灯光阴影类型

//---- 附加灯光 ----
_ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS // 附加灯光顶点光照 附加灯光光照
_ADDITIONAL_LIGHT_SHADOWS                    // 附加灯光阴影

//---- 反射探针 ----
_REFLECTION_PROBE_BLENDING                   // 反射探针混合
_REFLECTION_PROBE_BOX_PROJECTION             // 反射探针盒体投影

_SHADOWS_SOFT                                // 软阴影
_SCREEN_SPACE_OCCLUSION                      // 场景空间遮蔽
_DBUFFER_MRT1 _DBUFFER_MRT2 _DBUFFER_MRT3    // DBUFFER 与贴花相关
_LIGHT_LAYERS                                // 灯光层级
_LIGHT_COOKIES                               // 灯光Cookie纹理
_CLUSTERED_RENDERING                         // 集群渲染
```
#### （1）主灯光阴影
控制主灯光阴影类型的三个关键字：
```hlsl
_MAIN_LIGHT_SHADOWS             // 启用主灯光阴影
_MAIN_LIGHT_SHADOWS_CASCADE     // 启用主灯光级联阴影
_MAIN_LIGHT_SHADOWS_SCREEN      // 启用主灯光屏幕空间阴影
```
#### （2）集群渲染
\_CLUSTERED_RENDERING 关键字用于启用集群渲染。启用时，关闭额外光的顶点光照，开启额外光光照和集群光照；关闭时，不使用集群关照。
```hlsl
#if defined(_CLUSTERED_RENDERING)
#define _ADDITIONAL_LIGHTS 1
#undef _ADDITIONAL_LIGHTS_VERTEX
#define USE_CLUSTERED_LIGHTING 1
#else
#define USE_CLUSTERED_LIGHTING 0
#endif
```
#### （3）反射探针
```hlsl
_REFLECTION_PROBE_BLENDING                   // 反射探针混合
_REFLECTION_PROBE_BOX_PROJECTION             // 反射探针盒体投影
```
● \_REFLECTION_PROBE_BLENDING
在 Universal Render Pipeline Asset 中勾选 Probe Blending（混合探针）选项可以在 Shader 的预编译中激活 `_REFLECTION_PROBE_BLENDING`关键字，之后才能在 Shader 中使用该关键字。

● \_REFLECTION_PROBE_BOX_PROJECTION
在 Universal Render Pipeline Asset 中勾选 Box Projection（盒体投影）选项可以在 Shader 的预编译中激活 `_REFLECTION_PROBE_BOX_PROJECTION`关键字，之后才能在 Shader 中使用该关键字。此外还需在反射探针（Reflection Probes）中勾选 Box Projection 才能使用。
### 2. 4、Unity宏定义关键字
Unity定义的关键字大部分都与光照贴图的使用有关，此外还有阴影遮罩、雾效和Debug。
```hlsl
LIGHTMAP_SHADOW_MIXING       // 混合光照贴图阴影
SHADOWS_SHADOWMASK           // 阴影遮罩
DIRLIGHTMAP_COMBINED         // 启用具有方向的光照贴图
LIGHTMAP_ON                  // 启用光照贴图
DYNAMICLIGHTMAP_ON           // 启用动态光照贴图
//---------------------------------------------------------------------
#pragma multi_compile_fog    // 启用Unity内置雾效，使用的是Shader的快捷指令
//---------------------------------------------------------------------
DEBUG_DISPLAY                // Debug显示
```
#### （1）\_LIGHTMAP_ON
确定 ForwardLit Pass 是启用 LightMap 光照贴图还是 SH球谐光照，定义如下：
```HLSL
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
`LIGHTMAP_ON`中定义了三个关键字。
```
DECLARE_LIGHTMAP_OR_SH ：纹理坐标
OUTPUT_LIGHTMAP_UV : 光照贴图UV
OUTPUT_SH : SH球谐光
```
##### ● DECLARE_LIGHTMAP_OR_SH
```
// lmName：LightMap光照贴图名
// shName：SH球谐光照变量名
// index:  纹理的插槽的编号
#define DECLARE_LIGHTMAP_OR_SH(lmName, shName, index) half3  shName : TEXCOORD##index
#define DECLARE_LIGHTMAP_OR_SH(lmName, shName, index) float2 lmName : TEXCOORD##index
```
如果着色器使用LightMap，给 staticLightmapUV 声明一个纹理坐标：
```HLSL
float2 staticLightmapUV : TEXCOORD8
```
 如果着色器使用SH球谐光，给 vertexSH 声明一个纹理坐标
```HLSL
half3 vertexSH : TEXCOORD8
```
##### ● OUTPUT_LIGHTMAP_UV
着色器启用 LightMap 时，为 LightMap 输出一张UV：
```HLSL
#define OUTPUT_LIGHTMAP_UV(lightmapUV, lightmapScaleOffset, OUT) 
	   - OUT.xy = lightmapUV.xy * lightmapScaleOffset.xy + lightmapScaleOffset.zw
```
着色器启用 SH 时，输出为空：
```hlsl
#define OUTPUT_LIGHTMAP_UV(lightmapUV, lightmapScaleOffset, OUT)
```
##### ● OUTPUT_SH
当着色器使用 SH球谐光照时，输出 SH 顶点采样：
```HLSL
#define OUTPUT_SH(normalWS, OUT) OUT.xyz = SampleSHVertex(normalWS)
```
如果着色器使用 LightMap，输出为空：
```HLSL
#define OUTPUT_SH(normalWS, OUT)
```
`SampleSHVertex`函数定义详见 [[Shader 知识点#（2）Spherical Harmonic 球谐光照]]。

#### （2）Fog 雾效
`multi_compile_fog`是 Shader 的快捷指令，使用这个快捷指令时，Unity 会给 Shader 添加 `FOG_LINEAR`、`FOG_EXP`、`FOG_EXP2`三个关键字。快捷指令详细参考 [[Shader 知识点]]。
## 3、头文件
```hlsl
#include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
#include "Packages/com.unity.render-pipelines.universal/Shaders/LitForwardPass.hlsl"
```
# 五、ForwardLit Pass - Attributes 顶点输入结构体
Attributes结构体内是输入顶点着色器的模型顶点信息和UV数据。
```HLSL
struct Attributes
{
    float4 positionOS   : POSITION;         // 顶点的模型空间位置
    float3 normalOS     : NORMAL;           // 顶点在模型空间中的法线
    float4 tangentOS    : TANGENT;          // 顶点在模型空间中的切线
    float2 texcoord     : TEXCOORD0;        // 模型UV，存放在纹理插槽0
    float2 staticLightmapUV   : TEXCOORD1;  // 静态LightmapUV
    float2 dynamicLightmapUV  : TEXCOORD2;  // 动态LightmapUV
    UNITY_VERTEX_INPUT_INSTANCE_ID
};
```
TEXCOORD0表示模型的第一UV，如果模型有多套UV，TEXCOORD1、TEXCOORD2、TEXCOORD3可以表示第二、三、四UV。

# 六、ForwardLit Pass - Varyings 顶点输出结构体
Varyings结构体内是经过顶点着色器计算后输出的各种数据。
```HLSL
struct Varyings
{
    float2 uv                       : TEXCOORD0;

#if defined(REQUIRES_WORLD_SPACE_POS_INTERPOLATOR)
    float3 positionWS               : TEXCOORD1;
#endif

    float3 normalWS                 : TEXCOORD2;
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR)
    half4 tangentWS                : TEXCOORD3;    // xyz: tangent, w: sign
#endif
    float3 viewDirWS                : TEXCOORD4;

#ifdef _ADDITIONAL_LIGHTS_VERTEX
    half4 fogFactorAndVertexLight   : TEXCOORD5; // x: fogFactor, yzw: vertex light
#else
    half  fogFactor                 : TEXCOORD5;
#endif

#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
    float4 shadowCoord              : TEXCOORD6;
#endif

#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirTS                : TEXCOORD7;
#endif

	// 为staticLightmap或vertexSH声明纹理坐标
    DECLARE_LIGHTMAP_OR_SH(staticLightmapUV, vertexSH, 8); 
      
	// 启用动态光照贴图时，为dynamicLightmapUV声明纹理坐标
#ifdef DYNAMICLIGHTMAP_ON
    float2  dynamicLightmapUV : TEXCOORD9; // Dynamic lightmap UVs
#endif

    float4 positionCS               : SV_POSITION;
    UNITY_VERTEX_INPUT_INSTANCE_ID
    UNITY_VERTEX_OUTPUT_STEREO
};
```
## 1、positionWS
世界空间的顶点坐标。
```HLSL
#if defined(REQUIRES_WORLD_SPACE_POS_INTERPOLATOR)
    float3 positionWS               : TEXCOORD1;
#endif
```
**● REQUIRES_WORLD_SPACE_POS_INTERPOLATOR**
请求顶点世界空间坐标插值器（positionWS）。在Lit Shader中，该关键字只有两处会使用：
1）Cookie：只有在Lit使用Cookie，即使用`_LIGHT_COOKIES`时`REQUIRES_WORLD_SPACE_POS_INTERPOLATOR`才被会定义。Cookie的使用详见[[#● Cookie]]小节；
```HLSL
#if defined(_LIGHT_COOKIES)
    #ifndef REQUIRES_WORLD_SPACE_POS_INTERPOLATOR
        #define REQUIRES_WORLD_SPACE_POS_INTERPOLATOR 1
    #endif
#endif
```
2）Baked Shadow：计算烘焙阴影时需要先定义`REQUIRES_WORLD_SPACE_POS_INTERPOLATOR`，才能定义`CALCULATE_BAKED_SHADOWS`。
```HLSL
#define REQUIRES_WORLD_SPACE_POS_INTERPOLATOR
#if defined(LIGHTMAP_ON) || defined(LIGHTMAP_SHADOW_MIXING) || defined(SHADOWS_SHADOWMASK)
#define CALCULATE_BAKED_SHADOWS
#endif
```
## 2、tangentWS
世界空间的顶点切线。
```HLSL
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR)
    half4 tangentWS                : TEXCOORD3;    // xyz: tangent, w: sign
#endif
```
**● REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR**
请求顶点世界空间切线插值器（tangentWS）。在ForwardLit Pass中该关键字只有一处定义，如下：
```HLSL
#if (defined(_NORMALMAP) || (defined(_PARALLAXMAP) && !defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR))) || defined(_DETAIL)
#define REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR
#endif
```
## 3、fogFactor And VertexLight
雾效因子和附加灯光的顶点光照。
```HLSL
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    half4 fogFactorAndVertexLight   : TEXCOORD5; // x: fogFactor, yzw: vertex light
#else
    half  fogFactor                 : TEXCOORD5;
#endif
```
如果定义`_ADDITIONAL_LIGHTS_VERTEX`，则表示着色器需要计算附加灯光的顶点光照。ForwardLit Pass会将顶点光照和Fog雾效放在同一个half4变量`fogFactorAndVertexLight`中，x为雾效，yzw为顶点光照。
`_ADDITIONAL_LIGHTS_VERTEX`关键字详请参考 [[#（3）_ADDITIONAL_LIGHTS_VERTEX]]
## 4、shadowCoord
阴影纹理坐标。
```HLSL
#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
    float4 shadowCoord              : TEXCOORD6;
#endif
```
● **REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR**
请求阴影纹理坐标插值器（ShadowCoord）。
## 5、viewDirTS
切线空间的观察向量。
```HLSL
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirTS                : TEXCOORD7;
#endif
```
● **REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR**
请求切线空间观察插值器（viewDirTS）。该关键字与视差计算相关，当着色器使用视差贴图且不当前平台不是OpenGL ES2.0（桌面或移动）时，Lit会在顶点着色器中计算切线空间观察向量ViewTS。
```HLSL
#if defined(_PARALLAXMAP) && !defined(SHADER_API_GLES)
#define REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR
#endif
```
## 6、DECLARE_LIGHTMAP_OR_SH
选择启用LightMap光照贴图或SH（Spherical Harmonic）球谐光照，为其输出采样UV。
```HLSL
DECLARE_LIGHTMAP_OR_SH(staticLightmapUV, vertexSH, 8);
```
`DECLARE_LIGHTMAP_OR_SH`在不同情况下的声明：
```HLSL
// 启用LightMap
float2 staticLightmapUV : TEXCOORD8
// 启用SH
half3 vertexSH : TEXCOORD8
```
`DECLARE_LIGHTMAP_OR_SH`详细请参考 [[#● DECLARE_LIGHTMAP_OR_SH]]。
# 七、ForwardLit Pass - LitPassVertex 顶点着色器
```HLSL
Varyings LitPassVertex(Attributes input)
{
	/*-----------------------------------------------------------------------*/
	/*                          创建输出结构体output                           */
	/*-----------------------------------------------------------------------*/
    Varyings output = (Varyings)0;

    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_TRANSFER_INSTANCE_ID(input, output);
    UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);
	/*-----------------------------------------------------------------------*/
	/*                       VertexPosition 顶点位置信息                       */
	/*-----------------------------------------------------------------------*/
	// 获取顶点位置信息
    VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
	/*-----------------------------------------------------------------------*/
	/*                       VertexNormal 顶点法切线信息                       */
	/*-----------------------------------------------------------------------*/
	// 获取顶点的法线、切线和TBN矩阵。
	
    // normalWS and tangentWS already normalize.
    // this is required to avoid skewing the direction during interpolation
    // also required for per-vertex lighting and SH evaluation
    // 官方注释：NormalWS和tangentWS已经归一化。这是为了避免插值过程中的方向偏移，也需要逐顶点照明和SH评估。
    
    VertexNormalInputs normalInput = GetVertexNormalInputs(input.normalOS, input.tangentOS);
	/*-----------------------------------------------------------------------*/
	/*                      VertexLight 附加灯光顶点光照                       */
	/*-----------------------------------------------------------------------*/
	// 计算附加灯光的顶点光照（需要使用_ADDITIONAL_LIGHTS_VERTEX变体，否则vertexLight = 0不输出）
    half3 vertexLight = VertexLighting(vertexInput.positionWS, normalInput.normalWS);
	/*-----------------------------------------------------------------------*/
	/*                            FogFactor 雾效                             */
	/*-----------------------------------------------------------------------*/
	// 雾效：在顶点着色器计算（官方强制在片元着色器计算）
    half fogFactor = 0;
    #if !defined(_FOG_FRAGMENT)
        fogFactor = ComputeFogFactor(vertexInput.positionCS.z);
    #endif
	/*-----------------------------------------------------------------------*/
	/*                                  UV                                   */
	/*-----------------------------------------------------------------------*/
    output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);
	/*-----------------------------------------------------------------------*/
	/*                               normalWS                                */
	/*-----------------------------------------------------------------------*/
    // already normalized from normal transform to WS.
    // 官方注释：Normal变换到世界空间后已经归一化。
    output.normalWS = normalInput.normalWS;
	/*-----------------------------------------------------------------------*/
	/*                               tangentWS                               */
	/*-----------------------------------------------------------------------*/
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR) || defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    real sign = input.tangentOS.w * GetOddNegativeScale();
    half4 tangentWS = half4(normalInput.tangentWS.xyz, sign);
#endif
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR)
    output.tangentWS = tangentWS;
#endif
	/*-----------------------------------------------------------------------*/
	/*                               viewDirTS                               */
	/*-----------------------------------------------------------------------*/
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(vertexInput.positionWS);
    half3 viewDirTS = GetViewDirectionTangentSpace(tangentWS, output.normalWS, viewDirWS);
    output.viewDirTS = viewDirTS;
#endif
	/*-----------------------------------------------------------------------*/
	/*                              LightMap UV                              */
	/*-----------------------------------------------------------------------*/
	// 输出静态光照贴图UV
    OUTPUT_LIGHTMAP_UV(input.staticLightmapUV, unity_LightmapST, output.staticLightmapUV);
    // 输出动态光照贴图UV
#ifdef DYNAMICLIGHTMAP_ON
    output.dynamicLightmapUV = input.dynamicLightmapUV.xy * 
				    unity_DynamicLightmapST.xy + unity_DynamicLightmapST.zw;
#endif
	/*-----------------------------------------------------------------------*/
	/*                              SH 球谐光照                               */
	/*-----------------------------------------------------------------------*/
    OUTPUT_SH(output.normalWS.xyz, output.vertexSH);
    /*-----------------------------------------------------------------------*/
	/*                      选择是否输出附加灯光的顶点光照                       */
	/*-----------------------------------------------------------------------*/
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    output.fogFactorAndVertexLight = half4(fogFactor, vertexLight);
#else
    output.fogFactor = fogFactor;
#endif
	/*-----------------------------------------------------------------------*/
	/*                              positionWS                               */
	/*-----------------------------------------------------------------------*/
#if defined(REQUIRES_WORLD_SPACE_POS_INTERPOLATOR)
    output.positionWS = vertexInput.positionWS;
#endif
	/*-----------------------------------------------------------------------*/
	/*                          ShadowCoord 阴影坐标                          */
	/*-----------------------------------------------------------------------*/
#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
    output.shadowCoord = GetShadowCoord(vertexInput);
#endif
	/*-----------------------------------------------------------------------*/
	/*                              positionWS                               */
	/*-----------------------------------------------------------------------*/
    output.positionCS = vertexInput.positionCS;

    return output;
}
```
## 1、VertexPosition 顶点位置信息
```HLSL
/*-----------------------------------------------------------------------*/
/*                    VertexPosition 获取顶点位置信息                      */
/*-----------------------------------------------------------------------*/
VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
```
**（1）VertexPositionInputs 结构体**
```hlsl
struct VertexPositionInputs
{
    float3 positionWS; // World space position
    float3 positionVS; // View space position
    float4 positionCS; // Homogeneous clip space position
    float4 positionNDC;// Homogeneous normalized device coordinates
};
```

**（2）GetVertexPositionInputs 获取顶点坐标信息**
调用`GetVertexPositionInputs`获取顶点在不同空间的位置信息和屏幕空间UV。输入模型空间OS的顶点位置，计算世界空间WS、观察空间VS、裁剪空间CS的顶点坐标。
```HLSL
VertexPositionInputs GetVertexPositionInputs(float3 positionOS)
{
    VertexPositionInputs input;
    input.positionWS = TransformObjectToWorld(positionOS);       // 顶点世界空间坐标
    input.positionVS = TransformWorldToView(input.positionWS);   // 顶点观察空间坐标
    input.positionCS = TransformWorldToHClip(input.positionWS);  // 顶点裁剪空间坐标
	
	// "NDC"
    float4 ndc = input.positionCS * 0.5f;
    input.positionNDC.xy = float2(ndc.x, ndc.y * _ProjectionParams.x) + ndc.w;
    input.positionNDC.zw = input.positionCS.zw;

    return input;
}
```
**（3）PositionCS**
这里有一个很容易忽略的问题，在 positionWS -> positionCS 的过程中，`TransformWorldToHClip`对 positionCS 执行了两次变换，先进行视图（View）变换，再进行投影（Projection）变换，也就是说这个变换过程实际上是 positionWS -> positionVS -> positionCS。
`TransformWorldToHClip`函数具体定义如下：
```hlsl
float4 TransformWorldToHClip(float3 positionWS)
{
	// positionWS -> positionCS
    return mul(GetWorldToHClipMatrix(), float4(positionWS, 1.0));
}
```
`GetWorldToHClipMatrix`返回的是一个视图（View）与投影（Projection）的组合变换矩阵：
```hlsl
float4x4 GetWorldToHClipMatrix()
{
    return UNITY_MATRIX_VP;
}
```
而在投影变换过程中，positionVS 的z分量的负值会写入到 positionCS 的w分量中：
```hlsl
positionCS.w = -positionVS.z;
```
**（4）positionNDC**
positionNDC 等价于已经弃用的函数`ComputeScreenPos`，计算的是屏幕空间的UV，两者的计算公式完全相同。虽然名字里带有NDC，但它严格意义上并非表示标准化设备坐标NDC，而是“Homogeneous Normalized Device Coordinates”（近似NDC）。
ComputeScreenPos 函数具体定义如下：
```hlsl
float4 ComputeScreenPos(float4 positionCS)
{
    float4 o = positionCS * 0.5f;
    o.xy = float2(o.x, o.y * _ProjectionParams.x) + o.w;
    o.zw = positionCS.zw;
    return o;
}
```

**● \_ProjectionParams**
Unity内置的投影参数，用来处理投影矩阵的翻转。\_ProjectionParams.x 为1，如果投影式是翻转的则为-1。\_ProjectionParams 详请参考 [[Shader 知识点]]。

## 2、VertexNormal 获顶点法切线信息
```HLSL
/*-----------------------------------------------------------------------*/
/*                    VertexNormal 获取顶点法切线信息                      */
/*-----------------------------------------------------------------------*/
// 官方注释：NormalWS和tangentWS已经归一化。这是为了避免插值过程中的方向偏移，也需要逐顶点照明和SH评估。
VertexNormalInputs normalInput = GetVertexNormalInputs(input.normalOS, input.tangentOS);
```
VertexNormalInputs 结构体：
```hlsl
struct VertexNormalInputs
{
    real3 tangentWS;
    real3 bitangentWS;
    float3 normalWS;
};
```
调用`GetVertexNormalInputs`获取顶点的法线、切线和TBN矩阵。（获取的法线、切线均已归一化）
```HLSL
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
## 3、VertexLighting 附加灯光顶点光照
```HLSL
/*-----------------------------------------------------------------------*/
/*                      VertexLight 附加灯光顶点光照                       */
/*-----------------------------------------------------------------------*/
// 需要使用_ADDITIONAL_LIGHTS_VERTEX变体，否则vertexLight = 0
half3 vertexLight = VertexLighting(vertexInput.positionWS, normalInput.normalWS);
```
`VertexLighting`计算附加灯光的顶点光照。ForwardLit Pass在预编译中预设了使用附加灯光顶点光照的变体，关键字为`_ADDITIONAL_LIGHTS_VERTEX`。
```HLSL
half3 VertexLighting(float3 positionWS, half3 normalWS)
{
    half3 vertexLightColor = half3(0.0, 0.0, 0.0);

#ifdef _ADDITIONAL_LIGHTS_VERTEX
	// 获取附加灯光数
    uint lightsCount = GetAdditionalLightsCount();
    // 遍历计算每一个附加灯光光照
    LIGHT_LOOP_BEGIN(lightsCount)
	    // 获取灯光的数据
        Light light = GetAdditionalLight(lightIndex, positionWS);
        // 灯光光照颜色 = 灯光颜色 * 灯光的距离衰减
        half3 lightColor = light.color * light.distanceAttenuation;
        // 计算Lambert光照
        vertexLightColor += LightingLambert(lightColor, light.direction, normalWS);
    LIGHT_LOOP_END
#endif

    return vertexLightColor;
}
```

#### **● GetAdditionalLightsCount**
获取附加灯数量。
```HLSL
int GetAdditionalLightsCount()
{
#if USE_CLUSTERED_LIGHTING
    return 0;
#else
    return int(min(_AdditionalLightsCount.x, unity_LightData.y));
#endif
}
```
#### **● GetAdditionalLight**
获取附加灯光的数据。
```HLSL
Light GetAdditionalLight(uint i, float3 positionWS)
{
#if USE_CLUSTERED_LIGHTING
    int lightIndex = i;
#else
    int lightIndex = GetPerObjectLightIndex(i);
#endif
    return GetAdditionalPerObjectLight(lightIndex, positionWS);
}
```
**● LightingLambert**
计算Lambert光照。
```HLSL
half3 LightingLambert(half3 lightColor, half3 lightDir, half3 normal)
{
    half NdotL = saturate(dot(normal, lightDir));
    return lightColor * NdotL;
}
```
## 4、顶点着色器计算雾效
首先，着色器想要使用Unity的雾效，需要在预编译时声明`multi_compile_fog`。
```HLSL
#pragma multi_compile_fog
```
从以下的代码来看，当着色器没有定义关键字`_FOG_FRAGMENT`（雾效_片元）时，可以在顶点着色器中计算雾效。但实际上ForwardLit Pass是强制着色器在片元计算雾效的，因为 ShaderVariablesFunctions.hlsl 中定义了关键字`_FOG_FRAGMENT`，而ForwardLit Pass是一定会调用 ShaderVariablesFunctions.hlsl 的。
```HLSL
half fogFactor = 0;
#if !defined(_FOG_FRAGMENT)
	fogFactor = ComputeFogFactor(vertexInput.positionCS.z);
#endif
```
 ShaderVariablesFunctions.hlsl 中官方已经说明强制在片元着色器计算雾效，定义的关键字`_FOG_FRAGMENT`如下：
```HLSL
// Force enable fog fragment shader evaluation
// 官方注释：强制启用雾片段着色器评估
#define _FOG_FRAGMENT 1
```
如果需要在顶点着色器中计算雾效，请删除`!defined(_FOG_FRAGMENT)`。
## 5、UV
```HLSL
/*-----------------------------------------------------------------------*/
/*                                  UV                                   */
/*-----------------------------------------------------------------------*/
output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);
```
`TRANSFORM_TEX`计算了UV的Tiling（平铺）和Offset（偏移），其定义如下：
```HLSL
#define TRANSFORM_TEX(tex, name) ((tex.xy) * name##_ST.xy + name##_ST.zw)
```
使用`TRANSFORM_TEX`等价于
```HLSL
output.uv = (input.texcoord.xy) * _BaseMap_ST.xy + _BaseMap_ST.zw;
```
## 6、tangentWS
```HLSL
/*-----------------------------------------------------------------------*/
/*                               tangentWS                               */
/*-----------------------------------------------------------------------*/
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR) || defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    real sign = input.tangentOS.w * GetOddNegativeScale();
    half4 tangentWS = half4(normalInput.tangentWS.xyz, sign);
#endif
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR)
    output.tangentWS = tangentWS;
#endif
```
#### **● GetOddNegativeScale**
获取物体Odd Negative Scaling的情况。
```HLSL
real GetOddNegativeScale()
{
    return unity_WorldTransformParams.w >= 0.0 ? 1.0 : -1.0;
}
```
Q：Odd Negative Scaling是什么？
A：当一个物体transform（变换）的scale（缩放）的XYZ三个值乘积为负数时，该物体具有Odd Negative Scaling，即物体有奇数个负缩放轴。unity_WorldTransformParams.w表示的就是物体是否具有Odd Negative Scaling，不具有时为1，具有时为0。
## 7、viewDirTS
```HLSL
/*-----------------------------------------------------------------------*/
/*                               viewDirTS                               */
/*-----------------------------------------------------------------------*/
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(vertexInput.positionWS);
    half3 viewDirTS = GetViewDirectionTangentSpace(tangentWS, output.normalWS, viewDirWS);
    output.viewDirTS = viewDirTS;
#endif
```

#### ● GetWorldSpaceNormalizeViewDir
获取世界空间的观察向量，并归一化。
```HLSL
half3 GetWorldSpaceNormalizeViewDir(float3 positionWS)
{
	// 判断是否是透视相机
    if (IsPerspectiveProjection())
    {
        // Perspective 透视相机
        float3 V = GetCurrentViewPosition() - positionWS;
        return half3(normalize(V));
    }
    else
    {
        // Orthographic 正交相机
        return half3(-GetViewForwardDir());
    }
}
```
#### ● GetCurrentViewPosition
获取当前相机在世界空间中的位置。
```HLSL
float3 GetCameraPositionWS()
{
    return _WorldSpaceCameraPos; // 相机在世界空间的坐标
}
// 获取当前相机观察的位置（世界坐标）
float3 GetCurrentViewPosition()
{
    return GetCameraPositionWS();
}
```

#### ● GetViewDirectionTangentSpace
获取切线空间中的观察向量viewDirTS。`GetViewDirectionTangentSpace`先计算了TBN矩阵，然后用TBN矩阵和viewDirWS计算viewDirTS。
```HLSL
// Return view direction in tangent space, make sure tangentWS.w is already multiplied by GetOddNegativeScale()
half3 GetViewDirectionTangentSpace(half4 tangentWS, half3 normalWS, half3 viewDirWS)
{
    // must use interpolated tangent, bitangent and normal before they are normalized in the pixel shader.
    half3 unnormalizedNormalWS = normalWS; // 未归一化的世界空间法线
    
	// 计算一个归一化因子 renormFactor
	// 通过将未归一化的法线向量的长度取倒数得到归一化因子，这个因子将用于后续的归一化操作。
    const half renormFactor = 1.0 / length(unnormalizedNormalWS);

	/*-----------------------------------------------------------------------*/
	/*                                TBN 矩阵                                */
	/*-----------------------------------------------------------------------*/
    // use bitangent on the fly like in hdrp
    // IMPORTANT! If we ever support Flip on double sided materials ensure bitangent and tangent are NOT flipped.
    // 官方注释：如果我们支持双面材料上的翻转，请确保副切线和切线不会翻转。
    half crossSign = (tangentWS.w > 0.0 ? 1.0 : -1.0); // we do not need to multiple GetOddNegativeScale() here, as it is done in vertex shader
	
	// 计算世界空间副切线
    half3 bitang = crossSign * cross(normalWS.xyz, tangentWS.xyz);

	// 世界空间法线
    half3 WorldSpaceNormal = renormFactor * normalWS.xyz;       // we want a unit length Normal Vector node in shader graph

    // to preserve mikktspace compliance we use same scale renormFactor as was used on the normal.
    // This is explained in section 2.2 in "surface gradient based bump mapping framework"
    // 世界空间切线
    half3 WorldSpaceTangent = renormFactor * tangentWS.xyz;
    // 世界空间副切线
    half3 WorldSpaceBiTangent = renormFactor * bitang;
	
	// TBN矩阵
    half3x3 tangentSpaceTransform = half3x3(WorldSpaceTangent, WorldSpaceBiTangent, WorldSpaceNormal);
	/*-----------------------------------------------------------------------*/
	/*                               viewDirTS                               */
	/*-----------------------------------------------------------------------*/
    // 计算切线空间观察方向
    half3 viewDirTS = mul(tangentSpaceTransform, viewDirWS);

    return viewDirTS;
}
```
## 8、LightMap UV
```HLSL
/*-----------------------------------------------------------------------*/
/*                              LightMap UV                              */
/*-----------------------------------------------------------------------*/
// 输出静态光照贴图UV
OUTPUT_LIGHTMAP_UV(input.staticLightmapUV, unity_LightmapST, output.staticLightmapUV);
// 输出动态光照贴图UV
#ifdef DYNAMICLIGHTMAP_ON
    output.dynamicLightmapUV = input.dynamicLightmapUV.xy * 
				    unity_DynamicLightmapST.xy + unity_DynamicLightmapST.zw;
#endif
```
**1）静态光照贴图UV**
`OUTPUT_LIGHTMAP_UV`输出静态光照贴图的UV`output.staticLightmapUV`，如下：
```HLSL
output.staticLightmapUV.xy = input.staticLightmapUV.xy * unity_LightmapST.xy + unity_LightmapST.zw
```
`OUTPUT_LIGHTMAP_UV`的定义详细请参考 [[#● OUTPUT_LIGHTMAP_UV]]。
**2）动态光照贴图UV**
如果定义`DYNAMICLIGHTMAP_ON`输出动态光照贴图UV`output.dynamicLightmapUV`。
```HLSL
output.dynamicLightmapUV = input.dynamicLightmapUV.xy * unity_DynamicLightmapST.xy + unity_DynamicLightmapST.zw;
```
## 9、SH 球谐光照
输出SH球谐光照，详细请参考 [[#● OUTPUT_SH]]。
```HLSL
/*-----------------------------------------------------------------------*/
/*                              SH 球谐光照                               */
/*-----------------------------------------------------------------------*/
OUTPUT_SH(output.normalWS.xyz, output.vertexSH);
```
## 10、输出附加灯光的顶点光照
根据管线关键字`_ADDITIONAL_LIGHTS_VERTEX`选择是否输出附加灯光的顶点光照。`_ADDITIONAL_LIGHTS_VERTEX`关键字详请参考 [[#（3）_ADDITIONAL_LIGHTS_VERTEX]]
```HLSL
/*-----------------------------------------------------------------------*/
/*                      选择是否输出附加灯光的顶点光照                       */
/*-----------------------------------------------------------------------*/
// output.fogFactorAndVertexLight.x   存放雾效因子
// output.fogFactorAndVertexLight.yzw 存放附加灯光的顶点光照
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    output.fogFactorAndVertexLight = half4(fogFactor, vertexLight);
#else
    output.fogFactor = fogFactor;
#endif
```
# 八、ForwardLit Pass - LitPassFragment 片元着色器
```HLSL
half4 LitPassFragment(Varyings input) : SV_Target
{
    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input); // XR项目使用

	/*-----------------------------------------------------------------------*/
	/*                             PARALLAX 视差                              */
	/*-----------------------------------------------------------------------*/
	// 启用视差图计算视差
#if defined(_PARALLAXMAP)
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirTS = input.viewDirTS;
#else
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);
    half3 viewDirTS = GetViewDirectionTangentSpace(input.tangentWS, input.normalWS, viewDirWS);
#endif
    ApplyPerPixelDisplacement(viewDirTS, input.uv);
#endif
	/*-----------------------------------------------------------------------*/
	/*                         SurfaceData 数据初始化                         */
	/*-----------------------------------------------------------------------*/
	// 初始化物体表面数据
    SurfaceData surfaceData;
    InitializeStandardLitSurfaceData(input.uv, surfaceData);
	/*-----------------------------------------------------------------------*/
	/*                          InputData 数据初始化                          */
	/*-----------------------------------------------------------------------*/
	// 初始化计算物理光照需要的相关数据
    InputData inputData;
    InitializeInputData(input, surfaceData.normalTS, inputData);
    
    // 设置DEBUG的贴图数据
    SETUP_DEBUG_TEXTURE_DATA(inputData, input.uv, _BaseMap);

	// URP的贴花系统
#ifdef _DBUFFER
    ApplyDecalToSurfaceData(input.positionCS, surfaceData, inputData);
#endif

	// 计算PBR光照
    half4 color = UniversalFragmentPBR(inputData, surfaceData);

	// 混合雾效
    color.rgb = MixFog(color.rgb, inputData.fogCoord);
    // 输出Alpha通道信息
    color.a = OutputAlpha(color.a, _Surface);

    return color;
}
```
## 1、PARALLAX 视差
Pass在预编译中预设了使用视差计算的变体，关键字为`_PARALLAXMAP`。
```HLSL
#if defined(_PARALLAXMAP)
// viewDirTS
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirTS = input.viewDirTS;
#else
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);
    half3 viewDirTS = GetViewDirectionTangentSpace(input.tangentWS, input.normalWS, viewDirWS);
#endif
	// 计算视差
    ApplyPerPixelDisplacement(viewDirTS, input.uv);
#endif
```
计算视差需要使用切线空间的观察向量 viewDirTS，若着色器定义了关键字`REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR`，viewDirTS 会在顶点着色器中计算。
## 2、SurfaceData 物体表面数据
```HLSL
SurfaceData surfaceData;
InitializeStandardLitSurfaceData(input.uv, surfaceData);
```
### 2. 1、SurfaceData 结构体
SurfaceData 结构体用于存储物体表面的数据，主要是物体表面的物理属性，如颜色、模型法线、粗糙度等。这些属性可以直接使用纹理贴图（Texture）输入着色器，而无需通过计算得到。
```hlsl
struct SurfaceData
{
    half3 albedo;               // 基础色
    half3 specular;             // 高光色
    half  metallic;             // 金属度
    half  smoothness;           // 光泽度
    half3 normalTS;             // 法线（切线空间）
    half3 emission;             // 自发光
    half  occlusion;            // 遮挡值
    half  alpha;                // Alpha值
    half  clearCoatMask;        // 透明图层遮罩
    half  clearCoatSmoothness;  // 透明图层光滑度
};
```
### 2. 2、SurfaceData 初始化
`InitializeStandardLitSurfaceData`计算物体表面的物理属性数据，并由 SurfaceData 输出。
```HLSL
inline void InitializeStandardLitSurfaceData(float2 uv, out SurfaceData outSurfaceData)
{
	/*-----------------------------------------------------------------------*/
	/*                            BaseMap贴图采样                             */
	/*-----------------------------------------------------------------------*/
	// 采样BaseMap基础色贴图
    half4 albedoAlpha = SampleAlbedoAlpha(uv, TEXTURE2D_ARGS(_BaseMap, sampler_BaseMap));
	/*-----------------------------------------------------------------------*/
	/*                          Alpha计算和Clip剔除                           */
	/*-----------------------------------------------------------------------*/
    // 计算alpha值，并根据alpha进行Clip剔除
    outSurfaceData.alpha = Alpha(albedoAlpha.a, _BaseColor, _Cutoff);
	/*-----------------------------------------------------------------------*/
	/*                     SPECULAR_SETUP 高光工作流模式                       */
	/*-----------------------------------------------------------------------*/
    half4 specGloss = SampleMetallicSpecGloss(uv, albedoAlpha.a);
    outSurfaceData.albedo = albedoAlpha.rgb * _BaseColor.rgb;

	// 设置高光的工作流模式
#if _SPECULAR_SETUP
	// Specular反射流高光
    outSurfaceData.metallic = half(1.0);
    outSurfaceData.specular = specGloss.rgb;
#else
	// Metallic金属流高光
    outSurfaceData.metallic = specGloss.r;
    outSurfaceData.specular = half3(0.0, 0.0, 0.0);
#endif
	// smoothness光滑度
    outSurfaceData.smoothness = specGloss.a;
	/*-----------------------------------------------------------------------*/
	/*                             Normal贴图采样                             */
	/*-----------------------------------------------------------------------*/
    outSurfaceData.normalTS = SampleNormal(uv, TEXTURE2D_ARGS(_BumpMap, sampler_BumpMap), _BumpScale);
	/*-----------------------------------------------------------------------*/
	/*                             Occlusion采样                              */
	/*-----------------------------------------------------------------------*/
    outSurfaceData.occlusion = SampleOcclusion(uv);
	/*-----------------------------------------------------------------------*/
	/*                            Emission贴图采样                            */
	/*-----------------------------------------------------------------------*/
    outSurfaceData.emission = SampleEmission(uv, _EmissionColor.rgb, TEXTURE2D_ARGS(_EmissionMap, sampler_EmissionMap));
	/*-----------------------------------------------------------------------*/
	/*                   CLEARCOAT 透明涂层（Lit没有该项计算）                  */
	/*-----------------------------------------------------------------------*/
#if defined(_CLEARCOAT) || defined(_CLEARCOATMAP)
    half2 clearCoat = SampleClearCoat(uv);
    outSurfaceData.clearCoatMask       = clearCoat.r;
    outSurfaceData.clearCoatSmoothness = clearCoat.g;
#else
    outSurfaceData.clearCoatMask       = half(0.0);
    outSurfaceData.clearCoatSmoothness = half(0.0);
#endif
	/*-----------------------------------------------------------------------*/
	/*                            DETAIL 细节纹理                             */
	/*-----------------------------------------------------------------------*/
#if defined(_DETAIL)
    half detailMask = SAMPLE_TEXTURE2D(_DetailMask, sampler_DetailMask, uv).a;
    float2 detailUv = uv * _DetailAlbedoMap_ST.xy + _DetailAlbedoMap_ST.zw;
    outSurfaceData.albedo = ApplyDetailAlbedo(detailUv, outSurfaceData.albedo, detailMask);
    outSurfaceData.normalTS = ApplyDetailNormal(detailUv, outSurfaceData.normalTS, detailMask);
#endif
}
```
#### （1）BaseMap 贴图采样
`_BaseMap` 是 Unity 定义的基础色贴图，调用`SampleAlbedoAlpha`采样基础色贴图 BaseMap，`TEXTURE2D_ARGS`关键字定义了需要传入`SampleAlbedoAlpha`函数的贴图名和采样器名。
```HLSL
/*-----------------------------------------------------------------------*/
/*                            BaseMap贴图采样                             */
/*-----------------------------------------------------------------------*/
half4 albedoAlpha = SampleAlbedoAlpha(uv, TEXTURE2D_ARGS(_BaseMap, sampler_BaseMap));
```

>**BaseMap 基础色贴图**
>在Lit Shader中，BaseMap的RGB通道为基础色，Alpha 通道根据Shader是否定义`_SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A`有两种情况：
>（1）如果定义该关键字，则 BaseMap.a = Smoothness（光滑度）；
>（2）如果未定义该关键字，则 BaseMap.a =  Alpha（剔除值）。
>
>【KEYWORDS】
>_\_SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A_：Albedo 贴图 A 通道为 Smoothness 贴图。

**● SampleAlbedoAlpha**
```HLSL
half4 SampleAlbedoAlpha(float2 uv, TEXTURE2D_PARAM(albedoAlphaMap, sampler_albedoAlphaMap))
{
    return half4(SAMPLE_TEXTURE2D(albedoAlphaMap, sampler_albedoAlphaMap, uv));
}
```
函数作用：采样基础色贴图 BaseMap。
传入参数：UV 、基础色贴图`albedoAlphaMap`及其采样器`sampler_albedoAlphaMap`。
返回值：albedoAlphaMap 采样值。 

**● TEXTURE2D_PARAM 和 TEXTURE2D_ARGS 关键字**
`TEXTURE2D_PARAM`定义了一张 TEXTURE2D 贴图和该贴图的 SAMPLER 采样器。
```HLSL
#define TEXTURE2D_PARAM(textureName, samplerName)             TEXTURE2D(textureName),  SAMPLER(samplerName)
```
`TEXTURE2D_ARGS`定义了一个 Texture 贴图名和贴图相应的采样器名。
```HLSL
#define TEXTURE2D_ARGS(textureName, samplerName)     textureName, samplerName
```
`TEXTURE2D_PARAM`和`TEXTURE2D_ARGS`定义在D3D11、GLCore、GLES2、GLES3等多个HLSL文件中。
```HLSL
#include "com.unity.render-pipelines.core@12.1.10\ShaderLibrary\API\D3D11.hlsl"
#include "com.unity.render-pipelines.core@12.1.10\ShaderLibrary\API\GLCore.hlsl"
#include "com.unity.render-pipelines.core@12.1.10\ShaderLibrary\API\GLES2.hlsl"
#include "com.unity.render-pipelines.core@12.1.10\ShaderLibrary\API\GLES3.hlsl"
#include "com.unity.render-pipelines.core@12.1.10\ShaderLibrary\API\Metal.hlsl"
#include "com.unity.render-pipelines.core@12.1.10\ShaderLibrary\API\Switch.hlsl"
#include "com.unity.render-pipelines.core@12.1.10\ShaderLibrary\API\EntityLighting.hlsl"
```
#### （2）Alpha
混合 BaseMap 贴图的 Alpha 通道和 BaseColor 的 Alpha 通道，然后使用这个混合的alpha值进行 Clip 剔除操作。
```HLSL
/*-----------------------------------------------------------------------*/
/*                          Alpha计算和Clip剔除                           */
/*-----------------------------------------------------------------------*/
outSurfaceData.alpha = Alpha(albedoAlpha.a, _BaseColor, _Cutoff);
```

```hlsl
half Alpha(half albedoAlpha, half4 color, half cutoff)
{
/* 
Lit Shader未定义 _GLOSSINESS_FROM_BASE_ALPHA 关键字。
如果未定义_SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A关键字，则albedoAlpha表示剔除值alpha。
*/
#if !defined(_SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A) && !defined(_GLOSSINESS_FROM_BASE_ALPHA)
    half alpha = albedoAlpha * color.a;
#else
    half alpha = color.a;
#endif

// Alpha剔除
#if defined(_ALPHATEST_ON)
    clip(alpha - cutoff);
#endif

    return alpha;
}
```
**● Alpha 值**
如果 Shader 未定义`_SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A`和`_GLOSSINESS_FROM_BASE_ALPHA`关键字，albedoAlpha 的A通道是 Alpha 值。
```hlsl
half alpha = albedoAlpha.a * _BaseColor.a;
```
如果 Shader 定义了`_SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A`和`_GLOSSINESS_FROM_BASE_ALPHA`关键字，albedoAlpha 的A通道是 Smoothness 值，不加入计算。
```hlsl
half alpha = _BaseColor.a;
```
**● Alpha 剔除**
```hlsl
clip(alpha - cutoff);
```
#### （3）高光的工作流模式
URP Lit Shader提供两种高光模式，Specular反射流和Metallic金属流，Shader 的 Inspector 面板的 WorkflowMode 可以选择工作流模式。
```HLSL
	/*-----------------------------------------------------------------------*/
	/*                     SPECULAR_SETUP 高光工作流模式                       */
	/*-----------------------------------------------------------------------*/
	// 计算高光颜色
    half4 specGloss = SampleMetallicSpecGloss(uv, albedoAlpha.a);
    // 输出反照色albedo
    outSurfaceData.albedo = albedoAlpha.rgb * _BaseColor.rgb;

	// 设置高光的工作流模式
#if _SPECULAR_SETUP
	// Specular反射流高光
    outSurfaceData.metallic = half(1.0);
    outSurfaceData.specular = specGloss.rgb;
#else
	// Metallic金属流高光
    outSurfaceData.metallic = specGloss.r;
    outSurfaceData.specular = half3(0.0, 0.0, 0.0);
#endif
	// smoothness光滑度
    outSurfaceData.smoothness = specGloss.a;
```
**● SampleMetallicSpecGloss**
```HLSL
half4 specGloss = SampleMetallicSpecGloss(uv, albedoAlpha.a);
```
根据光泽度贴图和工作流模式计算specGloss（高光-光泽度）。specGloss.rgb 存储高光颜色，specGloss.a 存储光滑度。

> **【光泽度贴图】**
> Lit Shader 中定义的光泽度贴图是 MetallicGlossMap（金属-光泽度贴图）和 SpecularGlossMap（反射-光泽度贴图），这两种贴图的RGB通道分别存储的是金属度Metallic和反射度Specular，但A通道都存储的是光滑度Smoothness。

`SampleMetallicSpecGloss`函数定义如下：
```hlsl
half4 SampleMetallicSpecGloss(float2 uv, half albedoAlpha)
{
	//高光颜色-光泽度
    half4 specGloss;

// _METALLICSPECGLOSSMAP 关键字决定是否启用光泽度贴图
#ifdef _METALLICSPECGLOSSMAP
	// 启用光泽度贴图
    specGloss = half4(SAMPLE_METALLICSPECULAR(uv)); // 采样光泽度贴图
    
    // 计算光滑度Smoothness
    #ifdef _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A
        specGloss.a = albedoAlpha * _Smoothness;
    #else
        specGloss.a *= _Smoothness;
    #endif
#else 
	// 不启用光泽度贴图，根据高光的工作流模式计算高光颜色和光滑度
	
	// 计算高光颜色
    #if _SPECULAR_SETUP
	    // 反射流
        specGloss.rgb = _SpecColor.rgb;
    #else
	    // 金属流
        specGloss.rgb = _Metallic.rrr;
    #endif

	// 计算光滑度Smoothness
    #ifdef _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A
        specGloss.a = albedoAlpha * _Smoothness;
    #else
        specGloss.a = _Smoothness;
    #endif
#endif

    return specGloss;
}
```

**● _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A 关键字**

Pass在预编译中预设了使用Albedo的Alpha通道作为光滑度的变体，关键字为`_SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A`。
如果要使用该变体，需要在Inspector面板的Smoothness Source项选择Albedo Alpha选项。
```HLSL
#pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A
```

**● _METALLICSPECGLOSSMAP 关键字**

从语义上看，`_METALLICSPECGLOSSMAP`可解释为MetallicGlossMap（金属-光泽度贴图）和SpecularGlossMap（反射-光泽度贴图）。定义该关键字时，着色器将使用这两种贴图中的一种，Lit Shader在此将这两种贴图都称为光泽度贴图。

**● SAMPLE_METALLICSPECULAR 关键字**

`SAMPLE_METALLICSPECULAR`定义了一个2D纹理采样函数，当`SPECULAR_SETUP`工作流模式为反射流时，采样反射度贴图；当`SPECULAR_SETUP`工作流模式为金属流时，采样金属度贴图。

`SAMPLE_METALLICSPECULAR`定义如下：
```HLSL
#ifdef SPECULAR_SETUP
	// 反射流模式，采样反射度贴图SpecGlossMap
    #define SAMPLE_METALLICSPECULAR(uv) SAMPLE_TEXTURE2D(_SpecGlossMap, sampler_SpecGlossMap, uv)
#else
	// 金属流模式，采样金属度贴图MetallicGlossMap
    #define SAMPLE_METALLICSPECULAR(uv) SAMPLE_TEXTURE2D(_MetallicGlossMap, sampler_MetallicGlossMap, uv)
#endif
```
#### （4）Normal 贴图采样
```HLSL
/*-----------------------------------------------------------------------*/
/*                             Normal贴图采样                             */
/*-----------------------------------------------------------------------*/
outSurfaceData.normalTS = SampleNormal(uv, TEXTURE2D_ARGS(_BumpMap, sampler_BumpMap), _BumpScale);
```
SampleNormal函数定义如下：
```hlsl
half3 SampleNormal(float2 uv, TEXTURE2D_PARAM(bumpMap, sampler_bumpMap), half scale = half(1.0))
{
#ifdef _NORMALMAP
    half4 n = SAMPLE_TEXTURE2D(bumpMap, sampler_bumpMap, uv);
    // 
    #if BUMP_SCALE_NOT_SUPPORTED
        return UnpackNormal(n);
    #else
        return UnpackNormalScale(n, scale);
    #endif
#else
    return half3(0.0h, 0.0h, 1.0h);
#endif
}
```
#### （5）Occlusion 采样
```HLSL
/*-----------------------------------------------------------------------*/
/*                             Occlusion采样                              */
/*-----------------------------------------------------------------------*/
outSurfaceData.occlusion = SampleOcclusion(uv);
```
SampleOcclusion函数定义如下：
```HLSL
half SampleOcclusion(float2 uv)
{
    #ifdef _OCCLUSIONMAP
        // TODO: Controls things like these by exposing SHADER_QUALITY levels (low, medium, high)
	    
        #if defined(SHADER_API_GLES)
            return SAMPLE_TEXTURE2D(_OcclusionMap, sampler_OcclusionMap, uv).g;
        #else
            half occ = SAMPLE_TEXTURE2D(_OcclusionMap, sampler_OcclusionMap, uv).g;
            // 平台使用的不是OpenGL ES 2.0时，occ不得低于(1-_OcclusionStrength)
            return LerpWhiteTo(occ, _OcclusionStrength);
        #endif
    #else
        return half(1.0);
    #endif
}
```

**● _OCCLUSIONMAP 关键字**

Lit Shader预编译定义了使用遮蔽贴图的变体 _OCCLUSIONMAP，当Shader在Inspector面板使用OcclusionMap（遮蔽贴图）时启用该变体。
```HLSL
#pragma shader_feature_local_fragment _OCCLUSIONMAP
```

**● LerpWhiteTo 函数**

使采样得到的Occlusion的值不低于`(1-_OcclusionStrength)`（_OcclusionStrength遮挡强度）。
```HLSL
real LerpWhiteTo(real b, real t)
{
    real oneMinusT = 1.0 - t;
    return oneMinusT + b * t;
}
```
#### （6）Emission 贴图采样
```HLSL
/*-----------------------------------------------------------------------*/
/*                            Emission贴图采样                            */
/*-----------------------------------------------------------------------*/
outSurfaceData.emission = SampleEmission(uv, _EmissionColor.rgb, TEXTURE2D_ARGS(_EmissionMap, sampler_EmissionMap));
```

```HLSL
half3 SampleEmission(float2 uv, half3 emissionColor, TEXTURE2D_PARAM(emissionMap, sampler_emissionMap))
{
#ifndef _EMISSION
    return 0;
#else
    return SAMPLE_TEXTURE2D(emissionMap, sampler_emissionMap, uv).rgb * emissionColor;
#endif
}
```

**● _EMISSION 关键字**

Lit Shader预编译定义了使用自发光贴图的变体 _EMISSION，EmissionMap（自发光贴图）时启用该变体。
```HLSL
#pragma shader_feature_local_fragment _EMISSION
```
#### （7）CLEARCOAT 透明涂层
Lit Shader中没有启用透明涂层渲染，只有ComplexLit Shader才有。
```HLSL
/*-----------------------------------------------------------------------*/
/*                           CLEARCOAT 透明涂层                           */
/*-----------------------------------------------------------------------*/
#if defined(_CLEARCOAT) || defined(_CLEARCOATMAP)
    half2 clearCoat = SampleClearCoat(uv);
    outSurfaceData.clearCoatMask       = clearCoat.r;
    outSurfaceData.clearCoatSmoothness = clearCoat.g;
#else
    outSurfaceData.clearCoatMask       = half(0.0);
    outSurfaceData.clearCoatSmoothness = half(0.0);
#endif
```

#### （8）DETAIL 细节纹理
```HLSL
/*-----------------------------------------------------------------------*/ 
/*                            DETAIL 细节纹理                             */ 
/*-----------------------------------------------------------------------*/
#if defined(_DETAIL)
	// 采样细节纹理遮罩
    half detailMask = SAMPLE_TEXTURE2D(_DetailMask, sampler_DetailMask, uv).a;
    // 计算细节纹理漫反射贴图的UV
    float2 detailUv = uv * _DetailAlbedoMap_ST.xy + _DetailAlbedoMap_ST.zw;
    // 计算细节纹理的漫反射
    outSurfaceData.albedo = ApplyDetailAlbedo(detailUv, outSurfaceData.albedo, detailMask);
    // 计算细节纹理的法线（切线空间）
    outSurfaceData.normalTS = ApplyDetailNormal(detailUv, outSurfaceData.normalTS, detailMask);
#endif
```
**● ApplyDetailAlbedo 函数**
```HLSL
half3 ApplyDetailAlbedo(float2 detailUv, half3 albedo, half detailMask)
{
#if defined(_DETAIL)
	// 采样细节纹理的漫反射贴图
    half3 detailAlbedo = SAMPLE_TEXTURE2D(_DetailAlbedoMap, sampler_DetailAlbedoMap, detailUv).rgb;

    // In order to have same performance as builtin, we do scaling only if scale is not 1.0 (Scaled version has 6 additional instructions)
    // 官方注释：为了具有与内置相同的性能，我们仅在scale不是1.0时才进行缩放（缩放版本有6条附加指令）
#if defined(_DETAIL_SCALED)
    detailAlbedo = ScaleDetailAlbedo(detailAlbedo, _DetailAlbedoMapScale);
#else
    detailAlbedo = half(2.0) * detailAlbedo;
#endif

    return albedo * LerpWhiteTo(detailAlbedo, detailMask);
#else
    return albedo;
#endif
}
```
**● ApplyDetailNormal 函数**
```HLSL
half3 ApplyDetailNormal(float2 detailUv, half3 normalTS, half detailMask)
{
#if defined(_DETAIL)
#if BUMP_SCALE_NOT_SUPPORTED
    half3 detailNormalTS = UnpackNormal(SAMPLE_TEXTURE2D(_DetailNormalMap, sampler_DetailNormalMap, detailUv));
#else
    half3 detailNormalTS = UnpackNormalScale(SAMPLE_TEXTURE2D(_DetailNormalMap, sampler_DetailNormalMap, detailUv), _DetailNormalMapScale);
#endif

    // With UNITY_NO_DXT5nm unpacked vector is not normalized for BlendNormalRNM
    // For visual consistancy we going to do in all cases
    detailNormalTS = normalize(detailNormalTS);

    return lerp(normalTS, BlendNormalRNM(normalTS, detailNormalTS), detailMask); // todo: detailMask should lerp the angle of the quaternion rotation, not the normals
#else
    return normalTS;
#endif
}
```
## 3、InputData 空间数据
```HLSL
InputData inputData;
InitializeInputData(input, surfaceData.normalTS, inputData);
```
### 2. 1、InputData 结构体
InputData 结构体定义的成员变量主要是顶点着色器输出变量。
```HLSL
struct InputData
{
    float3  positionWS;                 // 顶点的世界空间位置
    float4  positionCS;                 // 顶点的裁剪空间位置
    float3  normalWS;                   // 世界空间中的法线
    half3   viewDirectionWS;            // 世界空间中的观察方向  
    float4  shadowCoord;                // 阴影纹理坐标，采样阴影纹理的uv坐标
    half    fogCoord;                   // 雾效坐标
    half3   vertexLighting;             // 顶点光照
    half3   bakedGI;                    // 全局光照烘焙
    float2  normalizedScreenSpaceUV;    // 归一化屏幕空间UV
    half4   shadowMask;                 // 阴影遮罩
    half3x3 tangentToWorld;             // TBN矩阵，切线空间转世界空间的变换矩阵

	// DEBUG展示使用的属性
    #if defined(DEBUG_DISPLAY)
    half2   dynamicLightmapUV;   // 动态Lightmap UV
    half2   staticLightmapUV;    // 静态Lightmap UV
    float3  vertexSH;            // 球谐光照

    half3 brdfDiffuse;           // BRDF漫反射
    half3 brdfSpecular;          // BRDF镜面反射（高光）
    float2 uv;
    uint mipCount;               // mip级别

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
### 2. 2、InputData 初始化
`InitializeInputData` 对从顶点着色器输入的数据进行再处理。
```HLSL
void InitializeInputData(Varyings input, half3 normalTS, out InputData inputData)
{
    inputData = (InputData)0;
    /*-----------------------------------------------------------------------*/
	/*                              positionWS                               */
	/*-----------------------------------------------------------------------*/
#if defined(REQUIRES_WORLD_SPACE_POS_INTERPOLATOR)
    inputData.positionWS = input.positionWS;
#endif
    /*-----------------------------------------------------------------------*/
	/*                      viewDirWS、normalWS、TBN矩阵                      */
	/*-----------------------------------------------------------------------*/
	// 获取世界空间观察向量 viewDirWS（已归一化）
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);
	// 获取世界空间法线 normalWS
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

	// 输出normalWS、viewDirectionWS（均已归一化）
    inputData.normalWS = NormalizeNormalPerPixel(inputData.normalWS);
    inputData.viewDirectionWS = viewDirWS;  
    /*-----------------------------------------------------------------------*/
	/*                        shadowCoord 阴影纹理坐标                        */
	/*-----------------------------------------------------------------------*/
#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
    inputData.shadowCoord = input.shadowCoord;
#elif defined(MAIN_LIGHT_CALCULATE_SHADOWS)
    inputData.shadowCoord = TransformWorldToShadowCoord(inputData.positionWS);
#else
    inputData.shadowCoord = float4(0, 0, 0, 0);
#endif
    /*-----------------------------------------------------------------------*/
	/*              fogCoord 雾效坐标 和 vertexLighting 顶点光照               */
	/*-----------------------------------------------------------------------*/
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    inputData.fogCoord = InitializeInputDataFog(float4(input.positionWS, 1.0), input.fogFactorAndVertexLight.x);
    inputData.vertexLighting = input.fogFactorAndVertexLight.yzw;
#else
    inputData.fogCoord = InitializeInputDataFog(float4(input.positionWS, 1.0), input.fogFactor);
#endif
    /*-----------------------------------------------------------------------*/
	/*                      Global IIIumination 全局光照                      */
	/*-----------------------------------------------------------------------*/
#if defined(DYNAMICLIGHTMAP_ON)
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.dynamicLightmapUV, input.vertexSH, inputData.normalWS);
#else
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.vertexSH, inputData.normalWS);
#endif
    /*-----------------------------------------------------------------------*/
	/*                        ScreenSpaceUV 屏幕空间UV                        */
	/*-----------------------------------------------------------------------*/
    inputData.normalizedScreenSpaceUV = GetNormalizedScreenSpaceUV(input.positionCS);
	/*-----------------------------------------------------------------------*/
	/*                          shadowMask 阴影遮罩                           */
	/*-----------------------------------------------------------------------*/
    inputData.shadowMask = SAMPLE_SHADOWMASK(input.staticLightmapUV);
    /*-----------------------------------------------------------------------*/
	/*                             DEBUG_DISPLAY                             */
	/*-----------------------------------------------------------------------*/
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
### （1）viewDirWS、normalWS
```hlsl
	/*-----------------------------------------------------------------------*/
	/*                      viewDirWS、normalWS、TBN矩阵                      */
	/*-----------------------------------------------------------------------*/
	// 获取世界空间观察向量 viewDirWS（已归一化）
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);

	// 计算normalWS，如果使用法线贴图或细节纹理，需要使用TBN矩阵将采样的法线从切线空间转换到世界空间
#if defined(_NORMALMAP) || defined(_DETAIL)
	// TBN矩阵
    float sgn = input.tangentWS.w;      // should be either +1 or -1
    float3 bitangent = sgn * cross(input.normalWS.xyz, input.tangentWS.xyz);
    half3x3 tangentToWorld = half3x3(input.tangentWS.xyz, bitangent.xyz, input.normalWS.xyz);

	// 将TBN矩阵存入tangentToWorld
    #if defined(_NORMALMAP)
    inputData.tangentToWorld = tangentToWorld;
    #endif
    
    // 使用TBN将normalTS转换为normalWS
    inputData.normalWS = TransformTangentToWorld(normalTS, tangentToWorld);
#else
    inputData.normalWS = input.normalWS;
#endif

	// 输出normalWS、viewDirectionWS（均已归一化）
    inputData.normalWS = NormalizeNormalPerPixel(inputData.normalWS);
    inputData.viewDirectionWS = viewDirWS;  
```
GetWorldSpaceNormalizeViewDir 详情参考 [[#● GetWorldSpaceNormalizeViewDir]]。
TBN矩阵详情参考 [[#● GetWorldSpaceNormalizeViewDir]]。
TransformTangentToWorld 详情参考 [[Shader 知识点#TransformTangentToWorld]]
### （2）shadowCoord 阴影纹理坐标
```hlsl
    /*-----------------------------------------------------------------------*/
	/*                        shadowCoord 阴影纹理坐标                        */
	/*-----------------------------------------------------------------------*/	
#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
	// 使用阴影纹理坐标插值器
    inputData.shadowCoord = input.shadowCoord;
#elif defined(MAIN_LIGHT_CALCULATE_SHADOWS)
	// 计算阴影纹理坐标
    inputData.shadowCoord = TransformWorldToShadowCoord(inputData.positionWS);
#else
    inputData.shadowCoord = float4(0, 0, 0, 0);
#endif
```
关键字`REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR`和`MAIN_LIGHT_CALCULATE_SHADOWS`详情参见 [[#（1）\_RECEIVE_SHADOWS_OFF]]。

**● TransformWorldToShadowCoord**
输入世界空间顶点坐标计算阴影坐标 。
```hlsl
float4 TransformWorldToShadowCoord(float3 positionWS)
{
// 如果使用级联阴影，计算级联的index
#ifdef _MAIN_LIGHT_SHADOWS_CASCADE
    half cascadeIndex = ComputeCascadeIndex(positionWS);
#else
    half cascadeIndex = half(0.0);
#endif

    float4 shadowCoord = mul(_MainLightWorldToShadow[cascadeIndex], float4(positionWS, 1.0));

    return float4(shadowCoord.xyz, 0);
}
```
关键字`_MAIN_LIGHT_SHADOWS_CASCADE`详请参考 [[#（1）主灯光阴影]]。
### （3）雾效坐标 和 顶点光照
```hlsl
    /*-----------------------------------------------------------------------*/
	/*              fogCoord 雾效坐标 和 vertexLighting 顶点光照               */
	/*-----------------------------------------------------------------------*/
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    inputData.fogCoord = InitializeInputDataFog(float4(input.positionWS, 1.0), input.fogFactorAndVertexLight.x);
    inputData.vertexLighting = input.fogFactorAndVertexLight.yzw;
#else
    inputData.fogCoord = InitializeInputDataFog(float4(input.positionWS, 1.0), input.fogFactor);
#endif
```

InitializeInputDataFog
输入世界空间的顶点坐标和顶点着色器输出的雾效因子，计算 Fog 雾效因子。
```hlsl
real InitializeInputDataFog(float4 positionWS, real vertFogFactor)
{
    real fogFactor = 0.0;
#if defined(_FOG_FRAGMENT)
    #if (defined(FOG_LINEAR) || defined(FOG_EXP) || defined(FOG_EXP2))
        // Compiler eliminates unused math --> matrix.column_z * vec
        float viewZ = -(mul(UNITY_MATRIX_V, positionWS).z);
        // View Z is 0 at camera pos, remap 0 to near plane.
        // 物体与近裁面的距离
        float nearToFarZ = max(viewZ - _ProjectionParams.y, 0);
        fogFactor = ComputeFogFactorZ0ToFar(nearToFarZ);
    #endif
#else
    fogFactor = vertFogFactor;
#endif
    return fogFactor;
}
```
此处虽然有关键字 `_FOG_FRAGMENT` 判断计算雾效因子的位置，但实际上 ForwardLit Pass 是强制在片元着色器计算雾效的，具体原因请见 [[#4、顶点着色器计算雾效]]。

`_ProjectionParams.y` 是近裁平面在view空间（相机空间）的z值，数值上等于相机设置中的近裁平面的值；`_ProjectionParams` 宏定义详请参考 [[Shader 知识点]]。
### （4）Bake GI 烘焙全局光照
Unity 的烘焙全局光照（Bake Global IIIumination）其实就是全局光照（间接光照）的漫反射，要么是光照贴图，要么是球谐光。这里着色器是判断是否使用了动态光照贴图（DynamicLightmap）。

`SAMPLE_GI`宏定义了采样光照贴图、SH球谐光的函数。
```hlsl
    /*-----------------------------------------------------------------------*/
	/*                          Bake GI 烘焙全局光照                          */
	/*-----------------------------------------------------------------------*/
#if defined(DYNAMICLIGHTMAP_ON)
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.dynamicLightmapUV, input.vertexSH, inputData.normalWS);
#else
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.vertexSH, inputData.normalWS);
#endif
```
● SAMPLE_GI
`SAMPLE_GI`宏定义了四种采样方式，使用光照贴图、动态光照贴图、SH球谐光照中的一种，或同时使用光照贴图和动态光照贴图，如下：
```hlsl
// We either sample GI from baked lightmap or from probes.
// If lightmap: sampleData.xy = lightmapUV
// If probe: sampleData.xyz = L2 SH terms

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

## 4、UniversalFragmentPBR
```hlsl
half4 UniversalFragmentPBR(InputData inputData, SurfaceData surfaceData)
{
	/*-----------------------------------------------------------------------*/
	/*                              镜面高光开光                               */
	/*-----------------------------------------------------------------------*/
	// 材质面板：Advanced options>Specular Highlights 开关镜面高光
    #if defined(_SPECULARHIGHLIGHTS_OFF)
    bool specularHighlightsOff = true;
    #else
    bool specularHighlightsOff = false;
    #endif
	/*-----------------------------------------------------------------------*/
	/*                             BRDF数据初始化                             */
	/*-----------------------------------------------------------------------*/
	// 初始化BRDF数据
    BRDFData brdfData;
    InitializeBRDFData(surfaceData, brdfData);
	/*-----------------------------------------------------------------------*/
	/*                                 Debug                                 */
	/*-----------------------------------------------------------------------*/
	// 开启Debug功能
    #if defined(DEBUG_DISPLAY)
    half4 debugColor;
	// 允许调试时覆盖输出的颜色
    if (CanDebugOverrideOutputColor(inputData, surfaceData, brdfData, debugColor))
    {
        return debugColor;
    }
    #endif
	/*-----------------------------------------------------------------------*/
	/*                           渲染相关的数据准备                            */
	/*-----------------------------------------------------------------------*/
    // Clear-coat清漆渲染数据准备（无效）
    BRDFData brdfDataClearCoat = CreateClearCoatBRDFData(surfaceData, brdfData);
    // shadowMask阴影遮罩
    half4 shadowMask = CalculateShadowMask(inputData);
    // AO系数
    AmbientOcclusionFactor aoFactor = CreateAmbientOcclusionFactor(inputData,
									    surfaceData);
    // 获取网格渲染层级
    uint meshRenderingLayers = GetMeshRenderingLightLayer();
	/*-----------------------------------------------------------------------*/
	/*                           灯光、光照相关数据                            */
	/*-----------------------------------------------------------------------*/
    // 获取主灯光数据
    Light mainLight = GetMainLight(inputData, shadowMask, aoFactor);

    // 混合实时光照和烘焙GI
    MixRealtimeAndBakedGI(mainLight, inputData.normalWS, inputData.bakedGI);
	// 创建一个光照数据结构用来存储一些光照颜色类的数据
    LightingData lightingData = CreateLightingData(inputData, surfaceData);
	/*-----------------------------------------------------------------------*/
	/*                                全局光照                                */
	/*-----------------------------------------------------------------------*/
    lightingData.giColor = GlobalIllumination(brdfData,brdfDataClearCoat,
				    surfaceData.clearCoatMask,inputData.bakedGI,
				    aoFactor.indirectAmbientOcclusion,inputData.positionWS,
				    inputData.normalWS, inputData.viewDirectionWS);	    
	/*-----------------------------------------------------------------------*/
	/*                               主灯光光照                               */
	/*-----------------------------------------------------------------------*/
	// 判断主灯光和网格渲染是否在同一层级，两者在同一层级时才计算主灯光的物理光照
    if (IsMatchingLightLayer(mainLight.layerMask, meshRenderingLayers))
    {
	    // 计算主灯光物理光照
        lightingData.mainLightColor = LightingPhysicallyBased(brdfData,
			        brdfDataClearCoat,mainLight,inputData.normalWS, 
			        inputData.viewDirectionWS,surfaceData.clearCoatMask, 
			        specularHighlightsOff);
    }
	/*-----------------------------------------------------------------------*/
	/*                              附加灯光光照                              */
	/*-----------------------------------------------------------------------*/
	// 启用附加灯光 Per Pixel（逐像素计算）
    #if defined(_ADDITIONAL_LIGHTS)
    // 获取附加灯光数
    uint pixelLightCount = GetAdditionalLightsCount();
	// 使用集群照明
    #if USE_CLUSTERED_LIGHTING
    /*
	    _AdditionalLightsDirectionalCount：额外定向光数（不确定）
	    MAX_VISIBLE_LIGHTS：最大可见光数
    */
    for (uint lightIndex = 0; lightIndex < min(
		    _AdditionalLightsDirectionalCount,MAX_VISIBLE_LIGHTS
	    );lightIndex++)
    {
	    // 获取额外光源数据
        Light light = GetAdditionalLight(lightIndex, inputData, shadowMask,
									        aoFactor);
		// 判断主灯光和网格渲染是否在同一层级，两者在同一层级时才计算主灯光的物理光照
        if (IsMatchingLightLayer(light.layerMask, meshRenderingLayers))
        {
	        // 计算附加灯光物理光照，并相叠加
            lightingData.additionalLightsColor += LightingPhysicallyBased(
				            brdfData, brdfDataClearCoat, light,
                            inputData.normalWS, inputData.viewDirectionWS,
                            surfaceData.clearCoatMask, specularHighlightsOff);
        }
    }
    #endif
	// 灯光循环
    LIGHT_LOOP_BEGIN(pixelLightCount)
	    // 获取额外光源数据
        Light light = GetAdditionalLight(lightIndex, inputData, shadowMask,
								         aoFactor);
		// 判断主灯光和网格渲染是否在同一层级，两者在同一层级时才计算主灯光的物理光照
        if (IsMatchingLightLayer(light.layerMask, meshRenderingLayers))
        {
	        // 计算附加灯光物理光照，并相叠加
            lightingData.additionalLightsColor += LightingPhysicallyBased(
						            brdfData, brdfDataClearCoat, light,
						            inputData.normalWS, 
						            inputData.viewDirectionWS,
						            surfaceData.clearCoatMask, 
						            specularHighlightsOff);
        }
    LIGHT_LOOP_END
    #endif
	
	// 启用附加灯光 Per Vertex（逐顶点计算）
    #if defined(_ADDITIONAL_LIGHTS_VERTEX)
    // 顶点光照在顶点着色器中计算，这里只需要从顶点着色器输出数据中取出
    lightingData.vertexLightingColor += 
	    inputData.vertexLighting * brdfData.diffuse;
    #endif
    
    /*-----------------------------------------------------------------------*/
	/*                          计算最后输出的渲染效果                          */
	/*-----------------------------------------------------------------------*/
    return CalculateFinalColor(lightingData, surfaceData.alpha);
}
```

Additional Lights Count

通过`GetAdditionalLightsCount()`着色器可以获得当前影响物体的附加灯光数量。
```hlsl
int GetAdditionalLightsCount()
{
#if USE_CLUSTERED_LIGHTING
    // Counting the number of lights in clustered requires traversing the bit list, and is not needed up front.
    // 官方注释：计算簇中灯光的数量需要遍历位列表，不需要预先进行。
    return 0;
#else
	/*
	官方注释：
    // TODO: we need to expose in SRP api an ability for the pipeline cap the amount of lights
    // in the culling. This way we could do the loop branch with an uniform
    // This would be helpful to support baking exceeding lights in SH as well
    我们需要在 SRP api 中公开管线限制灯光数量的功能。
    在剔除方面，这样我们就可以用统一的循环分支。
    这也有助于支持在 SH 中烘焙超出的灯光。（不确定exceeding lights如何翻译）
    */

	/*
		_AdditionalLightsCount.x：一个物体最大能接受的附加灯光数量
		unity_LightData.y：一个物体当前接收到的附加灯光数量		
	*/
	//限制影响物体的附加灯光数量不超过最大接受数量
    return int(min(_AdditionalLightsCount.x, unity_LightData.y));
#endif
}
```

### 4. 1、BRDF数据初始化
```hlsl
/*-----------------------------------------------------------------------*/ 
/*                                BRDF数据                                */ 
/*-----------------------------------------------------------------------*/ 
// 初始化BRDF数据 
BRDFData brdfData; 
InitializeBRDFData(surfaceData, brdfData);
```
Lit使用`InitializeBRDFData`和`InitializeBRDFDataDirect`两个函数完成BRDF数据初始化，这两个函数各有一个重载函数。 
#### 4. 1. 1、BRDFData 结构体
用来存放`InitializeBRDFData`计算的BRDF数据。
```hlsl
struct BRDFData
{
    half3 albedo;      // 基础色
    half3 diffuse;     // 漫反射
    half3 specular;    // 高光色
    half reflectivity; // 反射率
    half perceptualRoughness; // 感知粗糙度（之后说的粗糙度是这个感知粗糙度）
    half roughness;    // 粗糙度的二次方
    half roughness2;   // 粗糙度的四次方
    half grazingTerm;  

    // We save some light invariant BRDF terms so we don't have to recompute
    // them in the light loop. Take a look at DirectBRDF function for detailed explaination.
    // 官方注释：我们保存一些轻量不变的 BRDF 项，这样我们就不必重新计算它们在光循环中。查看 DirectBRDF 函数以获取详细说明。
    half normalizationTerm;     // roughness * 4.0 + 2.0
    half roughness2MinusOne;    // roughness^2 - 1.0
};
```
#### 4. 1. 2、BRDFData 初始化
##### （1）高光工作流数据
`InitializeBRDFData`函数确定要 inout 的 surfaceData 成员变量并回调其重载函数。
```hlsl
inline void InitializeBRDFData(inout SurfaceData surfaceData, out BRDFData brdfData)
{
    InitializeBRDFData(surfaceData.albedo,   surfaceData.metallic,
					   surfaceData.specular, surfaceData.smoothness, 
					   surfaceData.alpha,    brdfData);
}
```
`InitializeBRDFData`重载函数有两个作用：一是设置工作流，二是调用`InitializeBRDFDataDirect`计算其他BRDF数据。

工作流方面，着色器通过 `_SPECULAR_SETUP` 可以设置工作流是反射流（Specular）或金属流（Metallic）。工作流的不同就意味反射率、一减反射率、漫反射颜色、高光颜色均有不同。`InitializeBRDFData`如下：
```hlsl
inline void InitializeBRDFData(half3 albedo, half metallic, half3 specular, half smoothness, inout half alpha, out BRDFData outBRDFData)
{
#ifdef _SPECULAR_SETUP
	//Specular WorkFlow 反射流
    half reflectivity = ReflectivitySpecular(specular);
    half oneMinusReflectivity = half(1.0) - reflectivity;
    half3 brdfDiffuse = albedo * oneMinusReflectivity;
    half3 brdfSpecular = specular;
#else
	//Metallic WorkFlow 金属流
    half oneMinusReflectivity = OneMinusReflectivityMetallic(metallic);
    half reflectivity = half(1.0) - oneMinusReflectivity;
    half3 brdfDiffuse = albedo * oneMinusReflectivity;
    half3 brdfSpecular = lerp(kDieletricSpec.rgb, albedo, metallic);
#endif

    InitializeBRDFDataDirect(albedo, brdfDiffuse, brdfSpecular, reflectivity, 
							 oneMinusReflectivity, smoothness, 
							 alpha, outBRDFData);
}
```
`OneMinusReflectivityMetallic`计算了金属流模式下的一减反射率。
```hlsl
half OneMinusReflectivityMetallic(half metallic)
{
    // We'll need oneMinusReflectivity, so
    //   1-reflectivity = 1-lerp(dielectricSpec, 1, metallic) = lerp(1-dielectricSpec, 0, metallic)
    // store (1-dielectricSpec) in kDielectricSpec.a, then
    //   1-reflectivity = lerp(alpha, 0, metallic) = alpha + metallic*(0 - alpha) =
    //                  = alpha - metallic * alpha
    half oneMinusDielectricSpec = kDielectricSpec.a;
    return oneMinusDielectricSpec - metallic * oneMinusDielectricSpec;
}

```
反射流（Specular）与金属流（Metallic）的四个参数数据对比：

|     | 反射率（reflectivity）                      | 一减反射率（oneMinusReflectivity）        |
| --- | -------------------------------------- | ---------------------------------- |
| 反射流 | specular.r                             | 1 - specular.r                     |
| 金属流 | 1 - (1 - metallic) * kDielectricSpec.a | (1 - metallic) * kDielectricSpec.a |
|     |                                        |                                    |

|     | BRDF 漫反射（brdfDiffuse）                        | BRDF 高光色（brdfSpecular）                     |
| --- | -------------------------------------------- | ------------------------------------------ |
| 反射流 | albedo * (1 - specular.r)                    | specular.rgb                               |
| 金属流 | albedo * (1 - metallic) \* kDielectricSpec.a | lerp(kDieletricSpec.rgb, albedo, metallic) |
**注释：**
1. 使用反射流时的 specular 内存储的是反射贴图（光泽度贴图）；
2. kDielectricSpec 是Unity定义的入射角下的标准介质反射率系数（=4%），kDielectricSpec.a = 1.0 - 0.04 ；
3. 当前平台为 OpenGL ES 2.0 时，`ReflectivitySpecular`函数返回的反射率为 specular 的R通道，是其他平台时，返回的反射率是 specular 三个通道中值最大的。
##### （2）BRDF数据
`InitializeBRDFDataDirect`函数完成BRDF数据计算。
```hlsl
inline void InitializeBRDFDataDirect(half3 albedo, half3 diffuse, half3 specular, half reflectivity, half oneMinusReflectivity, half smoothness, inout half alpha, out BRDFData outBRDFData)
{
    outBRDFData = (BRDFData)0;
    outBRDFData.albedo = albedo;
    outBRDFData.diffuse = diffuse;
    outBRDFData.specular = specular;
    outBRDFData.reflectivity = reflectivity;
    
	// 用感知光滑度计算感知粗糙度，粗糙度的一次方
    outBRDFData.perceptualRoughness = PerceptualSmoothnessToPerceptualRoughness(smoothness);
    // 粗糙度的二次方，这个值不小于HALF_MIN_SQRT，即不小于0.0078125 
    outBRDFData.roughness           = max(PerceptualRoughnessToRoughness(outBRDFData.perceptualRoughness), HALF_MIN_SQRT);
    // 粗糙度的四次方，这个值不小于HALF_MIN，即不小于6.103515625e-5   
    outBRDFData.roughness2          = max(outBRDFData.roughness * outBRDFData.roughness, HALF_MIN);
    outBRDFData.grazingTerm         = saturate(smoothness + reflectivity);
    // normalizationTerm用于计算BRDF的V*F
    outBRDFData.normalizationTerm   = outBRDFData.roughness * half(4.0) + half(2.0);
    // 粗糙度的四次方减1
    outBRDFData.roughness2MinusOne  = outBRDFData.roughness2 - half(1.0);

// 与Alpha的multiply计算
#ifdef _ALPHAPREMULTIPLY_ON
    outBRDFData.diffuse *= alpha;
    alpha = alpha * oneMinusReflectivity + reflectivity; // NOTE: alpha modified and propagated up.
#endif
}
```
### 4. 2、渲染相关的数据准备 
```hlsl
/*-----------------------------------------------------------------------*/
/*                           渲染相关的数据准备                            */
/*-----------------------------------------------------------------------*/
// Clear-coat清漆渲染数据准备（无效）
BRDFData brdfDataClearCoat = CreateClearCoatBRDFData(surfaceData, brdfData);
// shadowMask阴影遮罩
half4 shadowMask = CalculateShadowMask(inputData);
// AO系数
AmbientOcclusionFactor aoFactor = CreateAmbientOcclusionFactor(inputData,
									surfaceData);
// 获取网格渲染层级
uint meshRenderingLayers = GetMeshRenderingLightLayer();
```
#### （1）ClearCoat 数据初始化
##### ●  CreateClearCoatBRDFData
创建一个存储清漆渲染相关数据的`BRDFData`结构体`brdfDataClearCoat`。该结构体并没有有效使用。
```hlsl
BRDFData CreateClearCoatBRDFData(SurfaceData surfaceData, inout BRDFData brdfData)
{
    BRDFData brdfDataClearCoat = (BRDFData)0;

    #if defined(_CLEARCOAT) || defined(_CLEARCOATMAP)
    InitializeBRDFDataClearCoat(surfaceData.clearCoatMask,
				surfaceData.clearCoatSmoothness, brdfData, brdfDataClearCoat);
    #endif

    return brdfDataClearCoat;
}
```
##### ●  InitializeBRDFDataClearCoat
计算渲染清漆需要的相关数据，如果使用非移动平台数据由`baseBRDFData`输出，移动平台则由`outBRDFData`输出。
```hlsl
inline void InitializeBRDFDataClearCoat(half clearCoatMask, half clearCoatSmoothness, inout BRDFData baseBRDFData, out BRDFData outBRDFData)
{
    outBRDFData = (BRDFData)0;
    outBRDFData.albedo = half(1.0);

    // Calculate Roughness of Clear Coat layer
    outBRDFData.diffuse             = kDielectricSpec.aaa; // 1 - kDielectricSpec
    outBRDFData.specular            = kDielectricSpec.rgb;
    outBRDFData.reflectivity        = kDielectricSpec.r;
	// 用感知光滑度计算感知粗糙度，粗糙度的一次方
    outBRDFData.perceptualRoughness = PerceptualSmoothnessToPerceptualRoughness(clearCoatSmoothness);
    // 粗糙度的二次方，这个值不小于HALF_MIN_SQRT，即不小于0.0078125 
    outBRDFData.roughness           = max(PerceptualRoughnessToRoughness(outBRDFData.perceptualRoughness), HALF_MIN_SQRT);
    // 粗糙度的四次方，这个值不小于HALF_MIN，即不小于6.103515625e-5   
    outBRDFData.roughness2          = max(outBRDFData.roughness * outBRDFData.roughness, HALF_MIN);
    // normalizationTerm用于计算BRDF的V*F
    outBRDFData.normalizationTerm   = outBRDFData.roughness * half(4.0) + half(2.0);
    // 粗糙度的四次方减1
    outBRDFData.roughness2MinusOne  = outBRDFData.roughness2 - half(1.0);
    outBRDFData.grazingTerm         = saturate(clearCoatSmoothness + kDielectricSpec.x);
//-----------------------------------------------------------------------------
// 非移动平台使用以下方法重新计算ClearCoat
//-----------------------------------------------------------------------------
//Relatively small effect, cut it for lower quality 
#if !defined(SHADER_API_MOBILE)
	/*
		官方注释：
	    // Modify Roughness of base layer using coat IOR
	    使用涂层 IOR 修改基层的粗糙度。
	    
	    关键字CLEAR_COAT_IETA：
	    #define CLEAR_COAT_IOR 1.5
	    #define CLEAR_COAT_IETA (1.0 / CLEAR_COAT_IOR)  
		IETA 是 eta 的倒数，是两个界面的 IOR 之比。
	*/
    half ieta                        = lerp(1.0h, CLEAR_COAT_IETA, clearCoatMask);
    // 涂层粗糙度强度
    half coatRoughnessScale          = Sq(ieta);
    // 计算感知粗糙度二次方的方差（使用的是最初传入由InitializeBRDFData计算的BRDF Data的感知粗糙度）
    half sigma                       = RoughnessToVariance(PerceptualRoughnessToRoughness(baseBRDFData.perceptualRoughness));
    
	// 计算新的感知粗糙度，(sigma * coatRoughnessScale)的方差
    baseBRDFData.perceptualRoughness = RoughnessToPerceptualRoughness(VarianceToRoughness(sigma * coatRoughnessScale));
    
	/*
	官方注释：
    //Recompute base material for new roughness, previous computation should be eliminated by the compiler (as it's unused)
    重新计算基础材料以获得新的粗糙度，编译器应消除先前的计算（因为它未使用）
    
    注意：未使用的是上方outBRDFData的计算结果，当前的计算结果将修改传出的是传入的BRDFData
    */
    
    // 粗糙度的二次方，这个值不小于HALF_MIN_SQRT，即不小于0.0078125
    baseBRDFData.roughness          = max(PerceptualRoughnessToRoughness(baseBRDFData.perceptualRoughness), HALF_MIN_SQRT);
    // 粗糙度的四次方，这个值不小于HALF_MIN，即不小于6.103515625e-5   
    baseBRDFData.roughness2         = max(baseBRDFData.roughness * baseBRDFData.roughness, HALF_MIN);
    // normalizationTerm用于计算BRDF的V*F
    baseBRDFData.normalizationTerm  = baseBRDFData.roughness * 4.0h + 2.0h;
    // 粗糙度的四次方减1
    baseBRDFData.roughness2MinusOne = baseBRDFData.roughness2 - 1.0h;
#endif

    // Darken/saturate base layer using coat to surface reflectance (vs. air to surface)
    // 官方注释：使用涂层到表面的反射率（相对于空气到表面）使基层变暗/饱和
    baseBRDFData.specular = lerp(baseBRDFData.specular, 
							    ConvertF0ForClearCoat15(baseBRDFData.specular), 
							    clearCoatMask);
    // TODO: what about diffuse? at least in specular workflow diffuse should be recalculated as it directly depends on it.
    // 官方注释：漫反射怎么样？ 至少在镜面工作流程中，应该重新计算漫反射，因为它直接依赖于它。
}
```

**● RoughnessToVariance**
在非移动端平台运行时，Lit会调用`RoughnessToVariance`重新计算ClearCoat使用的新粗糙度。
```hlsl
	// Use with stack BRDF (clear coat / coat) - This only used same equation to convert from Blinn-Phong spec power to Beckmann roughness
	// 官方注释：与堆栈 BRDF（透明涂层/涂层）一起使用 - 这只是使用相同的方程从 Blinn-Phong 规格功率转换为 Beckmann 粗糙度
real RoughnessToVariance(real roughness)
{
    return 2.0 / Sq(roughness) - 2.0;
}
```
#### （2）shadowMask 阴影遮罩
```hlsl
half4 shadowMask = CalculateShadowMask(inputData);
```
##### ●  CalculateShadowMask 
计算阴影遮罩，根据是否开启阴影和是否使用LightMap光照贴图分为三种情况。

【Unity 2021】
```hlsl
half4 CalculateShadowMask(InputData inputData)
{
    // To ensure backward compatibility we have to avoid using shadowMask input, as it is not present in older shaders
    // 官方注释：为了确保向后的兼容性，我们必须避免使用shadowMask输入，因为它在旧的着色器中不存在
    
	// 1.开启阴影遮罩和LIGHTMAP光照贴图
    #if defined(SHADOWS_SHADOWMASK) && defined(LIGHTMAP_ON)
    half4 shadowMask = inputData.shadowMask;
    // 2.只开启阴影遮罩
    #elif !defined (LIGHTMAP_ON)
    half4 shadowMask = unity_ProbesOcclusion;
    // 3.两者都不开启，输出阴影遮罩为1
    #else
    half4 shadowMask = half4(1, 1, 1, 1);
    #endif

    return shadowMask;
}
```

【Unity6】
```hlsl
half4 CalculateShadowMask(InputData inputData)
{
    // To ensure backward compatibility we have to avoid using shadowMask input, as it is not present in older shaders
    #if defined(SHADOWS_SHADOWMASK) && defined(LIGHTMAP_ON)
    half4 shadowMask = inputData.shadowMask; // Shadowmask was sampled from lightmap
    #elif !defined(LIGHTMAP_ON) && (defined(PROBE_VOLUMES_L1) || defined(PROBE_VOLUMES_L2))
    half4 shadowMask = inputData.shadowMask; // Shadowmask (probe occlusion) was sampled from APV
    #elif !defined (LIGHTMAP_ON)
    half4 shadowMask = unity_ProbesOcclusion; // Sample shadowmask (probe occlusion) from legacy probes
    #else
    half4 shadowMask = half4(1, 1, 1, 1); // Fallback shadowmask, fully unoccluded
    #endif

    return shadowMask;
}
```
##### ●  unity_ProbesOcclusion
Unity将`shadowMask`阴影遮罩数据烘焙到了光探针中，这个探针就是`unity_ProbesOcclusion`遮挡探针，或者我们可以将它称之为阴影探针。

`unity_ProbesOcclusion`的定义如下：
```hlsl
#define unity_ProbesOcclusion       UNITY_ACCESS_DOTS_INSTANCED_PROP_FROM_MACRO(float4,Metadataunity_ProbesOcclusion)
```
`UNITY_ACCESS_DOTS_INSTANCED_PROP_FROM_MACRO`相关详细参考--
#### （3） aoFactor 遮蔽因子
```hlsl
AmbientOcclusionFactor aoFactor = CreateAmbientOcclusionFactor(inputData, surfaceData);
```

AmbientOcclusionFactor 结构体
```hslsl
struct AmbientOcclusionFactor
{
    half indirectAmbientOcclusion; 
    half directAmbientOcclusion;
};
```
##### ●  CreateAmbientOcclusionFactor 
创建环境遮挡因子（aoFactor）。
```hlsl
//重载
AmbientOcclusionFactor CreateAmbientOcclusionFactor(float2 normalizedScreenSpaceUV, half occlusion)
{
    AmbientOcclusionFactor aoFactor = GetScreenSpaceAmbientOcclusion(normalizedScreenSpaceUV);
	
    aoFactor.indirectAmbientOcclusion = min(aoFactor.indirectAmbientOcclusion, occlusion);
    return aoFactor;
}

AmbientOcclusionFactor CreateAmbientOcclusionFactor(InputData inputData, SurfaceData surfaceData)
{
    return CreateAmbientOcclusionFactor(inputData.normalizedScreenSpaceUV, surfaceData.occlusion);
}
```
##### ●  GetScreenSpaceAmbientOcclusion
获取屏幕空间环境遮挡（SSAO）。只有在Unity管线开启屏幕空间遮挡且物体不透明时，Shader 才能计算 SSAO，否则直接环境遮蔽和间接环境遮蔽系数都为 1。
```hlsl
AmbientOcclusionFactor GetScreenSpaceAmbientOcclusion(float2 normalizedScreenSpaceUV)
{
    AmbientOcclusionFactor aoFactor;

	// 当Unity管线开启屏幕空间遮挡且物体不透明时计算SSAO
    #if defined(_SCREEN_SPACE_OCCLUSION) && !defined(_SURFACE_TYPE_TRANSPARENT)
    // 采样SSAO
    float ssao = SampleAmbientOcclusion(normalizedScreenSpaceUV);

    aoFactor.indirectAmbientOcclusion = ssao;
    aoFactor.directAmbientOcclusion = lerp(half(1.0), ssao, _AmbientOcclusionParam.w);
    #else
    aoFactor.directAmbientOcclusion = 1;
    aoFactor.indirectAmbientOcclusion = 1;
    #endif

	// DEBUG显示内容
    #if defined(DEBUG_DISPLAY)
    switch(_DebugLightingMode)
    {
        case DEBUGLIGHTINGMODE_LIGHTING_WITHOUT_NORMAL_MAPS:
            aoFactor.directAmbientOcclusion = 0.5;
            aoFactor.indirectAmbientOcclusion = 0.5;
            break;

        case DEBUGLIGHTINGMODE_LIGHTING_WITH_NORMAL_MAPS:
            aoFactor.directAmbientOcclusion *= 0.5;
            aoFactor.indirectAmbientOcclusion *= 0.5;
            break;
    }
    #endif

    return aoFactor;
}
```
`SampleAmbientOcclusion`会采样一张名为 ScreenSpaceOcclusionTexture 的由渲染管线生成的SSAO RT贴图。

ScreenSpaceAmbientOcclusion 相关详请参考 [[Shader 知识点]] 。
### 4. 3、灯光、光照相关数据
```hlsl
/*-----------------------------------------------------------------------*/
/*                           灯光、光照相关数据                            */
/*-----------------------------------------------------------------------*/
// 获取主灯光数据
Light mainLight = GetMainLight(inputData, shadowMask, aoFactor);

// 混合实时光照和烘焙GI
MixRealtimeAndBakedGI(mainLight, inputData.normalWS, inputData.bakedGI);
// 创建一个光照数据结构用来存储一些光照颜色类的数据
LightingData lightingData = CreateLightingData(inputData, surfaceData);
```
#### （1）Light 结构体
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
#### （2）主灯光数据
使用函数`GetMainLight`可以获取到主灯光的相关数据（Light 结构中的变量）。

● GetMainLight 函数定义如下：
GetMainLight 有两次重载，详请参考 [[Shader 知识点]] 。
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
#### （3）混合实时光照和烘焙GI
```hlsl
MixRealtimeAndBakedGI(mainLight, inputData.normalWS, inputData.bakedGI);
```
因为烘焙的光照贴图已经计算了主光源的光照和阴影，所以“混合”实际就是从光照贴图中减去直接主光源的光照和阴影，以避免与实时光照再叠加。最终得到的阴影是这个差值和实时阴影中更暗的那个。

要使用 MixRealtimeAndBakedGI 需要同时满足两个条件：
- Lit 使用光照贴图LightMap；
- 混合光照模式为Subtractive（Linghting>Scene>Mixed Lighting>Lighting Mode选项为Subtractive）。

**注意**：如果注释掉`MixRealtimeAndBakedGI`，使用LightMap的静态物体将无法接受来自动态物体的阴影。

MixRealtimeAndBakedGI 详请参考[[Shader 知识点]] 。
#### （4）创建光照数据结构
使用`CreateLightingData`创建一个 LightingData 结构，用来存储输出的光照颜色数据。

**● LightingData**
```hlsl
struct LightingData
{
	// 全局光照颜色
    half3 giColor;
    // 主灯光颜色
    half3 mainLightColor;
    // 额外光颜色
    half3 additionalLightsColor;
    // 顶点光颜色
    half3 vertexLightingColor;
    // 自发光颜色
    half3 emissionColor;
};
```
**● CreateLightingData**
```HLSL
LightingData CreateLightingData(InputData inputData, SurfaceData surfaceData)
{
    LightingData lightingData;

    lightingData.giColor = inputData.bakedGI;
    lightingData.emissionColor = surfaceData.emission;
    lightingData.vertexLightingColor = 0;
    lightingData.mainLightColor = 0;
    lightingData.additionalLightsColor = 0;

    return lightingData;
}
```
### 4. 4、全局光照
```hlsl
/*-----------------------------------------------------------------------*/
/*                                全局光照                                */
/*-----------------------------------------------------------------------*/
lightingData.giColor = GlobalIllumination(brdfData,brdfDataClearCoat,
				surfaceData.clearCoatMask,inputData.bakedGI,
				aoFactor.indirectAmbientOcclusion,inputData.positionWS,
				inputData.normalWS, inputData.viewDirectionWS);	   
```
在前向渲染中全局光照部分计算的是间接光照，间接光照又分为了间接光漫反射（Indirect Diffuse）和间接光高光（Indirect Specular）两个部分。`GlobalIllumination`函数如下：
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

#if defined(_CLEARCOAT) || defined(_CLEARCOATMAP)
    half3 coatIndirectSpecular = GlossyEnvironmentReflection(reflectVector, positionWS, brdfDataClearCoat.perceptualRoughness, 1.0h);
    // TODO: "grazing term" causes problems on full roughness
    half3 coatColor = EnvironmentBRDFClearCoat(brdfDataClearCoat, clearCoatMask, coatIndirectSpecular, fresnelTerm);

    // Blend with base layer using khronos glTF recommended way using NoV
    // Smooth surface & "ambiguous" lighting
    // NOTE: fresnelTerm (above) is pow4 instead of pow5, but should be ok as blend weight.
    half coatFresnel = kDielectricSpec.x + kDielectricSpec.a * fresnelTerm;
    return (color * (1.0 - coatFresnel * clearCoatMask) + coatColor) * occlusion;
#else
    return color * occlusion;
#endif
}
```
##### （1）间接光漫反射
间接光的漫反射就是烘焙光照，没有 LightMap 时是球谐光照。
```hlsl
half3 indirectDiffuse = bakedGI;
```
##### （2）间接光高光
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

	#if defined(UNITY_USE_NATIVE_HDR)  // Unity使用原生的HDR
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
计算间接光的镜面反射光（环境高光），再加上间接光照的漫反射（环境反射），得到完整的间接光照。
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
### 4. 5、主灯光光照
```hlsl
/*-----------------------------------------------------------------------*/
/*                               主灯光光照                               */
/*-----------------------------------------------------------------------*/
if (IsMatchingLightLayer(mainLight.layerMask, meshRenderingLayers))
{
	// 计算主灯光物理光照
	lightingData.mainLightColor = LightingPhysicallyBased(brdfData,
				brdfDataClearCoat,mainLight,inputData.normalWS, 
				inputData.viewDirectionWS,surfaceData.clearCoatMask, 
				specularHighlightsOff);
}
```
#### ● LightingPhysicallyBased
计算物理光照。
`LightingPhysicallyBased` 的传入参数：
```hlsl
BRDFData brdfData 
	- (half3)brdfData.diffuse  // BRDF 漫反射
	- (half3)brdfData.specular // BRDF 高光颜色
BRDFData brdfDataClearCoat  // 未使用
Light light
	- (half3)light.color       // 灯光颜色
	- (half3)light.direction   // 灯光方向
	- (half) light.distanceAttenuation * light.shadowAttenuation // 灯光距离衰减 * 灯光阴影衰减 
half3 normalWS                // 世界空间法线
half3 viewDirectionWS         // 世界空间观察方向
half clearCoatMask            // 未使用
half specularHighlightsOff    // 高光反射开关
```
`LightingPhysicallyBased` 函数定义如下：
```hlsl
half3 LightingPhysicallyBased(BRDFData brdfData, BRDFData brdfDataClearCoat, Light light, half3 normalWS, half3 viewDirectionWS, half clearCoatMask, bool specularHighlightsOff)
{
    return LightingPhysicallyBased(brdfData, brdfDataClearCoat, light.color, light.direction,
							     light.distanceAttenuation * light.shadowAttenuation, 
							     normalWS, viewDirectionWS, clearCoatMask, specularHighlightsOff);
}
```
`LightingPhysicallyBased` 的重载函数计算物理光照：
```hlsl
half3 LightingPhysicallyBased(BRDFData brdfData, BRDFData brdfDataClearCoat,
    half3 lightColor, half3 lightDirectionWS, half lightAttenuation,
    half3 normalWS, half3 viewDirectionWS,
    half clearCoatMask, bool specularHighlightsOff)
{
	// Lambert Lighting
    half NdotL = saturate(dot(normalWS, lightDirectionWS));
    half3 radiance = lightColor * (lightAttenuation * NdotL);

	// BRDF漫反射
    half3 brdf = brdfData.diffuse;
#ifndef _SPECULARHIGHLIGHTS_OFF // GUI的宏开关，控制开关高光反射
    [branch] if (!specularHighlightsOff)
    {
	    // BRDF = BRDF漫反射 + BRDF高光反射
        brdf += brdfData.specular * DirectBRDFSpecular(brdfData, normalWS, lightDirectionWS, viewDirectionWS);

// Lit Shader并没有启用透明涂层渲染，所有以下透明涂层的渲染计算无效。
/*
#if defined(_CLEARCOAT) || defined(_CLEARCOATMAP)
        half brdfCoat = kDielectricSpec.r * DirectBRDFSpecular(brdfDataClearCoat, normalWS, lightDirectionWS, viewDirectionWS);

            half NoV = saturate(dot(normalWS, viewDirectionWS));
            half coatFresnel = kDielectricSpec.x + kDielectricSpec.a * Pow4(1.0 - NoV);

        brdf = brdf * (1.0 - clearCoatMask * coatFresnel) + brdfCoat * clearCoatMask;
#endif // _CLEARCOAT
*/
    }
#endif // _SPECULARHIGHLIGHTS_OFF
	
	// BRDF乘以光照模型计算物理光照
    return brdf * radiance;
}
```
#### ● DirectBRDFSpecular
计算直接光的BRDF高光反射，也可称为镜面反射。
`DirectBRDFSpecular`传入参数：
```hlsl
BRDFData brdfData 
	- (half)brdfData.roughness2MinusOne  // 粗糙度的四次方减1
	- (half)brdfData.roughness2          // 粗糙度的四次方
	- (half)brdfData.normalizationTerm   // 轻量级不变的BRDF项
half3 normalWS           // 世界空间法线
half3 lightDirectionWS   // 灯光方向
half3 viewDirectionWS    // 世界空间观察方向
```
`DirectBRDFSpecular`函数定义：
```hlsl
half DirectBRDFSpecular(BRDFData brdfData, half3 normalWS, half3 lightDirectionWS, half3 viewDirectionWS)
{
	// 灯光方向（世界空间）
    float3 lightDirectionWSFloat3 = float3(lightDirectionWS);
    // 半角向量
    float3 halfDir = SafeNormalize(lightDirectionWSFloat3 + float3(viewDirectionWS));
	
    float NoH = saturate(dot(float3(normalWS), halfDir));
    half LoH = half(saturate(dot(lightDirectionWSFloat3, halfDir)));
    
	// 法线分布项D GGX
    float d = NoH * NoH * brdfData.roughness2MinusOne + 1.00001f;

	// D项代入Unity官方公式 BRDFspec = (D * V * F) / 4.0 计算高光
    half LoH2 = LoH * LoH;
    half specularTerm = brdfData.roughness2 / ((d * d) * max(0.1h, LoH2) * brdfData.normalizationTerm);

// 移动平台支持half精度时启用。官方文档查不到SHADER_API_SWITCH这个宏
#if defined (SHADER_API_MOBILE) || defined (SHADER_API_SWITCH)
    specularTerm = specularTerm - HALF_MIN; // HALF_MIN : 6.103515625e-5（2^-14）
    specularTerm = clamp(specularTerm, 0.0, 100.0); // 防止FP16（HDR Mode）在手机上溢出
#endif

	return specularTerm;
}
```
**相关补充：**
**1）FP16**
HDR 缓冲区的格式，图形层启用 **HDR** 时默认为FP16。

**2）HALF_MIN**
官方在 Macros.hlsl 中定义的宏。
```hlsl
#define HALF_MIN 6.103515625e-5 // 2^-14
```

**3）\[branch]**
`UNITY_BRANCH`在 HLSL 平台上扩展为 `[branch]`。在条件语句之前添加`UNITY_BRANCH`宏，告知编译器应将其编译为实际分支。
### 4. 6、计算最后输出的渲染效果
```hlsl
/*-----------------------------------------------------------------------*/
/*                          计算最后输出的渲染效果                          */
/*-----------------------------------------------------------------------*/
return CalculateFinalColor(lightingData, surfaceData.alpha);
```
`CalculateFinalColor`函数定义如下：
```hlsl
half4 CalculateFinalColor(LightingData lightingData, half alpha)
{
    half3 finalColor = CalculateLightingColor(lightingData, 1);

    return half4(finalColor, alpha);
}
```

CalculateLightingColor 计算最终输出的渲染效果，就是将全局光、物理光照、附加光照、自发光相加。函数中使用了很多 if 分支，是为控制部分光照效果的开光和 Debug。
```hlsl
half3 CalculateLightingColor(LightingData lightingData, half3 albedo)
{
    half3 lightingColor = 0;

	// 只输出AO（计算全局光时只输出了occlusion）
    if (IsOnlyAOLightingFeatureEnabled())
    {
        return lightingData.giColor; // Contains white + AO
    }

	// 在最终输出光照中加上全局光照GI
    if (IsLightingFeatureEnabled(DEBUGLIGHTINGFEATUREFLAGS_GLOBAL_ILLUMINATION))
    {
        lightingColor += lightingData.giColor;
    }
    
	// 在最终输出光照中加上主灯光光照
    if (IsLightingFeatureEnabled(DEBUGLIGHTINGFEATUREFLAGS_MAIN_LIGHT))
    {
        lightingColor += lightingData.mainLightColor;
    }

	// 在最终输出光照中加上附加灯光光照
    if (IsLightingFeatureEnabled(DEBUGLIGHTINGFEATUREFLAGS_ADDITIONAL_LIGHTS))
    {
        lightingColor += lightingData.additionalLightsColor;
    }

	// 在最终输出光照中加上顶点光照
    if (IsLightingFeatureEnabled(DEBUGLIGHTINGFEATUREFLAGS_VERTEX_LIGHTING))
    {
        lightingColor += lightingData.vertexLightingColor;
    }

	// 这个地方很奇怪，CalculateLightingColor 输入的albedo是1
    lightingColor *= albedo;

	// 在最终输出光照中加上自发光
    if (IsLightingFeatureEnabled(DEBUGLIGHTINGFEATUREFLAGS_EMISSION))
    {
        lightingColor += lightingData.emissionColor;
    }

    return lightingColor;
}

half4 CalculateFinalColor(LightingData lightingData, half alpha)
{
    half3 finalColor = CalculateLightingColor(lightingData, 1);

    return half4(finalColor, alpha);
}
```
CalculateFinalColor 的重载函数用于 Unlit Shader 无光照着色器：
```hlsl
half4 CalculateFinalColor(LightingData lightingData, half3 albedo, half alpha, float fogCoord)
{
    #if defined(_FOG_FRAGMENT)
        #if (defined(FOG_LINEAR) || defined(FOG_EXP) || defined(FOG_EXP2))
        float viewZ = -fogCoord;
        float nearToFarZ = max(viewZ - _ProjectionParams.y, 0);
        half fogFactor = ComputeFogFactorZ0ToFar(nearToFarZ);
    #else
        half fogFactor = 0;
        #endif
    #else
    half fogFactor = fogCoord;
    #endif
    half3 lightingColor = CalculateLightingColor(lightingData, albedo);
    half3 finalColor = MixFog(lightingColor, fogFactor);

    return half4(finalColor, alpha);
}
```


