title: Media Apps 
categories: Android
tags: [Android]
---

# Media Apps Overview

## Player and UI
一个多媒体程序通常包含两个部分：

- player, 获取媒体数据播放
- ui, 用来控制player以及现实媒体信息

Android开发播放器

- 使用MediaPlayer类
- 使用开源ExoPlayer库

## Media session and media controller
![](/img/android/media_apps/MediaSessionMediaController.png)

### Media session
media session与player进行通信。

### Media controller

## Video apps versus audio apps

### Video app
视频app通常实现为一个activity
![](/img/android/media_apps/Video_app.png)

### Audio app
可以用两个component来实现audio app, 一个activity实现UI，一个service用来实现player
![](/img/android/media_apps/Audio_app.png)

## Media apps and the Android audio infrastructure
## The media-compat library

# Working with a Media Session
media sessions的生命周期和它管理的player一致。

## Initialize the media session
- 设置flags，来使media session能接收到media controller和media button的callback
- 创建一个PlaybackStateCompat并分配到media session
- 创建MediaSessionCompat.Callback实例并分配到media session

## Maintain the playback state and metadata
有两个类用来描述media session的状态.

1. PlaybackStateCompat: 用来描述当前player的operational状态。
2. MediaMetadataCompat: 用来描述播放的媒体的信息。

### States and errors

## Media session lock screens

## Media session callbacks


### 参考
[Media Apps](https://developer.android.com/guide/topics/media-apps/index.html)
