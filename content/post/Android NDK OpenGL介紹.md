---
title: "Android NDK OpenGL介紹"
date: 2021-02-28T16:17:42+08:00
tags : [ "OpenGL", "Android"]
layout: post
type:  "post"
draft: false
---

現在來介紹Android NDK OpenGL 的 code，

這段code是出在這裡：
http://blog.csdn.net/wangchenggggdn/article/details/8896453
本來想從ios的code來porting，不過人家有類似的android code，就直接拿來修改比較快。




#include <jni.h>
#include <android/log.h>

#include <GLES2/gl2.h>
#include <GLES2/gl2ext.h>

#include <stdio.h>
#include <stdlib.h>
#include <math.h>

#include "Shader.vert"
#include "Shader.frag"

#include "SenaoCommon.h"

這是開頭需要include沒什麼好講，主要要看的是
#include "Shader.vert"
#include "Shader.frag"
這兩個會導入vertrice and fragment shader code，如下：

//Shader.vert文件内容
static const char* VERTEX_SHADER =
"attribute vec4 vPosition;    \n"
"attribute vec2 a_texCoord;   \n"
"varying vec2 tc;     \n"
"void main()                  \n"
"{                            \n"
"   gl_Position = vPosition;  \n"
"   tc = a_texCoord;  \n"
"}                            \n";


//Shader.frag文件内容
static const char* FRAG_SHADER =
"varying lowp vec2 tc;\n"
"uniform sampler2D SamplerY;\n"
"uniform sampler2D SamplerU;\n"
"uniform sampler2D SamplerV;\n"
"void main(void)\n"
"{\n"
"mediump vec3 yuv;\n"
"lowp vec3 rgb;\n"
"yuv.x = texture2D(SamplerY, tc).r;\n"
"yuv.y = texture2D(SamplerU, tc).r - 0.5;\n"
"yuv.z = texture2D(SamplerV, tc).r - 0.5;\n"
"rgb = mat3( 1,   1,   1,\n"
"0,       -0.39465,  2.03211,\n"
"1.13983,   -0.58060,  0) * yuv;\n"
"gl_FragColor = vec4(rgb, 1);\n"
"}\n";


這段shader code之前出現過很多次了，這邊就跳過不介紹了。




int ATTRIB_VERTEX,ATTRIB_TEXTURE;

這邊是ATTRIB_VERTEX跟ATTRIB_TEXTURE。
很重要等等會用到。



static GLuint g_texYId;
static GLuint g_texUId;
static GLuint g_texVId;
static GLuint simpleProgram;

static char *              g_buffer = NULL;
static int                 g_width = 0;
static int                 g_height = 0;
static int                 backing_width = 0;
static int                 backing_height = 0;

一些參數，也是之前介紹ios時有看過的，
三個texture，program，g_buffer（pixel data）, g_width (frame width), g_height(frame height), backing_width(view width), backing_height(view height).





static GLfloat squareVertices[] = {
        -1.0f, -1.0f,
        1.0f, -1.0f,
        -1.0f,  1.0f,
        1.0f,  1.0f,
    };


這邊頂點座標也是跟ios一樣。





static void checkGlError(const char* op)
{
    GLint error;
    for (error = glGetError(); error; error = glGetError())
    {
        LOGI("error::after %s() glError (0x%x)\n", op, error);
    }
}


印出log，LOGI是定義在senaoCommon裡的。





static GLuint buildShader(const char* source, GLenum shaderType)
{
    GLuint shaderHandle = glCreateShader(shaderType);

    if (shaderHandle)
    {
        glShaderSource(shaderHandle, 1, &source, 0);
        glCompileShader(shaderHandle);

        GLint compiled = 0;
        glGetShaderiv(shaderHandle, GL_COMPILE_STATUS, &compiled);
        if (!compiled)
        {
            GLint infoLen = 0;
            glGetShaderiv(shaderHandle, GL_INFO_LOG_LENGTH, &infoLen);
            if (infoLen)
            {
                char* buf = (char*) malloc(infoLen);
                if (buf)
                {
                    glGetShaderInfoLog(shaderHandle, infoLen, NULL, buf);
                    LOGE("error::Could not compile shader %d:\n%s\n", shaderType, buf);
                    free(buf);
                }
                glDeleteShader(shaderHandle);
                shaderHandle = 0;
            }
        }else{
        
LOGI("compiled shader");
        }

    }

    return shaderHandle;
}



