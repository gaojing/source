title: PCM (digital audio) interface
categories: ALSA
tags: [ALSA]
---

# machine 驱动代码

## sdp4430_soc_init
注册platform设备

	sdp4430_snd_device = platform_device_alloc("soc-audio", -1);


注册dai

	ret = snd_soc_register_dais(&sdp4430_snd_device->dev, dai, ARRAY_SIZE(dai));

### dai


	static struct snd_soc_dai_driver dai[] = {
	{
		.name = "Bluetooth",
		.playback = {
			.stream_name = "BT Playback",
			.channels_min = 1,
			.channels_max = 2,
			.rates = SNDRV_PCM_RATE_8000 | SNDRV_PCM_RATE_16000 |
						SNDRV_PCM_RATE_48000,
			.formats = SNDRV_PCM_FMTBIT_S16_LE,
		},
		.capture = {
			.stream_name = "BT Capture",
			.channels_min = 1,
			.channels_max = 2,
			.rates = SNDRV_PCM_RATE_8000 | SNDRV_PCM_RATE_16000 |
						SNDRV_PCM_RATE_48000,
			.formats = SNDRV_PCM_FMTBIT_S16_LE,
		},
	},
	/* TODO: make this a separate FM CODEC driver or DUMMY */
	{
		.name = "FM Digital",
		.playback = {
			.stream_name = "FM Playback",
			.channels_min = 1,
			.channels_max = 2,
			.rates = SNDRV_PCM_RATE_48000,
			.formats = SNDRV_PCM_FMTBIT_S16_LE,
		},
		.capture = {
			.stream_name = "FM Capture",
			.channels_min = 1,
			.channels_max = 2,
			.rates = SNDRV_PCM_RATE_48000,
			.formats = SNDRV_PCM_FMTBIT_S16_LE,
		},
	},
	{
		.name = "HDMI",
		.playback = {
			.stream_name = "HDMI Playback",
			.channels_min = 2,
			.channels_max = 8,
			.rates = SNDRV_PCM_RATE_32000 | SNDRV_PCM_RATE_44100 |
					SNDRV_PCM_RATE_48000,
			.formats = SNDRV_PCM_FMTBIT_S16_LE | SNDRV_PCM_FMTBIT_S24_LE,
		},
	},
	};

# snd_soc_card结构体

	static struct snd_soc_card snd_soc_sdp4430 = {
		.driver_name = "OMAP4",
		.long_name = "TI OMAP4 Board",
		.dai_link = sdp4430_dai,
		.num_links = ARRAY_SIZE(sdp4430_dai),
		.stream_event = sdp4430_stream_event,
	};