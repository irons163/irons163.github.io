---
title: "kxmovie解析（一）"
date: 2021-02-12T15:14:10+08:00
tags : [ "FFMpeg", "OpenGL", "iOS"]
layout: post
type:  "post"
draft: false
---

## 對kxmovie進行解析

- 解析１：

1. CAEAGLLayer

```objc
/* Set CAEAGLLayer */
CAEAGLLayer *eaglLayer = (CAEAGLLayer*) self.layer;
eaglLayer.contentsScale = [[UIScreen mainScreen] scale];
eaglLayer.opaque = YES;
eaglLayer.drawableProperties = [NSDictionary dictionaryWithObjectsAndKeys:
[NSNumber numberWithBool:FALSE], kEAGLDrawablePropertyRetainedBacking,
                                        kEAGLColorFormatRGBA8, kEAGLDrawablePropertyColorFormat,nil];
_context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
```
-----------------------------------------------------------------------------------------------------------------------
從`UIView`取得`CAEAGLLayer`，對`CAEAGLLayer`進行設置。
其中`eaglLayer.contentsScale = [[UIScreen mainScreen] scale];`
讓畫面可以scale，如此畫面才會清晰。

`[[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];`
可以選擇OpenGL版本：
```objc
typedef NS_ENUM(NSUInteger, EAGLRenderingAPI)
{
kEAGLRenderingAPIOpenGLES1 = 1,
kEAGLRenderingAPIOpenGLES2 = 2,
kEAGLRenderingAPIOpenGLES3 = 3,
};
```
2. Gen and bind buffers

```objc
/* Attaches an EAGLDrawable as storage for the OpenGL ES renderbuffer object bound to GL_RENDERBUFFER */
glGenFramebuffers(1, &_framebuffer);
glGenRenderbuffers(1, &_renderbuffer);
glBindFramebuffer(GL_FRAMEBUFFER, _framebuffer);
glBindRenderbuffer(GL_RENDERBUFFER, _renderbuffer);
[_context renderbufferStorage:GL_RENDERBUFFER fromDrawable:(CAEAGLLayer*)self.layer];
glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_WIDTH, &_backingWidth);
glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_HEIGHT, &_backingHeight);
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_RENDERBUFFER, _renderbuffer);
```
-----------------------------------------------------------------------------------------------------------------------

一開始先創建FrameBuffer跟RenderBuffer，
FrameBuffer：用於放置decode完後的Frame等待繪製。
RenderBuffer：用於繪製。
FrameBuffer跟RenderBuffer是可以複數個的，
建立多個FrameBuffer可以達成多緩衝繪圖機制（off screen），
例如Android SurfaceView就是雙緩衝繪圖。
緩衝種類：`GL_COLOR_ATTACHMENT0`（顏色緩衝），`GL_DEPTH_ATTACHMENT`（深度緩衝），`GL_STENCIL_ATTACHMENT`（模板緩衝）

-----------------------------------------------------------------------------------------------------------------------
ＷＩＫＩ介紹：
模版緩衝（stencil buffer）或印模緩衝，是在OpenGL三維繪圖等計算機圖像硬體中常見的除顏色緩衝、像素緩衝、深度緩衝之外另一種數據緩衝。詞源模版（stencil）是指一種印刷技術，通常以蠟紙或者鋼板印刷文字或圖形；區別於模板（template），用木板為外形修剪的依據來複製形狀；模版（stencil）是指印模，而模板（template）主要是指形模。模版緩衝是以像素為單位的，整數數值的緩衝，通常給每個像素分配一個字節長度的數值。深度緩衝與模版緩衝經常在圖形硬體的隨機存取記憶體（RAM）中分享相同的區域。

最簡單的情況，模版緩衝被用於限制渲染的區域。更多高級應用會利用深度緩衝與模版緩衝的在圖形渲染流水線中的強關聯關係。例如，模版的數值可以按每個像素是否通過深度測試來自動增加或減少。

簡單組合使用深度測試與模版修飾符可以使得大量的本來需要多次渲染過程的效果（例如陰影、外形的繪製或複合幾何圖元(Geometric primitive)的交叉部份的高光處理）可以簡單實現，因此減輕了圖形硬體的負擔。

最典型的應用是給三維圖像加陰影。也用於平面反射。
-----------------------------------------------------------------------------------------------------------------------
在這裡player只需要顏色緩衝，而且也沒有實作多緩衝繪圖機制，
所以創建`FrameBuffer`跟`RenderBuffer`各一個即可。創建完後bind。
`glGenFramebuffers`, `glGenRenderbuffers`, `glBindFramebuffer`, `glBindRenderbuffer`。
```objc
[_context renderbufferStorage:GL_RENDERBUFFER fromDrawable:(CAEAGLLayer*)self.layer];
```
把`CAEAGLLaye`r當做`renderbufferStorage`附加給`GL_RENDERBUFFER`，而這裡的`GL_RENDERBUFFER`就是剛才創建並且bind上去的_renderbuffer。

```objc
glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_WIDTH, &_backingWidth);
glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_HEIGHT, &_backingHeight);
```
取得`GL_RENDERBUFFER`的寬高，因為剛才把`CAEAGLLayer`當做`renderbufferStorage`附加給`GL_RENDERBUFFER`，所以這裡可以取得`CAEAGLLayer`的寬高。

```objc
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_RENDERBUFFER, _renderbuffer);
```
將`FrameBuffer`跟`RenderBuffer`關聯。這裡是用於顏色緩衝（`GL_COLOR_ATTACHMENT0`）。
