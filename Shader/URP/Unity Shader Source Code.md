## TEXTURE2D宏
TEXTURE2D的定义在Library\PackageCache\com.unity.render-pipelines.core@12.1.10\ShaderLibrary\API目录下的7个HLSL文件中：
```hlsl
D3D11.hlsl
GLCore.hlsl
GLES2.hlsl
GLES3.hlsl
Metal.hlsl
Switch.hlsl
Validate.hlsl
Vulkan.hlsl
```
常用的TEXTURE2D定义：
```hlsl
#define TEXTURE2D(textureName)                         Texture2D textureName
#define TEXTURECUBE(textureName)                       TextureCube textureName

#define TEXTURE2D_ARGS(textureName, samplerName)       textureName, samplerName
#define TEXTURECUBE_ARGS(textureName, samplerName)     textureName, samplerName

#define TEXTURE2D_PARAM(textureName, samplerName)      TEXTURE2D(textureName), SAMPLER(samplerName)
#define TEXTURECUBE_PARAM(textureName, samplerName)    TEXTURECUBE(textureName), SAMPLER(samplerName)
```
