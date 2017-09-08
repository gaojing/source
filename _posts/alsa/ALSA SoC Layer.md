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