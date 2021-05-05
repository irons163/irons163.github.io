---
title: "IRDragTimeline"
date: 2021-04-22T13:22:14+08:00
description: "IRDragTimeline is a powerful drag-and-drop expandable tableview for iOS."
link: 'https://github.com/irons163/IRDragTimeline'
layout: portfolio
draft: false
---

## Screenshots
| Demo1 | Demo2 |
|:---:|:---:|
|![Demo](IRDragTimeline/ScreenShots/demo1.png)|![Demo](IRDragTimeline/ScreenShots/demo2.png)|

## Features
- Expandable tableview with nested.
- Able to drag-and-drop.
- Has a demo.

## Install
### Git
- Git clone this project.
- Copy `Class` amd `Utility` folders into your own project.
- See how to use it in `README` or `ViewController.m`.

### Cocoapods
- Not support yet.

## Usage

### Basic
- Set Branch
``` objective-c
#import "TimelineManager.h"

...

Branch *branch = [[TimelineManager sharedInstance] branchFromClientJourneyData:_clientJourneyData];
branch.tableView = self.timelineTableView;
branch.delegate = self;

[self.timelineTableView reloadDataWithCompletion:^{
    [self.delegate didUpdate:nil];
}];
```

- Set Delegate
``` objective-c
#import "TimelineManager.h"

...

@interface MonitorClientsDetailTimelineTableViewCell ()<BranchDelegate>

...

- (void)willUpdate:(NSNumber *)pos {
    [self.delegate willUpdate:pos];
}

- (void)didUpdate:(NSNumber *)pos {
    [self layoutIfNeeded];
    
    [self.delegate didUpdate:pos];
}

```

