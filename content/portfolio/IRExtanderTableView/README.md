---
title: "IRExpandableTableView"
date: 2021-04-24T13:22:14+08:00
description: "IRExpandableTableView is a powerful expandable tableview for iOS."
link: 'https://github.com/irons163/IRExpandableTableView'
layout: portfolio
draft: false
---

## Screenshots
| Demo1 | Demo2 |
|:---:|:---:|
|![Demo](IRExtanderTableView/ScreenShots/demo1.png)|![Demo](IRExtanderTableView/ScreenShots/demo2.png)|

## Features
- Expandable tableview with nested.
- Easy to customize. Easy to management.
- Nice stucture.
- Has a demo project.

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

