title: Android_Audio
categories: Android
tags: [Android]
---

# 1. Overview
Android audio HAL 连接android.media中音频相关的API和底层的audio驱动和硬件。

## architecture
![](/images/ape_fwk_audio.png)

# Implementation

## Overview

### Implementing the audio HAL
Audio HAL由下面接口组成
   
- hardware/libhardware/include/hardware/audio.h:一个audio设备的主要函数   
- hardware/libhardware/include/hardware/audio_effect.h：

### Audio Header file
Android6.0以后system/media/audio/include/system/audio.h

## Configuring Audio Policies
 