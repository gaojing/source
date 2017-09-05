title: Linux ALSA sound notes
categories: ALSA
tags: [ALSA]
---

# Structure of ALSA and terminology
![](/images/alsa/alsa-archtect.gif)

# The kernel module level
Some important design principles of the ALSA kernel modules:

- The kernel modules are designed to provide both an operational interface via entries in the /dev/ tree, and a status and configuration interface via entries in the /proc/asound/ tree.
- The operational interface in /dev/ contains three main types of devices, PCM devices for recording or playing digitized sound samples, CTL devices that allow manipulating the internal mixer and routing of the card, and MIDI devices to control the MIDI port of the card if any. There may be also sequencer devices if the card has a builtin sound synthesizer, and timer devices to help in using the sequencer.
- The kernel modules have been designed so that they offer a similar interface if they are for similar cards. In particular the names of controls have been deliberately kept similar, so that the master volume control, for example, is always called Master Volume.

PCM devices come in two varieties: output and input. MIDI devices are output only, and so are sequencer devices. Then the sound card has input or output sockets, and the mixer, which is controlled via the CTL device, routes sound samples from/to the PCM (or MIDI, sequencer) devices and the sockets, or among the devices.

# Names and functions of controls
通过alsa card的control interface可以获取controls的列表。

- amixer scontrols：返回简单的controls信息
- amixer controls：返回完全的列表
- amixer contents：返回完全的列表以及当前和可能的值

三种类型的controls：

- Playback controls，通常与一个output device关联，可以被mute或者unmute，通常包含一个volume level和unmute switch，通常与output或者copy(input-to-output)routes关联。
- Capture controls，与一个input device关联，可以设置为capture和not to capture，通常包含一个volume level和capture switch， 与input或者copy(output-to-input)routes关联。
- Feature control，不与device对应，驱动card或者mixer的功能。

# The /proc/asound kernel interface

# The /dev/snd device interface

# The ALSA library level

## asound.conf

# Plugins
