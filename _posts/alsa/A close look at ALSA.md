title: A close look at ALSA
categories: ALSA
tags: [ALSA]
---

# Intro
Useful resource  
[A close look at ALSA](http://www.volkerschatz.com/noise/alsa.html)  
[Linux ALSA sound notes](http://www.sabi.co.uk/Notes/linuxSoundALSA.html)   
[Introduction to Sound Programming with ALSA](http://www.linuxjournal.com/article/6735)   
[Alsa-sound-mini-HOWTO](http://www.alsa-project.org/~valentyn/Alsa-sound-mini-HOWTO.html)

# ALSA concepts

## Sound cards and hardware devices
ALSA通过cards， devices和subdevices层级来描述音频硬件设备。

alsa cards与声卡硬件对应， 大多数alsa硬件访问位于device层。

## PCM parameters and the configuration space
数字音频包含采样率，通道数和sample value存储的格式等参数。ALSA通过一个N维的空间来描述这些参数。

## ALSA devices and plugins
为了防止混淆，ALSA设备由字符串标注。ALSA device在配置文件中定义，一般都是plugin的封装。

hw plugin，直接访问硬件驱动。
plug plugin，加了一些转换的功能。

# configure alsa

## configuration file
configuration file用来定义alsa device。

Alsa支持三种层级的配置文件。第一个是alsa.conf，通常在alsa数据目录，/usr/share/alsa。第二个系统全局配置文件在/etc/asoundrc。用户自己的配置文件.asoundrc。

## Basic configuration file format
alsa配置文件是按层级组织的parameter-value对。最上一层与alsa提供的interface对应。alsa的interface由一些alsa api的集合构成，包含打开设备，操作，关闭设备等。   
列出PCM interface的定义

	aplay -L

# Troubleshooting

## Useful tools
获取声卡的信息可以通过下面命令

	aplay -l
	arecord -l

aplay的-v选项可以打印出播放中使用的每个subdevice的hardware parameter。

当subdevice正在使用中时，可以通过/proc/asound/card#/pcm#p/sub#/hw_params获取信息。

## alsacap
[source](http://www.volkerschatz.com/noise/alsacap.tgz)
[mannual](http://www.volkerschatz.com/noise/alsacap.html)

