title: alsa plugin
categories: ALSA
tags: [ALSA]
---
PCM plugin扩展PCM device的功能，包括sample conversion，channel间的sample copy。

# Slave definition
slave插件可以用一个string或者一个compound configuration node来定义。

	pcm_slave.NAME {
        pcm STR         # PCM name
        # or
        pcm { }         # PCM definition
        format STR      # Format or "unchanged"
        channels INT    # Count of channels or "unchanged" string
        rate INT        # Rate in Hz or "unchanged" string
        period_time INT # Period time in us or "unchanged" string
        buffer_time INT # Buffer time in us or "unchanged" string
	}

# Plugin: hw
这个插件与alsa kernel driver直接通信。

	pcm.name {
        type hw                 # Kernel PCM
        card INT/STR            # Card name (string) or number (integer)
        [device INT]            # Device number (default 0)
        [subdevice INT]         # Subdevice number (default -1: first available)
        [sync_ptr_ioctl BOOL]   # Use SYNC_PTR ioctl rather than the direct mmap access for control structures
        [nonblock BOOL]         # Force non-blocking open mode
        [format STR]            # Restrict only to the given format
        [channels INT]          # Restrict only to the given channels
        [rate INT]              # Restrict only to the given rate
        [chmap MAP]             # Override channel maps; MAP is a string array
	}

