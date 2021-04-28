---
title: "EGL介紹"
date: 2021-02-15T15:26:18+08:00
tags : [ "OpenGL", "iOS"]
layout: post
type:  "post"
draft: false
---

現在要介紹ＥＧＬ。


>WIKI介紹：
EGL is an interface between Khronos rendering APIs (such as OpenGL, OpenGL ES or OpenVG) and the underlying native platformwindowing system. EGL handles graphics context management, surface/buffer binding, rendering synchronization, and enables "high-performance, accelerated, mixed-mode 2D and 3D rendering using other Khronos APIs."[2] EGL is managed by the non-profit technology consortium Khronos Group.

-----------------------------------
EGL 介紹

EGL 是為 OpenGL ES 提供平台獨立性而設計， 是 OpenGL ES 和底層 Native 平台視窗系統之間的接口。 OpenGL ES 為附加功能和可能的平台特性開發提供了擴展機制，但仍然需要一個可以讓 OpenGL ES 和本地視窗系統交互且平台無關的層。 OpenGL ES 本質上是一個圖形渲染管線的狀態機，而 EGL 則是用於監控這些狀態以及維護 Frame buffer 和其他渲染 Surface 的外部層。

EGL 視窗設計是基於人們熟悉的用於 Microsoft Windows （ WGL ）和 UNIX （ GLX ）上的 OpenGL 的 Native 接口，與後者比較接近。 OpenGL ES 圖形管線的狀態被存儲於 EGL 管理的一個 Context 中。 Frame Buffers 和其他繪製 Surfaces 通過 EGL API 創建、管理和銷毀。 EGL 同時也控制和提供了對設備顯示和可能的設備渲染配置的訪問。



初始化EGL
OpenGL ES是一個平台中立的圖形庫，在它能夠工作之前，需要與一個實際的窗口系統關聯起來，這與OpenGL是一樣的。但不一樣的是，這部份工作有標準，這個標 準就是EGL。而OpenGL時代在不同平台上有不同的機制以關聯窗口系統，在Windows上是wgl，在X-Window上是xgl，在Apple OS上是agl(或EAGL)等。 EGL的工作方式和部份術語都接近於xgl。

OpenGL ES的初始化過程如下圖所示意：

Display → Config → Surface
                         ↑
                       Context
                         ↑
Application → OpenGL Command

 

1. 獲取Display。
Display 代表顯示器，在有些系統上可以有多個顯示器，也就會有多個Display。獲得Display要調用EGLboolean eglGetDisplay(NativeDisplay dpy)，參數一般為 EGL_DEFAULT_DISPLAY 。該參數實際的意義是平台實現相關的，在X-Window下是XDisplay ID，在MS Windows下是Window DC。

2. 初始化egl。
調用 EGLboolean eglInitialize(EGLDisplay dpy, EGLint *major, EGLint *minor)，該函數會進行一些內部初始化工作，並傳回EGL版本號(major.minor)。

3. 選擇Config。
所 為Config實際指的是FrameBuffer的參數，在MS Windows下對應於PixelFormat，在X-Window下對應Visual。一般用EGLboolean eglChooseConfig(EGLDisplay dpy, const EGLint * attr_list, EGLConfig * config, EGLint config_size, EGLint *num_config)，其中attr_list是以EGL_NONE結束的參數數組，通常以id,value依次存放，對於個別標識性的屬性可以只有id，沒有value。另一個辦法是用EGLboolean eglGetConfigs(EGLDisplay dpy, EGLConfig * config, EGLint config_size, EGLint *num_config) 來獲得所有config。這兩個函數都會返回不多於config_size個Config，結果保存在config[]中，系統的總Config個數保存 在num_config中。可以利用eglGetConfig()中間兩個參數為0來查詢系統支持的Config總個數。
Config有眾多的Attribute，這些Attribute決定FrameBuffer的格式和能力，通過eglGetConfigAttrib ()來讀取，但不能修改。