buildShader用來建立shader，glShaderSource導入source code，glCompileShader進行編譯。
 if (!compiled) 編譯不成功，印出訊息。



static GLuint buildProgram(const char* vertexShaderSource,
        const char* fragmentShaderSource)
{
    GLuint vertexShader = buildShader(vertexShaderSource, GL_VERTEX_SHADER);
    GLuint fragmentShader = buildShader(fragmentShaderSource, GL_FRAGMENT_SHADER);
    GLuint programHandle = glCreateProgram();

    if (programHandle)
    {
        glAttachShader(programHandle, vertexShader);
        checkGlError("glAttachShader");
        glAttachShader(programHandle, fragmentShader);
        checkGlError("glAttachShader");
        glLinkProgram(programHandle);

        GLint linkStatus = GL_FALSE;
        glGetProgramiv(programHandle, GL_LINK_STATUS, &linkStatus);
        if (linkStatus != GL_TRUE) {
            GLint bufLength = 0;
            glGetProgramiv(programHandle, GL_INFO_LOG_LENGTH, &bufLength);
            if (bufLength) {
                char* buf = (char*) malloc(bufLength);
                if (buf) {
                    glGetProgramInfoLog(programHandle, bufLength, NULL, buf);
//                    log_easy("error::Could not link program:\n%s\n", buf);
                    free(buf);
                }
            }
            glDeleteProgram(programHandle);
            programHandle = 0;
        }

    }

    return programHandle;
}



    GLuint vertexShader = buildShader(vertexShaderSource, GL_VERTEX_SHADER);
    GLuint fragmentShader = buildShader(fragmentShaderSource, GL_FRAGMENT_SHADER);
    GLuint programHandle = glCreateProgram();

glGetProgramiv(programHandle, GL_LINK_STATUS, &linkStatus);
判斷link status，失敗就印出訊息。



void gl_initialize()
{
    g_buffer = NULL;

    LOGI("OPEN_GL Init");

    simpleProgram = buildProgram(VERTEX_SHADER, FRAG_SHADER);
    glUseProgram(simpleProgram);
    glGenTextures(1, &g_texYId);
    glGenTextures(1, &g_texUId);
    glGenTextures(1, &g_texVId);
}


一開使要先call 此init函式。
先build program再use。
產生三個textures(YUV)。





void gl_uninitialize()
{

    g_width = 0;
    g_height = 0;

    if (g_buffer)
    {
        free(g_buffer);
        g_buffer = NULL;
    }
}


結束時call此uninit函式進行釋放。



//設置圖像數據
void gl_set_framebuffer(const char* buffer, int buffersize, int width, int height)
{

    if (g_width != width || g_height != height)
    {
        if (g_buffer)
            free(g_buffer);

        g_width = width;
        g_height = height;

        g_buffer = (char *)malloc(buffersize);
    }

    if (g_buffer)
        memcpy(g_buffer, buffer, buffersize);

}


傳入frame data，這邊進行memcpy，將data拷貝，之後在繪製texture時會用到。
我覺得應該有辦法省掉這個copy，這樣可以讓效率提升一些。




void updateVertices(int glWidth, int glHeight);

先宣告函式，因為等等要用。





void gl_setbackingAndGLWH(int backingWidth, int backingHeight, int glWidth, int glHeight){
backing_width = backingWidth;
backing_height = backingHeight;
updateVertices(glWidth, glHeight);
}


傳入畫面寬高(就是surfaceview的寬高)與frame寬高。
並且更新頂點座標。



void updateVertices(int glWidth, int glHeight)
{
const bool fit = true;
const float width   = glWidth;
const float height  = glHeight;
const float dH      = (float)backing_height / height;
const float dW      = (float)backing_width  / width;
const float dd      = fit ? (dH<dW ? dH:dW) : (dH>dW ? dH:dW);
const float h       = (height * dd / (float)backing_height);
const float w       = (width  * dd / (float)backing_width );

squareVertices[0] = - w;
squareVertices[1] = - h;
squareVertices[2] =   w;
squareVertices[3] = - h;
squareVertices[4] = - w;
squareVertices[5] =   h;
squareVertices[6] =   w;
squareVertices[7] =   h;
}


