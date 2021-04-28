---
title: "IRPlayer"
date: 2021-04-28T13:22:14+08:00
description: "IRPlayer is a powerful video player framework for iOS."
link: 'https://github.com/irons163/IRPlayer'
screenshot: 
layout: portfolio
featured: false
draft: false
---

## Screenshots
| Normal | VR |
|:---:|:---:|
| ![Normal](IRPlayer/ScreenShots/demo1.PNG)  |  ![VR](IRPlayer/ScreenShots/demo2.PNG)  |
| VR Box| Fisheye 360 |
| ![VR Box](IRPlayer/ScreenShots/demo3.PNG) | ![Fisheye 360](IRPlayer/ScreenShots/demo4.PNG) |
| Panorama| Modes Selection |
| ![Panorama](IRPlayer/ScreenShots/demo5.PNG) | ![Modes Selection](IRPlayer/ScreenShots/demo6.PNG) |
| Multi Windows |  |
| ![Multi Windows](IRPlayer/ScreenShots/demo7.PNG)|  |

## Features

- Support Normal video mode.
- Support VR mode.
- Support VR Box mode.
- Support Fisheye mode.
    - Support Normal Fisheye mode.
    - Support Fisheye to Panorama mode.
    - Support Fisheye to Perspective mode.
- Support multi windows.
- Support multi modes selection.

- 0.3.6
    - Support set the specific renders to each mode.
    - Support custom video input(IRFFVideoInput). See what it works in [IRIPCamera](https://github.com/irons163/IRIPCamera).
    - Support custom display view(inherit IRGLView). See what it works in [IREffectPlayer](https://github.com/irons163/IREffectPlayer).

## Install
### Cocoapods
- Add `pod 'IRPlayer', '~> 0.3.2'`  in the `Podfile`
- `pod install`

## Usage

- more examples in the demo applications.

### Basic

```obj-c

self.player = [IRPlayerImp player];
[self.mainView insertSubview:self.player.view atIndex:0];

NSURL * normalVideo = [NSURL fileURLWithPath:[[NSBundle mainBundle] pathForResource:@"i-see-fire" ofType:@"mp4"]];
NSURL * vrVideo = [NSURL fileURLWithPath:[[NSBundle mainBundle] pathForResource:@"google-help-vr" ofType:@"mp4"]];
NSURL * fisheyeVideo = [NSURL fileURLWithPath:[[NSBundle mainBundle] pathForResource:@"fisheye-demo" ofType:@"mp4"]];

```

#### Set custom video source

- See what it works in [IRIPCamera](https://github.com/irons163/IRIPCamera).

``` obj-c
IRFFVideoInput *input = [[IRFFVideoInput alloc] init];
[self.player replaceVideoWithInput:input videoType:IRVideoTypeNormal];

...

IRFFAVYUVVideoFrame * yuvFrame = [[IRFFAVYUVVideoFrame alloc] init];
/*
setup the yuvFrame.
*/
[input updateFrame:frame];
```

## Extra
- Use IRPlayer to play video.
    - See demo.
- Use IRPlayer to make video player with custom UI.
    - See [IRPlayerUIShell](https://github.com/irons163/IRPlayerUIShell)
- Use IRPlayer to play IP Camera stream.
    - See [IRIPCamera](https://github.com/irons163/IRIPCamera)
- Use IRPlayer to make Screen Recoder.
    - See [IRRecoder](https://github.com/irons163/IRRecoder)
- Use IRPlayer to make RTMP streaming.
    - See [IRLiveKit](https://github.com/irons163/IRLiveKit)
- Use IRPlayer to make video player with effects .
    - See [IREffectPlayer](https://github.com/irons163/IREffectPlayer)
- Real Live player App.
    - See [IRLive](https://github.com/irons163/IRLive)