4. 構造Surface。
Surface 實際上就是一個FrameBuffer，通過 EGLSurface eglCreateWindowSurface(EGLDisplay dpy, EGLConfig confg, NativeWindow win, EGLint *cfg_attr) 來創建一個可實際顯示的Surface。系統通常還支持另外兩種Surface：PixmapSurface和PBufferSurface，這兩種都不是可顯示的Surface，PixmapSurface是保存在系統內存中的位圖，PBuffer則是保存在顯存中的幀 。
Surface 也有一些attribute，基本上都可以故名思意， EGL_HEIGHT EGL_WIDTH EGL_LARGEST_PBUFFER EGL_TEXTURE_FORMAT EGL_TEXTURE_TARGET EGL_MIPMAP_TEXTURE EGL_MIPMAP_LEVEL，通過eglSurfaceAttrib()設置、eglQuerySurface()讀取。

5. 創建Context。
OpenGL 的pipeline從程序的角度看就是一個狀態機，有當前的顏色、紋理坐標、變換矩陣、絢染模式等一大堆狀態，這些狀態作用於程序提交的頂點坐標等圖元從而形成幀緩衝內的像素。在OpenGL的編程接口中，Context就代表這個狀態機，程序的主要工作就是向Context提供圖元、設置狀 態，偶爾也從Context裡獲取一些信息。
用EGLContext eglCreateContext(EGLDisplay dpy, EGLSurface write, EGLSurface read, EGLContext * share_list)來創建一個Context。

6. 繪製。
應用程序通過OpenGL API進行繪製，一幀完成之後，調用eglSwapBuffers(EGLDisplay dpy, EGLContext ctx) 來顯示。
----------------------------

EGL加載OpenGL ES庫

涉及的庫

首先，由於涉及的庫較多，先列出來(高通平台，原生的只有前4個)，

//算是android中的egl庫，用來加載具體的實現
`system\lib\libEGL.so`
//opengl具體實現的wrapper
`system\lib\libGLESv1_CM.so`
`system\lib\libGLESv2.so`
//opengl軟件實現，即agl
`system\lib\egl\libGLES_android.so`
//egl的實現
`system\vendor\lib\egl\libEGL_adreno.so`
//opengl硬件實現
`system\vendor\lib\egl\libGLESv1_CM_adreno.so`
`system\vendor\lib\egl\libGLESv2_adreno.so`

這邊只有前面的3到4個是必要的，其他都是各家手機自己的相關so。
-----------------------------------------------

這邊開始從code介紹：

```objc
void MediaPlayer::EnableOpenGL(bool bEnableOpenGL)
{
    m_bEnableOpenGL = bEnableOpenGL;
    LOGI("MediaPlayer EnableOpenGL =>%d",bEnableOpenGL);
}
```

如果要啟動OpenGL，這邊先設為true，即啟動opengl。
這邊也可以改成用ifdef來啟動。



```objc
    colorFormat = PIX_FMT_YUV420P;
    colorByteNumber = 4;

    _context=EGL_NO_CONTEXT;
    glbuffer = NULL;
```

設colorFormat  為PIX_FMT_YUV420P
colorByteNumber =4
```objc
    _context設為EGL_NO_CONTEXT; //初始
    glbuffer = NULL ; //初始
```

```objc
else if(colorFormat == PIX_FMT_YUV420P) {
int pixelnumber = surfaceWidth* surfaceHeight;
int i;
uint32_t *pixel = (uint32_t*)buffer.bits;
for(i = 0; i < pixelnumber; i++)
pixel[i] = 0xff000000;
}
```

如果是PIX_FMT_YUV420P，一開始時buffer先放入預設值。


```objc
bool MediaPlayer::updateRenderSurface(JNIEnv* env, jobject self, jobject obj, int bn, int w, int h)
{
    ......

    if(sWindow != NULL)
        ANativeWindow_release(sWindow);

    ......

    if(_context!=EGL_NO_CONTEXT)
    destroy();

    return (sWindow != NULL || obj == NULL);
}
```

這個很關鍵，就是在surfaceview 變動時，這邊EGL context要destroy();。
因為surfaceview會產生一個新的，window也會需要realse再create一個新的。

```objc
if(_context==EGL_NO_CONTEXT){
    const EGLint attribs[] = {

    EGL_RENDERABLE_TYPE, EGL_OPENGL_ES2_BIT,

            EGL_BLUE_SIZE, 8,
            EGL_GREEN_SIZE, 8,
            EGL_RED_SIZE, 8,
            EGL_ALPHA_SIZE, 0,
            EGL_DEPTH_SIZE, 0,
            EGL_STENCIL_SIZE, 0,
            EGL_NONE
        };

    const EGLint contextAttribs[] = {
    EGL_CONTEXT_CLIENT_VERSION, 2, EGL_NONE
    };
        EGLDisplay display;
        EGLConfig config;
        EGLint numConfigs;
        EGLint format;
        EGLSurface surface;
        EGLContext context = EGL_NO_CONTEXT;
        EGLint width;
        EGLint height;
```

