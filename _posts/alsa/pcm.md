title: PCM (digital audio) interface
categories: ALSA
tags: [ALSA]
---

# General overview
ALSA使用ring buffer来储存outgoing(playback)和incoming（capture, record)samples。

# transfer methods in unix
数据通过标准的I/O系统调用或者event waiting函数(poll or select)。

## standard I/O transfer
read或者write函数，blocked或者non-blocked。

## event waiting routines
poll或者select。

## Asynchronous notification
通过SIGIO来做异步通知。

# Blocked和non-blocked open

# Asynchronous mode

# Handshake between application and library
ALSA PCM API定义了一些状态，来表示应用和library通信的阶段，snd_pcm_state()返回状态。

- SND\_PCM\_STATE\_OPEN   
PCM设备处理打开状态。
- SND\_PCM\_STATE\_SETUP   
PCM设备已经接收了参数，等待snd_pcm_prepare()调用来准备设备。
- SND\_PCM\_STATE\_PREPARED   
   


[1](http://alsa-project.org/alsa-doc/alsa-lib/pcm.html)