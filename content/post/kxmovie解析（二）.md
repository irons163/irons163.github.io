---
title: "kxmovie解析（二）"
date: 2021-02-13T15:14:15+08:00
tags : [ "FFMpeg", "OpenGL", "iOS"]
layout: post
type:  "post"
draft: false
---

3. Set Program and Shader

`GLuint ShaderProgram = glCreateProgram();`
創建一個program object，待會將所有的shaders連結到這個object上。
`GLuint ShaderObj = glCreateShader(ShaderType);`
創建兩個shader objects，`GL_VERTEX_SHADER` , `GL_FRAGMENT_SHADER`。

```objc
GLint status;
const GLchar *sources = (GLchar *)shaderString.UTF8String;
    
    GLuint shader = glCreateShader(type);
    if (shader == 0 || shader == GL_INVALID_ENUM) {
        LoggerVideo(0, @"Failed to create shader %d", type);
        return 0;
    }
    
    glShaderSource(shader, 1, &sources, NULL);
    glCompileShader(shader);
NSString *const vertexShaderString = SHADER_STRING
(
 attribute vec4 position;
 attribute vec2 texcoord;
 uniform mat4 modelViewProjectionMatrix;
 varying vec2 v_texcoord;
 
 void main()
 {
     gl_Position = modelViewProjectionMatrix * position;
     v_texcoord = texcoord.xy;
 }
);
NSString *const rgbFragmentShaderString = SHADER_STRING
(
 varying highp vec2 v_texcoord;
 uniform sampler2D s_texture;
 
 void main()
 {
     gl_FragColor = texture2D(s_texture, v_texcoord);
 }
);
```

先create shader object，然後判斷一下是否創建成功，之後在編譯shader之前，必須指定其source code，`glShaderSource`將`shaderString(source code)`連結shader，
這邊的source code為`vertexShaderString`與`rgbFragmentShaderString`。
稍後會詳加介紹。

`glCompileShader(ShaderObj);`
編譯shader。

```objc
GLint status;
glGetShaderiv(shader, GL_COMPILE_STATUS, &status);
    if (status == GL_FALSE) {
        glDeleteShader(shader);
        LoggerVideo(0, @"Failed to compile shader:\n");
        return 0;
    }
```

檢查shader是否編譯成功。

`glAttachShader(ShaderProgram, ShaderObj);`
將編譯好的shader跟shaderProgram object做連結。
`glLinkProgram(ShaderProgram);`
到這裡program object已經設定完畢，將其與OpenGL連結。

```objc
glGetProgramiv(ShaderProgram, GL_LINK_STATUS, &Success);
if (Success == 0) {
    glGetProgramInfoLog(ShaderProgram, sizeof(ErrorLog), NULL, ErrorLog);
    fprintf(stderr, "Error linking shader program: '%s'\n", ErrorLog);
}
```

這裏使用`glGetProgramInfoLog`來取得錯誤訊息。

`glValidateProgram(ShaderProgram);`
確認program合法性，在每一禎被繪製前做檢查，避免多個shader或複雜的狀態改變時會出錯。
`glUseProgram(ShaderProgram);`
最後，使用這個program。這個program將用於所有的繪製呼叫，直到將program做更換或設為`ＮＵＬＬ`(透過`glUseProgram`)。


4. Shader Source Code
在上述步驟中，會發現shader 的source code，有許多意義不明的東西。
這裡來說明那些東西的意義。

```objc
NSString *const vertexShaderString = SHADER_STRING
(
 attribute vec4 position;
 attribute vec2 texcoord;
 uniform mat4 modelViewProjectionMatrix;
 varying vec2 v_texcoord;
 
 void main()
 {
     gl_Position = modelViewProjectionMatrix * position;
     v_texcoord = texcoord.xy;
 }
);
NSString *const rgbFragmentShaderString = SHADER_STRING
(
 varying highp vec2 v_texcoord;
 uniform sampler2D s_texture;
 
 void main()
 {
     gl_FragColor = texture2D(s_texture, v_texcoord);
 }
);
```