attribs[] 定義EGL的參數：
EGL_RENDERABLE_TYPE, EGL_OPENGL_ES2_BIT,
EGL_BLUE_SIZE, 8,
EGL_GREEN_SIZE, 8,
EGL_RED_SIZE, 8,
EGL_ALPHA_SIZE, 0,
EGL_DEPTH_SIZE, 0,
EGL_STENCIL_SIZE, 0,
EGL_NONE
最後一個一定要是EGL_NONE，
其他則是兩個為一組的數據：
EGL_RENDERABLE_TYPE代表使用版本設為OpenGL ES2，
EGL_BLUE_SIZE設為8，green and red the same。
所以就是argb = 0888。
DEPTH與STENCIL不需要。


contextAttribs：
EGL_CONTEXT_CLIENT_VERSION, 2, EGL_NONE
一樣最後一個一定要是EGL_NONE，
EGL_CONTEXT_CLIENT_VERSION代表EGL的版本設為2。

```objc
        EGLDisplay display;
        EGLConfig config;
        EGLint numConfigs;
        EGLint format;
        EGLSurface surface;
        EGLContext context = EGL_NO_CONTEXT;
        EGLint width;
        EGLint height;
```
定義一些參數。

```objc
        LOGI("Init EGL!");

        if ((display = eglGetDisplay(EGL_DEFAULT_DISPLAY)) == EGL_NO_DISPLAY) {
        LOGE("eglGetDisplay() returned error %d", eglGetError());
                return false;
            }
            if (!eglInitialize(display, 0, 0)) {
            LOGE("eglInitialize() returned error %d", eglGetError());
                return false;
            }

            if (!eglChooseConfig(display, attribs, &config, 1, &numConfigs)) {
            LOGE("eglChooseConfig() returned error %d", eglGetError());
                destroy();
                return false;
            }

            if (!eglGetConfigAttrib(display, config, EGL_NATIVE_VISUAL_ID, &format)) {
            LOGE("eglGetConfigAttrib() returned error %d", eglGetError());
                destroy();
                return false;
            }

            ANativeWindow_setBuffersGeometry(sWindow, surfaceWidth, surfaceHeight, format);

            if (!(surface = eglCreateWindowSurface(display, config, sWindow, 0))) {
            LOGE("eglCreateWindowSurface() returned error %d", eglGetError());
                destroy();
                return false;
            }

            if (!(context = eglCreateContext(display, config, EGL_NO_CONTEXT, contextAttribs))) {
            LOGE("eglCreateContext() returned error %d", eglGetError());
                destroy();
                return false;
            }

            if (!eglMakeCurrent(display, surface, surface, context)) {
            LOGE("eglMakeCurrent() returned error %d", eglGetError());
                destroy();
                return false;
            }
            eglQuerySurface(display, surface, EGL_WIDTH, &width);
            if (!eglQuerySurface(display, surface, EGL_WIDTH, &width) ||
                !eglQuerySurface(display, surface, EGL_HEIGHT, &height)) {
            LOGE("eglQuerySurface() returned error %d", eglGetError());
                destroy();
                return false;
            }

            _display = display;
            _surface = surface;
            _context = context;

            gl_initialize();
            gl_setbackingAndGLWH(surfaceWidth, surfaceHeight, m_nVideoWidth, m_nVideoHeight);
}
```

在來是要初始化EGL：
1. eglGetDisplay(EGL_DEFAULT_DISPLAY)
2. eglInitialize(display, 0, 0)
3. eglChooseConfig(display, attribs, &config, 1, &numConfigs)
4. eglGetConfigAttrib(display, config, EGL_NATIVE_VISUAL_ID, &format)
5. surface = eglCreateWindowSurface(display, config, sWindow, 0)
6. context = eglCreateContext(display, config, EGL_NO_CONTEXT, contextAttribs)
7. eglMakeCurrent(display, surface, surface, context)
8. eglQuerySurface(display, surface, EGL_WIDTH, &width)

