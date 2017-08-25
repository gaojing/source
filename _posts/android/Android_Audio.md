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

### File format and location
新的audio policy配置文件在/system/etc的audio_policy_configuration.xml文件中。   
[配置例子](https://source.android.com/reference/hal/)   
最上层结构对应HAL硬件模块，每个模块包含mix ports，device ports和routes列表。  
- Mix ports:描述了streams的配置。   
- Device ports:
- Routes