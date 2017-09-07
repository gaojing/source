title: Designs and Implementations
categories: ALSA
tags: [ALSA]
---

# Chapter 1 Standard ALSA Control Names
描述mixer controls的标准名称。
## Standard Syntax
Syntax: [LOCATION] SOURCE [CHANNEL] [DIRECTION] FUNCTION
...

# Chapter 2 ALSA PCM channel-mapping API
## general
channel mapping api运行用户查询可能的channel maps和当前的channel map，改变map。
一个channel map是一个包含每个PCM channel位置的数组。

## Design
每个PCM substream可能包含一个control element提供channel mapping信息和配置。通过下面定义

- iface = SNDRV_CTL_ELEM_IFACE_PCM
- name = “Playback Channel Map” or “Capture Channel Map”
- device = the same device number for the assigned PCM substream
- index = the same index number for the assigned PCM substream

每个control提供TLV read operation和read operation。可能write operation。

## TLV
TLV operation返回可能的channel maps的列表。列表的每一项可能是type data-bytes ch0 ch1 ch2...

# Chapter 3 ALSA Compress-Offload API

# Chapter 6 Proc Files of ALSA Drivers
## General
ALSA拥有自己的proc tree，/proc/asound。
## Global information