只要有一個初始化失敗，印出失敗訊息後呼叫destroy();。


EGL初始化完後，初始化OpenGL：
`gl_initialize();`
`gl_setbackingAndGLWH(surfaceWidth, surfaceHeight, m_nVideoWidth, m_nVideoHeight);`


```objc
LOGI("OpenGL render.");

int i;
uint8_t* dst = (uint8_t*)glbuffer;
uint8_t* src = ffmpeg_fields.pFrame->data[0];
int width = ffmpeg_fields.pFrame->linesize[0] < m_nVideoWidth?
ffmpeg_fields.pFrame->linesize[0]: m_nVideoWidth;
// copy Y
for(i = 0; i < m_nVideoHeight; i++){
memcpy(dst, src, width);
dst += width;
src += ffmpeg_fields.pFrame->linesize[0];
}
// copy U
src = ffmpeg_fields.pFrame->data[1];
width = ffmpeg_fields.pFrame->linesize[1] < m_nVideoWidth / 2?
ffmpeg_fields.pFrame->linesize[1]: m_nVideoWidth / 2;
for(i = 0; i < m_nVideoHeight / 2; i++){
memcpy(dst, src, width);
dst += width;
src += ffmpeg_fields.pFrame->linesize[1];
}
// copy V
src = ffmpeg_fields.pFrame->data[2];
width = ffmpeg_fields.pFrame->linesize[2] < m_nVideoWidth / 2?
ffmpeg_fields.pFrame->linesize[2]: m_nVideoWidth / 2;
for(i = 0; i < m_nVideoHeight / 2; i++){
memcpy(dst, src, width);
dst += width;
src += ffmpeg_fields.pFrame->linesize[2];
}
//render
gl_set_framebuffer((const char*)glbuffer, glbuffsize , m_nVideoWidth, m_nVideoHeight);
gl_render_frame();

if (!eglSwapBuffers(_display, _surface)) {
                LOGE("eglSwapBuffers() returned error %d", eglGetError());
}
```

這段code是用來拷貝資料的，從ios porting過來的。

拷貝Y的值，透過ffmpeg_fields.pFrame->data[0];與ffmpeg_fields.pFrame->linesize[0];
進行拷貝。UV的值也是相同的處理方式。
要注意的YUV的大小：
Y:w*h
U:w/2*h/2
V:w/2*h/2

資料拷貝完後，傳入opengl.cpp：
gl_set_framebuffer((const char*)glbuffer, glbuffsize , m_nVideoWidth, m_nVideoHeight);
然後開始render: 
gl_render_frame();
--------------

最後render完用swapBuffers將frame buffer顯示於view上：

```objc
if (!eglSwapBuffers(_display, _surface)) {
                LOGE("eglSwapBuffers() returned error %d", eglGetError());
            }

if(m_bEnableOpenGL){
if(glbuffer != NULL)
free(glbuffer);
glbuffsize = m_nVideoWidth*m_nVideoHeight* 3 / 2;
glbuffer = (void*)malloc(sizeof(uint8_t)* glbuffsize);
}
```

`glbuffsize ＝ m_nVideoWidth*m_nVideoHeight* 3 / 2;`
因為
Y:w*h
U:w/2*h/2
V:w/2*h/2
所以YUV就是m_nVideoWidth*m_nVideoHeight* 3 / 2;
-----------


```objc
void MediaPlayer::destroy() {
    LOGI("Destroying context");

    eglMakeCurrent(_display, EGL_NO_SURFACE, EGL_NO_SURFACE, EGL_NO_CONTEXT);
    eglDestroyContext(_display, _context);
    eglDestroySurface(_display, _surface);
    eglTerminate(_display);

    _display = EGL_NO_DISPLAY;
    _surface = EGL_NO_SURFACE;
    _context = EGL_NO_CONTEXT;

    //return;
}
```

destroy是進行釋放的動作，這是很重要的。
------------

EGL介紹到此，也是一個滿複雜的東西，
幸好目前在media player上使用不需要太複雜，
一些參數設定好，初始化函式順序注意一下，
基本就可以運作。不過他的除錯也是有些麻煩，
有時有錯就要花不少時間解決。
