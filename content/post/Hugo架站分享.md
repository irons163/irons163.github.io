---
title: "Hugo架站分享"
date: 2021-04-25T16:17:55+08:00
tags : [ "Hugo" ]
layout: post
type:  "post"
draft: false
---

如果你想架站，現在早已經有一推網站可以讓你註冊個人部落格，
分享文章以及點點滴滴。

但是你是否曾經有覺得不安全過？畢竟你辛苦完成的照片及文章都在別人的伺服器上，
萬一網站倒閉、硬碟毀損或被駭客入侵，就有可能什麼都沒有了！

幸運的是，現在架設一個簡單的靜態網站一點都不難。
而且你可以保留全部的檔案在自家的電腦裡，若搭配本篇會一起介紹的Github網站，
等於又多了一個備份在雲端中，安全性大大提升。


現在開始正式介紹今天的主角：Hugo


## Install

我的環境是MacOS，所以以下都用MacOS來解說，若是使用其他作業系統的朋友，可以透過連結去看官方安裝手冊：[Hugo Install](https://gohugo.io/getting-started/installing/)。

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Then

```
brew install hugo
```