變量修飾符
修飾符給出了變量的特殊含義，有以下修飾符：
· const – 聲明一個編譯期常量。
· attribute– 隨不同頂點變化的全局變量，由OpenGL應用程序傳給頂點shader。這個修飾符只能用在頂點shader中，在shader中它是一個只讀變量。
· uniform– 隨不同圖變化的全局變量（即不能在`glBegin`/`glEnd`中設置），由OpenGL應用程序傳給shader。這個修飾符能用在頂點和片斷shader中，在shader中它是一個只讀變量。
· varying –用於頂點shader和片斷shader間傳遞的插值數據，在頂點shader中可寫，在片斷shader中只讀。

1. Uniform變量
不同於頂點屬性在每個頂點有其自己的值，一個uniform變量在一個圖的繪製過程中是不會改變的，uniform變量是外部application程序傳遞給（vertex和fragment）shader的變量，uniform變量就像是C語言裡面的常量（const ），它不能被shader程序修改。 （shader只能用，不能改）

獲取一個uniform變量的存儲位置只需要給出其在shader中定義的變量名即可：

uniform變量一般用來表示：變換矩陣，材質，光照參數和顏色等信息


以下是例子：
```objc
uniform mat4 viewProjMatrix; //投影+視圖矩陣
uniform mat4 viewMatrix; //視圖矩陣
uniform vec3 position; //光源位置
```

2.Attribute變量
attribute變量是只能在vertex shader中使用的變量。 （它不能在fragment shader中聲明attribute變量，也不能被fragment shader中使用）
一般用attribute變量來表示一些頂點的數據，如：頂點坐標，法線，紋理坐標，頂點顏色等。
在application中，一般用函數`glBindAttribLocation（）`來綁定每個attribute變量的位置，然後用函數`glVertexAttribPointer（）`為每個attribute變量賦值。

以下是例子：
```objc
uniform mat4 u_matViewProjection;
attribute vec4 a_position;
attribute vec2 a_texCoord0;
varying vec2 v_texCoord;
void main(void)
{
gl_Position = u_matViewProjection * a_position;
v_texCoord = a_texCoord0;
}
```

3.varying變量
varying變量是vertex和fragment shader之間做數據傳遞用的。一般vertex shader修改varying變量的值，然後fragment shader使用該varying變量的值。因此varying變量在vertex和fragment shader二者之間的聲明必須是一致的。 application不能使用此變量。

以下是例子：

```objc
// Vertex shader
uniform mat4 u_matViewProjection;
attribute vec4 a_position;
attribute vec2 a_texCoord0;
varying vec2 v_texCoord; // Varying in vertex shader
void main(void)
{
gl_Position = u_matViewProjection * a_position;
v_texCoord = a_texCoord0;
}
```

```objc
// Fragment shader
precision mediump float;
varying vec2 v_texCoord; // Varying in fragment shader
uniform sampler2D s_bas​​eMap;
uniform sampler2D s_lightMap;
void main()
{
vec4 baseColor;
vec4 lightColor;
baseColor = texture2D(s_bas​​eMap, v_texCoord);
lightColor = texture2D(s_lightMap, v_texCoord);
gl_FragColor = baseColor * (lightColor + 0.25);
}
```

4. Vectors and Matrix

```objc
vec2 a = vec2(1.0, 2.0);
vec3 b = vec3(-1.0, 0.0, 0.0);
vec4 c = vec4(0.0, 0.0, 0.0, 1.0);

mat3 m = mat3(
   1.1, 2.1, 3.1, // first column (not row!)
   1.2, 2.2, 3.2, // second column
   1.3, 2.3, 3.3  // third column
);
```

要特別注意Matrix的橫列代表矩陣的column，這在矩陣乘法中有差別。

5.精度修飾符
任何浮點數或者整數聲明前面都可以添加如下精度修飾符：


舉例：
```objc
lowp float color;
varying mediump vec2 Coord;
lowp ivec2 foo(lowp mat3);
highp mat4 m;
```
       
精度修飾符聲明了底層實現存儲這些變量必須要使用的最小範圍和精度。實現可能會使用比要求更大的範圍和精度，但絕對不會比要求少。

