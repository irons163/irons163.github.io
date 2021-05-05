---
title: "IRSingleButtonGroup"
date: 2021-01-21T13:22:14+08:00
description: "IRSingleButtonGroup is a powerful buttons group framework for iOS."
link: 'https://github.com/irons163/IRSingleButtonGroup'
layout: portfolio
draft: false
---

## Screenshots
![Demo](IRSingleButtonGroup/ScreenShots/demo1.png)

- See swift version in here: [IRSingleButtonGroup-swift](https://github.com/irons163/IRSingleButtonGroup-swift).

## Features

- Single Button Selection.
- Single Button Selection Demo: Deselect able.
- Multi Buttons Selection.

## Install
### Cocoapods
- Add `pod 'IRSingleButtonGroup'`  in the `Podfile`
- `pod install`

## Usage

- more examples in the demo applications.

### Basic

```obj-c

IRSingleButtonGroup* singleButtonGroup = [[IRSingleButtonGroup alloc] init];
singleButtonGroup.buttons = @[self.button1, self.button2, self.button3];
singleButtonGroup.delegate = self;

#pragma mark - SingleButtonGroupDelegate
- (void)didSelectedButton:(UIButton *)button {
    NSLog(@"Button%ld", button.tag);
}

- (void)didDeselectedButton:(UIButton *)button {
    NSLog(@"Button%ld", button.tag);
}
```