更新頂點座標的算法是把ios的code寫成android，所以運作原理同ios。
根據fit來決定顯示方式：
fit為true時，畫面不裁切，會變形。
fit為false時，畫面會裁切，不會變形。




//畫屏
void gl_render_frame()
{
    if (0 == g_width || 0 == g_height)
        return;

    const char *buffer = g_buffer;
    int width = g_width;
    int height = g_height;

    glViewport(0, 0, backing_width, backing_height);
    bindTexture(g_texYId, buffer, width, height);
    bindTexture(g_texUId, buffer + width * height, width/2, height/2);
    bindTexture(g_texVId, buffer + width * height * 5 / 4, width/2, height/2);
   renderFrame();
}
//如果設置圖像數據和花屏是兩個線程的話,記住要加鎖。


呼叫gl_render_frame用以繪圖，
g_width，g_height是frame大小，
glViewport(0, 0, backing_width, backing_height);是設定視圖大小，範圍就是surfaceview的大小，

然後透過bindTexture設定texture，這邊有個跟ios不一樣的地方，
就是ios分成三個data傳入，分別對應Y,U,V，
而這邊只傳入一個g_buffer，這個g_buffer是內含ＹＵＶ三組data，
所以透過 buffer, buffer + width * height, buffer + width * height * 5 / 4，就可以讀到所需要的data。
Y size: width*height 
U size: width/2*height/2 
V szie: width/2*height/2




static GLuint bindTexture(GLuint texture, const char *buffer, GLuint w , GLuint h)
{
    glBindTexture ( GL_TEXTURE_2D, texture );
    checkGlError("glBindTexture");
    glTexImage2D ( GL_TEXTURE_2D, 0, GL_LUMINANCE, w, h, 0, GL_LUMINANCE, GL_UNSIGNED_BYTE, buffer);
    checkGlError("glTexImage2D");
    glTexParameteri ( GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR );
    checkGlError("glTexParameteri");
    glTexParameteri ( GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR );
    checkGlError("glTexParameteri");
    glTexParameteri ( GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE );
    checkGlError("glTexParameteri");
    glTexParameteri ( GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE );
    checkGlError("glTexParameteri");

    return texture;
}


這邊一樣bind ＹＵＶ textures，
並設定GL_TEXTURE_MIN_FILTER，GL_TEXTURE_MAG_FILTER ，GL_TEXTURE_WRAP_S，GL_TEXTURE_WRAP_T。





static void renderFrame()
{
    static GLfloat coordVertices[] = {
        0.0f, 1.0f,
        1.0f, 1.0f,
        0.0f,  0.0f,
        1.0f,  0.0f,
    };

    glClearColor(0.5f, 0.5f, 0.5f, 1);
    checkGlError("glClearColor");
    glClear(GL_COLOR_BUFFER_BIT);
    checkGlError("glClear");
    GLint tex_y = glGetUniformLocation(simpleProgram, "SamplerY");
    checkGlError("glGetUniformLocation");
    GLint tex_u = glGetUniformLocation(simpleProgram, "SamplerU");
    checkGlError("glGetUniformLocation");
    GLint tex_v = glGetUniformLocation(simpleProgram, "SamplerV");
    checkGlError("glGetUniformLocation");

    ATTRIB_VERTEX=glGetAttribLocation(simpleProgram,"vPosition");
    checkGlError("glBindAttribLocation");

    ATTRIB_TEXTURE=glGetAttribLocation(simpleProgram,"a_texCoord");
    checkGlError("glBindAttribLocation");

    glVertexAttribPointer(ATTRIB_VERTEX, 2, GL_FLOAT, 0, 0, squareVertices);
    checkGlError("glVertexAttribPointer");
    glEnableVertexAttribArray(ATTRIB_VERTEX);
    checkGlError("glEnableVertexAttribArray");

    glVertexAttribPointer(ATTRIB_TEXTURE, 2, GL_FLOAT, 0, 0, coordVertices);
    checkGlError("glVertexAttribPointer");
    glEnableVertexAttribArray(ATTRIB_TEXTURE);
    checkGlError("glEnableVertexAttribArray");

    glActiveTexture(GL_TEXTURE0);
    checkGlError("glActiveTexture");
    glBindTexture(GL_TEXTURE_2D, g_texYId);
    checkGlError("glBindTexture");
    glUniform1i(tex_y, 0);
    checkGlError("glUniform1i");

    glActiveTexture(GL_TEXTURE1);
    checkGlError("glActiveTexture");
    glBindTexture(GL_TEXTURE_2D, g_texUId);
    checkGlError("glBindTexture");
    glUniform1i(tex_u, 1);
    checkGlError("glUniform1i");

    glActiveTexture(GL_TEXTURE2);
    checkGlError("glActiveTexture");
    glBindTexture(GL_TEXTURE_2D, g_texVId);
    checkGlError("glBindTexture");
    glUniform1i(tex_v, 2);
    checkGlError("glUniform1i");

    glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
    checkGlError("glDrawArrays");
}



