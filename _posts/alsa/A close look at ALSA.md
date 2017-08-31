title: A close look at ALSA
categories: ALSA
tags: [ALSA]
---

# Intro
Useful resource
[Linux ALSA sound notes](http://www.sabi.co.uk/Notes/linuxSoundALSA.html)   
[Introduction to Sound Programming with ALSA](http://www.linuxjournal.com/article/6735)   
[Alsa-sound-mini-HOWTO](http://www.alsa-project.org/~valentyn/Alsa-sound-mini-HOWTO.html)

# ALSA concepts

## Sound cards and hardware devices
ALSA通过cards， devices和subdevices层级来描述音频硬件设备。

alsa cards与声卡硬件对应， 大多数alsa硬件访问位于device层。

##　PCM parameters and the configuration space
数字音频包含采样率，通道数和sample value存储的格式等参数。ALSA通过一个N维的空间来描述这些参数。

## ALSA devices and plugins
为了防止混淆，