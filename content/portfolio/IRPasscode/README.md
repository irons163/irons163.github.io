---
title: "IRPasscode"
date: 2021-04-21T13:22:14+08:00
description: "IRPasscode is a powerful passcode for iOS."
link: 'https://github.com/irons163/IRPasscode'
layout: portfolio
draft: false
---

## Screenshots
| Demo | Passcode Settings |
|:---:|:---:|
| ![Demo](IRPasscode/ScreenShots/demo1.png) | ![Passcode Settings](IRPasscode/ScreenShots/demo2.png) |
| Set Passcode  | Confirm Passcode |
| ![Set Passcode](IRPasscode/ScreenShots/demo3.png) | ![Confirm Passcode](IRPasscode/ScreenShots/demo4.png) |
| Confirm Passcode Fail | Change Passcode |
| ![Confirm Passcode Fail](IRPasscode/ScreenShots/demo5.png) | ![Change Passcode](IRPasscode/ScreenShots/demo6.png) |
| Unlock Passcode | Demo Private Data |
| ![Unlock Passcode](IRPasscode/ScreenShots/demo7.png) | ![Demo Private Data](IRPasscode/ScreenShots/demo8.png) |

## Features
- 4 Pin support.
- FingerPrint support.
- High Security - KeyChain support.

## Install
### Git
- Git clone this project.
- Copy this project into your own project.
- Add the .xcodeproj into you  project and link it as embed framework.
#### Options
- You can remove the `demo` and `ScreenShots` folder.

### Cocoapods
- Add `pod 'IRPasscode'`  in the `Podfile`
- `pod install`

## Usage

### Basic
- Open `Passcode Setting Page`.
```obj-c
#import <IRPasscode/IRPasscode.h>

NSBundle *xibBundle = [NSBundle bundleForClass:[IRPasscodeLockSettingViewController class]];
IRPasscodeLockSettingViewController *vc = [[IRPasscodeLockSettingViewController alloc] initWithNibName:@"IRPasscodeLockSettingViewController" bundle:xibBundle];
[self.navigationController pushViewController:vc animated:YES];
```

- Open `Passcode verify page`.
```obj-c
if ([IRSecurityPinManager sharedInstance].pinCode)
    [[IRSecurityPinManager sharedInstance] presentSecurityPinViewControllerForUnlockWithAnimated:YES completion:nil result:nil];
```
