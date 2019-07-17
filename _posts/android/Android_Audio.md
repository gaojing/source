title: Android_Audio
categories: Android
tags: [Android]
---

# Table of Contents
- [1. Overview](#section1)
	- [1.1 architecture](#section1.1)
- [2. Implementation](#section2)
	- [2.1 Overview](#section2.1)
		- [2.1.1 Implementing the audio HAL](#section2.1.1)
		- [2.1.2 Audio Header file](#section2.1.2)
	- [2.2 Configuring Audio Policies](#section2.2)
		- [2.2.1 File format and location](#section2.2.1)

<a name="section1"></a>
# 1. Overview
Android audio HAL 连接android.media中音频相关的API和底层的audio驱动和硬件。

<a name="section1.1"></a>
## 1.1 architecture
![](/images/ape_fwk_audio.png)

<a name="section2"></a>
# 2. Implementation

<a name="section2.1"></a>
## 2.1 Overview

<a name="section2.1.1"></a>
### 2.1.1 Implementing the audio HAL
Audio HAL由下面接口组成
   
- hardware/libhardware/include/hardware/audio.h:一个audio设备的主要函数   
- hardware/libhardware/include/hardware/audio_effect.h：

<a name="section2.1.2"></a>
### 2.1.2 Audio Header file
Android6.0以后system/media/audio/include/system/audio.h

<a name="section2.2"></a>
## 2.2 Configuring Audio Policies

<a name="section2.2.1"></a>
### 2.2.1 File format and location
新的audio policy配置文件在/system/etc的audio_policy_configuration.xml文件中。   