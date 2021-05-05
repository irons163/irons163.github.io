---
title: "IRImagePickerController"
date: 2021-04-23T13:22:14+08:00
description: "A clone of UIImagePickerController with multiple selection support."
link: 'https://github.com/irons163/IRImagePickerController'
layout: portfolio
draft: false
---

## Screenshots
| Demo1 | Demo2 |
|:---:|:---:|
|![screenshot01.png](IRImagePickerController/ScreenShots/screenshot01.png)|![screenshot02.png](IRImagePickerController/ScreenShots/screenshot02.png)|

## Features

- Allows multiple selection of photos and videos
- Fast and memory-efficient scrolling
- Provides similar user interface to the built-in image picker
- Customizable (grid size, navigation message, etc.)
- Supports both portrait mode and landscape mode
- Compatible with iPhone 6/6Plus, and iPad

## Cooperation
This project is nice to cooperate with [IRGallery](https://github.com/irons163/IRGallery).
See demo to learn how to use it.

![demo1.png](IRImagePickerController/ScreenShots/demo1.png)

## Example
```obj-c
    IRImagePickerController *imagePickerController = [IRImagePickerController new];
    imagePickerController.delegate = self;
    imagePickerController.allowsMultipleSelection = YES;
    imagePickerController.maximumNumberOfSelection = 6;
    imagePickerController.showsNumberOfSelectedAssets = YES;

    [self presentViewController:imagePickerController animated:YES completion:NULL];
```

##### This project is inspired from [QBImagePicker](https://github.com/questbeat/QBImagePicker).
