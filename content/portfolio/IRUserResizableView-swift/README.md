---
title: "IRUserResizableView-swift"
date: 2021-04-25T13:22:14+08:00
description: "IRUserResizableView-swift is a powerful resizable view for iOS."
link: 'https://github.com/irons163/IRUserResizableView-swift'
layout: portfolio
draft: false
---

## Screenshots
![Demo](IRUserResizableView-swift/ScreenShots/demo1.png)

- IRUserResizableView-swift is a powerful resizable view for iOS.
- IRUserResizableView-swift is modeled after the resizable image view from the Pages iOS app. Any UIView can be provided as the content view for the IRUserResizableView-swift. As the view is respositioned and resized, setFrame: will be called on the content view accordingly.
- Objc version see here:[IRUserResizableView](https://github.com/irons163/IRUserResizableView)

## Features
- Resizable.

## Usage

### Basic
``` swift
import IRUserResizableView_swift

...

override func viewDidLoad() {
    super.viewDidLoad()

    let appFrame = UIScreen.main.bounds
    self.view = UIView.init(frame: appFrame);
    self.view.backgroundColor = .green;

    // (1) Create a user resizable view with a simple red background content view.
    let gripFrame = CGRect.init(x: 50, y: 50, width: 200, height: 150)
    let userResizableView = IRUserResizableView.init(frame: gripFrame)
    let contentView = UIView.init(frame: gripFrame);
    contentView.backgroundColor = .red;
    userResizableView.contentView = contentView;
    userResizableView.delegate = self;
    userResizableView.showEditingHandles()
    self.view.addSubview(userResizableView)
}
```

##### This project is inspired from [SPUserResizableView](https://github.com/spoletto/SPUserResizableView).
