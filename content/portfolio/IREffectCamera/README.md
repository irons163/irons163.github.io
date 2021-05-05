---
title: "IREffectCamera"
date: 2021-05-05T00:02:14+08:00
description: "IREffectCamera is a powerful camera view controller use face stickers and filters for iOS."
link: 'https://github.com/irons163/IREffectCamera'
layout: portfolio
draft: false
---

## Screenshots
| Demo Main Page | Enable Face Sticker |
|:---:|:---:|
|![Demo Main Page](IREffectCamera/ScreenShots/demo1.png)|![Enable Face Sticker](IREffectCamera/ScreenShots/demo2.png)| 
| Confirm | Custom filters |
|![Confirm](IREffectCamera/ScreenShots/demo3.png)|![Custom filters](IREffectCamera/ScreenShots/demo4.png)| 
| Custom filters | Update display view |
|![Custom filters](IREffectCamera/ScreenShots/demo5.png)|![Update display view](IREffectCamera/ScreenShots/demo6.png)| 

## Features
-  Camera basic functions.
-  Face stickers and filters

## Technologies
- Camera [IRCameraViewController](https://github.com/irons163/IRCameraViewController).
- Face Stickers [IRCameraSticker](https://github.com/irons163/IRCameraSticker)
- Filters [GPUImage](https://github.com/BradLarson/GPUImage)

## Install
### Git
- Git clone this project.
- Copy this project into your own project.
- Add the .xcodeproj into you  project and link it as embed framework.
#### Options
- You can remove the `demo` and `ScreenShots` folder.

### Cocoapods
- Add `pod 'IREffectCamera'`  in the `Podfile`
- `pod install`

## Usage

### Basic

#### Basic functions
- See [IRCameraViewController](https://github.com/irons163/IRCameraViewController).

#### Face Stickers
- See [IRCameraSticker](https://github.com/irons163/IRCameraSticker).

```obj-c
#import <IRCameraSticker/IRCameraSticker.h>
#import <IRCameraSticker/IRCameraStickerFilter.h>
#import <IRCameraSticker/IRCameraStickersManager.h>

...

- (IBAction)faceStickerTapped {
    [_camera displayFaceSticker];
}
```

### Advanced settings

Custom image filters(You can see how GPUImage work in the demo project):

- Return `YES` by `customizePhotoProcessingView` in the `IRCameraDelegate` to disable the default filters
```obj-c

#pragma mark - IRCameraDelegate

- (BOOL)customizePhotoProcessingView {
    return YES;
}

```

- Deal with the image by your own way:
```obj-c

#import <GPUImage/GPUImage.h>

- (UIImage *)imageWithSketchFilter:(UIImage *)originImage {
    GPUImageFilter *imageFilter = [[GPUImageSketchFilter alloc] init];
    return [imageFilter imageByFilteringImage:originImage];
}

```



