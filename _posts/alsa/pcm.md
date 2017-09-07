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
PCM Device准备好了，应用程序可以调用snd_pcm_start()，开始读取或发送数据。
- SND\_PCM\_STATE\_RUNNING   
PCM Device已经启动了，开始处理samples。可以通过snd\_pcm\_drop()和snd\_pcm\_drain()停止。
- SND\_PCM\_STATE\_XRUN   
- SND\_PCM\_STATE\_DRAINING
- SND\_PCM\_STATE\_PAUSED
- SND\_PCM\_STATE\_SUSPENDED
- SND\_PCM\_STATE\_DISCONNECTED

# PCM formats
format list snd\_pcm\_format\_t。

# ALSA transfers
有两个版本的读写函数。一个是标准的读写，另一个通过audio buffer直接与硬件通信。

## read / write transfer
两个版本，interleaved和non-interleaved。
snd\_pcm\_writei() snd\_pcm\_readi()
snd\_pcm\_writen() and snd\_pcm\_readn()

# Direct Read / Write transfer (via mmap'ed areas)
三种ring buffer

- SND\_PCM\_ACCESS\_MMAP\_INTERLEAVED
- SND\_PCM\_ACCESS\_MMAP\_NONINTERLEAVED
- SND\_PCM\_ACCESS\_MMAP\_COMPLEX

应用通过snd\_pcm\_mmap\_begin()获取一个memory area的访问权，传输完数据后通过snd\_pcm\_mmap\_commit()通知alsa lib改变ring buffer指针。

与标准读写类似的接口 snd\_pcm\_mmap\_readi(), snd\_pcm\_mmap\_writei(), snd\_pcm\_mmap\_readn() and snd\_pcm\_mmap\_writen()

# Managing parameters
Alsa pcm device有两类的参数，hardware parameters包含stream description，format，rate等，software parameter包含driver相关的参数。

## Hardware related parameters
snd\_pcm\_hw\_params\_t

### Access modes

### formats
snd\_pcm\_format\_t

## Software related parameters
snd\_pcm\_sw\_params\_t

# PCM naming conventions
## Default device
	defaults.pcm.card 0
	defaults.pcm.device 0
	defaults.pcm.subdevice -1
## HW device
	hw
	hw:0
	hw:0,0
	hw:supersonic,1
	hw:soundwave,1,2
	hw:DEV=1,CARD=soundwave,SUBDEV=2
## Plug->HW device
	plughw
	plughw:0
	plughw:0,0
	plughw:supersonic,1
	plughw:soundwave,1,2
	plughw:DEV=1,CARD=soundwave,SUBDEV=2
 


[pcm](http://alsa-project.org/alsa-doc/alsa-lib/pcm.html)