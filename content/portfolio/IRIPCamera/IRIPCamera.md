---
title: "IRIPCamera"
description: "IRIPCamera is a powerful URL/Rtsp/IPCam player/viewer for iOS."
date: 2021-04-26T15:08:42+08:00
link: 'https://project-p1.com'
screenshot: 'p1.png'
layout: 'portfolio'
featured: true
draft: true
---

![Build Status](https://img.shields.io/badge/build-%20passing%20-brightgreen.svg)
![Platform](https://img.shields.io/badge/Platform-%20iOS%20-blue.svg)

## How it works?
- Basically, it works by `IRPlayer` + `Live555` + iOS Native API.
    - [IRPlayer](https://github.com/irons163/IRPlayer)
    - [Live555](http://www.live555.com/)
- `Live555` can make a connection with a rtsp server/streaming.
- Decoding the frames by iOS VideoToolbox. The pixel format is NV12.
- `IRPlayer` is the video player which can receive the frames and play it.
    - If you are interested in this part, you can see how it works in `IRFFVideoInput`.
- Playing the audio by iOS AudioToolbox.

## Features
- Support Rtsp streaming.
- Support for customize connection to your streaming device or IPCam.
- Provide a demo that using `H264-RTSP-Server-iOS` as a RTSP IPCamera and `IRIPCamera` as a RTSP Player.
    - See [H264-RTSP-Server-iOS](https://github.com/irons163/H264-RTSP-Server-iOS).

## How the demo works?
- Prepare 2 iPhones, connecting them in the same network.
- Run [H264-RTSP-Server-iOS](https://github.com/irons163/H264-RTSP-Server-iOS) in an iPhone, it would show the local IP in the top of the screen.
- Run this project in the other iPhone, type the RTSP Url into the setting page.
- Enjoy your personal iPhoneCam : )

## Future
- Support Multi viewer.
- More powerful custom settings.

## Screenshots
|Display|Setting|
|---|---|
|![Demo](IRIPCamera/ScreenShots/demo1.png)|![Demo](IRIPCamera/ScreenShots/demo2.png)|
|![Demo](IRIPCamera/ScreenShots/demo3.png)|![Demo](IRIPCamera/ScreenShots/demo4.png)|
