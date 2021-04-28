---
title: "FFMpeg and OpenGL心得（二）：FFMpeg優化策略"
date: 2021-01-27T23:29:50+08:00
tags : [ "FFMpeg", "OpenGL", "iOS"]
layout: post
type:  "post"
draft: false
---

FFMpeg優化策略：

１．CodecContext Optimization to Decode H264

https://ffmpeg.org/pipermail/libav-user/2013-February/003796.html

２．使用arm的組合語言neon來加速FFMpeg的處理：

https://trac.ffmpeg.org/attachment/ticket/1309/build-arm-neon.sh

ＳＤＬ： １．An ffmpeg and SDL Tutorial How to Write a Video Player in Less Than 1000 Lines http://dranger.com/ffmpeg/tutorial01.html （介紹的非常詳細，應該很有用） ２．Using OpenGL With SDL https://www.libsdl.org/release/SDL-1.2.15/docs/html/guidevideoopengl.html
