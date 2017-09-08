title: ALSA SoC Layer
categories: ALSA
tags: [ALSA]
---

# Chapter 1 ALSA SoC Layer Overview
ASOC目标是为soc提供更好的alsa支持和portable audio codec。

## ASoC Design
- Codec class drivers

The codec class driver is platform independent and contains audio controls, audio interface capabilities, codec DAPM definition and codec IO functions.

- Platform class drivers

The platform class driver includes the audio DMA engine driver, digital audio interface (DAI) drivers (e.g. I2S, AC97, PCM) and any audio DSP drivers for that platform.

- Machine class driver

The machine driver class acts as the glue that describes and binds the other component drivers together to form an ALSA “sound card device”.

# Chapter 2 ASoC Codec Class Driver
codec class driver是硬件无关的，配置codec,fm,modem,bt或者外部的dsp来提供audio palyback和capture。

每个codec class driver必须提供下面的功能。

- Codec DAI and PCM configuration
- Codec control IO - using RegMap API
- Mixers and audio controls
- Codec audio operations
- DAPM description.
- DAPM event handler

可以选择提供

- DAC Digital mute control.

## ASoC Codec driver breakdown
### Codec DAI and PCM configuration
每个codec driver必须提供struct snd\_soc\_dai\_driver来定义它的DAI和PCM capabilities与operations。这个被export并且有machine driver将其注册到core中。

	static struct snd_soc_dai_ops wm8731_dai_ops = {
      .prepare        = wm8731_pcm_prepare,
      .hw_params      = wm8731_hw_params,
      .shutdown       = wm8731_shutdown,
      .digital_mute   = wm8731_mute,
      .set_sysclk     = wm8731_set_dai_sysclk,
      .set_fmt        = wm8731_set_dai_fmt,
	};
	
	struct snd_soc_dai_driver wm8731_dai = {
	      .name = "wm8731-hifi",
	      .playback = {
	              .stream_name = "Playback",
	              .channels_min = 1,
	              .channels_max = 2,
	              .rates = WM8731_RATES,
	              .formats = WM8731_FORMATS,},
	      .capture = {
	              .stream_name = "Capture",
	              .channels_min = 1,
	              .channels_max = 2,
	              .rates = WM8731_RATES,
	              .formats = WM8731_FORMATS,},
	      .ops = &wm8731_dai_ops,
	      .symmetric_rates = 1,
	};

### Codec control IO
通常codec由I2C，SPI等控制，coder driver应当使用remap api来使用codec io。

### mixers和audio control
通过宏可以定义mixer与audio control

	#define SOC_SINGLE(xname, reg, shift, mask, invert)

定义一个control

	xname = Control name e.g. "Playback Volume"
	reg = codec register
	shift = control bit(s) offset in register
	mask = control bit size(s) e.g. mask of 7 = 3 bits
	invert = the control is inverted

### Codec Audio Operations
codec driver支持下面的alsa pcm operations

	struct snd_soc_ops {
      int (*startup)(struct snd_pcm_substream *);
      void (*shutdown)(struct snd_pcm_substream *);
      int (*hw_params)(struct snd_pcm_substream *, struct snd_pcm_hw_params *);
      int (*hw_free)(struct snd_pcm_substream *);
      int (*prepare)(struct snd_pcm_substream *);
	};

### DAPM description
Dynamic Audio Power Management description。

### DAPM event handler

### Codec DAC digital mute control
大多数codecs在DAC前都有一个digital mute， mute阻止digital数据进入DAC。

为每个codec DAI创建一个asoc core调用的callback，当mute使用或释放时调用。

# Chapter 3 ASoC Digital Audio Interface (DAI)
ASOC支持三种主要的Digital Audio Interface, AC97, I2S, PCM。

- AC97   

- I2S   
I2S是四线DAI，tx和rx用于传输音频数据，bit clock(BCLK)和left/right clock(LRC)用于同步。   
I2S operating mode：

	- I2S   
	MSB is transmitted on the falling edge of the first BCLK after LRC transition
	- Left Justified   
	MSB is transmitted on transition of LRC
	- Right Justified   
	MSB is transmitted sample size BCLKs before LRC transition.

- PCM

# Chapter 4 Dynamic Audio Power Management for Portable Devices

# Chapter 5 ASoC Platform Driver
ASoc platform driver由audio DMA drivers, SoC DAI drivers and DSP drivers组成。

## Audio DMA
DMA driver可以支持下面operation

	/* SoC audio ops */
	struct snd_soc_ops {
	      int (*startup)(struct snd_pcm_substream *);
	      void (*shutdown)(struct snd_pcm_substream *);
	      int (*hw_params)(struct snd_pcm_substream *, struct snd_pcm_hw_params *);
	      int (*hw_free)(struct snd_pcm_substream *);
	      int (*prepare)(struct snd_pcm_substream *);
	      int (*trigger)(struct snd_pcm_substream *, int);
	};

platform driver通过snd_soc_platform_driver到处DMA功能

	struct snd_soc_platform_driver {
	      char *name;
	
	      int (*probe)(struct platform_device *pdev);
	      int (*remove)(struct platform_device *pdev);
	      int (*suspend)(struct platform_device *pdev, struct snd_soc_cpu_dai *cpu_dai);
	      int (*resume)(struct platform_device *pdev, struct snd_soc_cpu_dai *cpu_dai);
	
	      /* pcm creation and destruction */
	      int (*pcm_new)(struct snd_card *, struct snd_soc_codec_dai *, struct snd_pcm *);
	      void (*pcm_free)(struct snd_pcm *);
	
	      /*
	       * For platform caused delay reporting.
	       * Optional.
	       */
	      snd_pcm_sframes_t (*delay)(struct snd_pcm_substream *,
	              struct snd_soc_dai *);
	
	      /* platform stream ops */
	      struct snd_pcm_ops *pcm_ops;
	};

## SoC DAI Drivers
必须支持下面功能

- Digital audio interface (DAI) description
- Digital audio interface configuration
- PCM’s description
- SYSCLK configuration
- Suspend and resume (optional)

## SoC DSP Drivers
通常提供下面的功能

- DAPM graph
- Mixer controls
- DMA IO to/from DSP buffers (if applicable)
- Definition of DSP front end (FE) PCM devices.

# Chapter 6 ASoC Machine Driver
The ASoC machine (or board) driver is the code that glues together all the component drivers (e.g. codecs, platforms and DAIs). It also describes the relationships between each component which include audio paths, GPIOs, interrupts, clocking, jacks and voltage regulators.

The machine driver can contain codec and platform specific code. It registers the audio subsystem with the kernel as a platform device and is represented by the following struct:

	/* SoC machine */
	struct snd_soc_card {
	      char *name;
	
	      ...
	
	      int (*probe)(struct platform_device *pdev);
	      int (*remove)(struct platform_device *pdev);
	
	      /* the pre and post PM functions are used to do any PM work before and
	       * after the codec and DAIs do any PM work. */
	      int (*suspend_pre)(struct platform_device *pdev, pm_message_t state);
	      int (*suspend_post)(struct platform_device *pdev, pm_message_t state);
	      int (*resume_pre)(struct platform_device *pdev);
	      int (*resume_post)(struct platform_device *pdev);
	
	      ...
	
	      /* CPU <--> Codec DAI links  */
	      struct snd_soc_dai_link *dai_link;
	      int num_links;
	
	      ...
	};