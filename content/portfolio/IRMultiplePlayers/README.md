---
title: "IRMultiplePlayers"
date: 2021-04-22T13:22:14+08:00
description: "IRMultiplePlayers is a multiple video players  for iOS."
link: 'https://github.com/irons163/IRMultiplePlayers'
layout: portfolio
draft: false
---

## Screenshots
| Demo1 | Demo2 |
|:---:|:---:|
| ![Demo1](IRMultiplePlayers/ScreenShots/demo1.png) | ![Demo2](IRMultiplePlayers/ScreenShots/demo2.png) |
| Demo3 | |
| ![Demo3](IRMultiplePlayers/ScreenShots/demo3.png) | |


- Using the video player([IRPlayer](https://github.com/irons163/IRPlayer)).
- Using MVVM collectionView/tableView stucture([IRCollectionTableViewModel](https://github.com/irons163/IRCollectionTableViewModel)).

## Features
- Using MVVM stucture.
- Support TableView.
- Support CollectionView.

## Install
### Git
- Git clone this project.

## Usage

### Basic
- Set `reuseIdentifier` to every items in the collection view.
```obj-c
NSString *identifier = [NSString stringWithFormat:@"Identifier_%d-%d-%d", (int)indexPath.section, (int)indexPath.row, (int)indexPath.item];
[collectionView registerNib:[UINib nibWithNibName:CollectionViewCell.identifier bundle:nil] forCellWithReuseIdentifier:identifier];

CollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:identifier forIndexPath:indexPath];
```
