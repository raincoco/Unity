# URP-Lit
Lit Shader是URP管线内置的用于渲染写实效果的光照着色器，该着色器使用URP中计算量最大的着色模型。  
Lit Shader要正常渲染至少需要保证有ForwardLit、DepthNormals两个Pass，如果要渲染阴影则还需ShadowCaster Pass。
## 一、ForwardLit Pass
Lit Shader的前向渲染Pass，这个Pass包含了大量的shader_feature与multi_compile变体。  

补充：shader_feature与multi_compile的区别  
两者的区别在于Unity在最终的版本中不会包括shader_feature着色器的未使用的变体。shader_feature更适合处理从material中设置的关键字，而multi_compile则更适合用来处理从全局代码中设置的关键字。
### 1、ForwardLit Pass的结构
这个Pass的代码结构都包含在以下两个hsls文件中。
```hlsl
#include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
#include "Packages/com.unity.render-pipelines.universal/Shaders/LitForwardPass.hlsl"
```
LitInput.hlsl定义了Shader所需要输入的数据变量，LitForwardPass.hlsl则负责Shader的渲染流程。  
