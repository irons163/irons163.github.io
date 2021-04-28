---
title: "IRMusicPlayer"
date: 2021-04-27T13:22:14+08:00
description: "IRMusicPlayer is a powerful music player for iOS."
link: 'https://github.com/irons163/IRMusicPlayer'
layout: portfolio
draft: false
---

## Screenshots
![Demo](IRMusicPlayer/ScreenShots/demo1.png)

## Features
- Support online/local play.
- Support to show music cover.
- Support randon mode.
- Support repeat modes: repeat all musics once, repeat current music forever, repeat all musics forever.

## Future
- Support background play.

## Usage

### Basic
```obj-c
@import IRMusicPlayer;

MusicPlayerViewController *vc = [[MusicPlayerViewController alloc] initWithNibName:@"MusicPlayerViewController" bundle:xibBundle];

[vc.musicListArray addObject:@{@"musicAddress": [[NSBundle mainBundle] pathForResource:@"1" ofType:@"mp3"]}];
[vc.musicListArray addObject:@{@"musicAddress": [[NSBundle mainBundle] pathForResource:@"2" ofType:@"mp3"]}];
[vc.musicListArray addObject:@{@"musicAddress": [[NSBundle mainBundle] pathForResource:@"3" ofType:@"mp3"]}];

[self presentViewController:vc animated:YES completion:nil];
```
