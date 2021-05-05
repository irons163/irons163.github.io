---
title: "IRPlayerUIShell "
date: 2021-04-27T13:22:14+08:00
description: "IRPlayerUIShell is a powerful UI Shell framework for the video player(IRPlayer) for iOS."
link: 'https://github.com/irons163/IRPlayerUIShell '
layout: portfolio
draft: false
---

## Screenshots
|Play|Seek|
|:---:|:---:|
|![Demo](IRPlayerUIShell/ScreenShots/demo1.png)|![Demo](IRPlayerUIShell/ScreenShots/demo2.png)|
|Volume|Brightness|
|![Demo](IRPlayerUIShell/ScreenShots/demo3.png)|![Demo](IRPlayerUIShell/ScreenShots/demo4.png)|
|Full Screen|Lock Screen|
|![Demo](IRPlayerUIShell/ScreenShots/demo5.png)|![Demo](IRPlayerUIShell/ScreenShots/demo6.png)|

## Features
- Support customize UI for [IRPlayer](https://github.com/irons163/IRPlayer).
- Support some media controllers.
    - Seek Bar
    - Brightness
    - Volume
    - Full Screen
- Has a Demo

## Future
- Support Multi video player in one page(UITableView, UICollectionView, etc).
- More powerful custom UI design.
- Make a framework for this project.

## Usage

### Basic
- See `IRPlayerUIShellViewController` for demo.

- Create a [IRPlayer](https://github.com/irons163/IRPlayer) instance.
```obj-c
self.playerImp = [IRPlayerImp player];
self.playerImp.decoder = [IRPlayerDecoder FFmpegDecoder];
[self.playerImp replaceVideoWithURL:VIDEO_URL];
```

- Create a IRPlayerController instance, set the player and containerView while init, and then set the controlView.
```obj-c
self.player = [IRPlayerController playerWithPlayerManager:self.playerImp containerView:self.containerView];
self.player.controlView = self.controlView;
```

- Set the video urls, and then the first video will play!
```obj-c
self.player.assetURLs = self.assetURLs;
```

### Advanced settings
- If app is in the background, still play continue.
```obj-c
self.player.pauseWhenAppResignActive = NO;
```

- Listener for orientation change.
```obj-c
@weakify(self)
self.player.orientationWillChange = ^(IRPlayerController * _Nonnull player, BOOL isFullScreen) {
    @strongify(self)
    [self setNeedsStatusBarAppearanceUpdate];
};
```

- Listener for player go end.
```obj-c
    self.player.playerDidToEnd = ^(id  _Nonnull asset) {
        @strongify(self)
        [self.player.currentPlayerManager pause];
        [self.player.currentPlayerManager play];
        
        [self.player playTheNext];
        if (!self.player.isLastAssetURL) {
        NSString *title = [NSString stringWithFormat:@"title:%zd",self.player.currentPlayIndex];
            [self.controlView showTitle:title coverURLString:kVideoCover fullScreenMode:IRFullScreenModeLandscape];
        } else {
            [self.player stop];
        }
    };
```

##### This project is inspired from [ZFPlayer](https://github.com/renzifeng/ZFPlayer).