coordVertices跟ios一樣。
glClearColor(0.5f, 0.5f, 0.5f, 1);  顏色跟ios不同，不過這沒差，只是清除時的顏色而已。
checkGlError("glClearColor");

    glClear(GL_COLOR_BUFFER_BIT);
    checkGlError("glClear");
用上一行設定的清除顏色進行清除。

    GLint tex_y = glGetUniformLocation(simpleProgram, "SamplerY");
    checkGlError("glGetUniformLocation");
    GLint tex_u = glGetUniformLocation(simpleProgram, "SamplerU");
    checkGlError("glGetUniformLocation");
    GLint tex_v = glGetUniformLocation(simpleProgram, "SamplerV");
    checkGlError("glGetUniformLocation");
取得sampler YUV location。


    ATTRIB_VERTEX=glGetAttribLocation(simpleProgram,"vPosition");
    checkGlError("glBindAttribLocation");

    ATTRIB_TEXTURE=glGetAttribLocation(simpleProgram,"a_texCoord");
    checkGlError("glBindAttribLocation");
可以取得 attribute的location。


    glVertexAttribPointer(ATTRIB_VERTEX, 2, GL_FLOAT, 0, 0, squareVertices);
    checkGlError("glVertexAttribPointer");
    glEnableVertexAttribArray(ATTRIB_VERTEX);
    checkGlError("glEnableVertexAttribArray");

    glVertexAttribPointer(ATTRIB_TEXTURE, 2, GL_FLOAT, 0, 0, coordVertices);
    checkGlError("glVertexAttribPointer");
    glEnableVertexAttribArray(ATTRIB_TEXTURE);
    checkGlError("glEnableVertexAttribArray");
傳入data給attribute。


    glActiveTexture(GL_TEXTURE0);
    checkGlError("glActiveTexture");
    glBindTexture(GL_TEXTURE_2D, g_texYId);
    checkGlError("glBindTexture");
    glUniform1i(tex_y, 0);
    checkGlError("glUniform1i");

    glActiveTexture(GL_TEXTURE1);
    checkGlError("glActiveTexture");
    glBindTexture(GL_TEXTURE_2D, g_texUId);
    checkGlError("glBindTexture");
    glUniform1i(tex_u, 1);
    checkGlError("glUniform1i");

    glActiveTexture(GL_TEXTURE2);
    checkGlError("glActiveTexture");
    glBindTexture(GL_TEXTURE_2D, g_texVId);
    checkGlError("glBindTexture");
    glUniform1i(tex_v, 2);
    checkGlError("glUniform1i");
active三個textures（ＹＵＶ），並且傳入給uniform sampler2D。


    glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
    checkGlError("glDrawArrays");
繪製，關於glDrawArrays前面ios也有介紹過，是一樣的。



到這邊android ndk opengl render講完了，
下次會提到EGL，
這個EGL是用來連接view與opengl，其概念有點像SDL，在ios中也是用到EGL(ios叫EAGL)。
