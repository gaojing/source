title: 深入理解Android内核设计思想 第十三章 音频系统
categories: 深入理解Android内核设计思想
tags: [深入理解Android内核设计思想]
---

## 13.2 音频框架
### 13.2.2 TinyAlsa
代码位于external/tinyalsa
### 13.2.3 android系统中的音频框架
2. framework   
MediaPlayer和MediaRecorder是开发音频相关产品使用最广泛的两个类。还有专门管理音频的类AudioTrack和AudioRecorder,MediaPlayerService内部的音频实现就是通过同门实现的。还有AudioManager,AudioService和AudioSystem类。
3. Library   
源码frameworks/av/media/libmedia。AudioTrack,AudioRecorder,MediaPlayer和MediaRecorder在底层库都有对应的类。   
还有系统服务，AudioFlinger和AudioPolicyService。源码在frameworks/av/services/audioflinger目录中，生成的库名libaudioflinger。   
另外重要的系统服务是MediaPlayerService，在frameworks/av/media/libmediaplayerservice中。
4. HAL   
音频方面的硬件抽象层的核心是AudioFlinger和AudioPolicyService。

## 13.3 音频系统的核心 AudioFlinger
### 13.3.1 AudioFlinger服务的启动和运行
Android系统服务分为Java层和Native层的system service，AudioFlinger和SurfaceFlinger属于后者。AudioFlinger利用一个linux程序来间接创建。