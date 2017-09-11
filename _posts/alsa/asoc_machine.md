title: ASOC Machine
categories: ALSA
tags: [ALSA]
---
Machine负责platform和codec之间的耦合。

# 1. 注册platform device
ASOC把声卡注册为platform device。参考galaxy nexus例子，代码位于/sound/soc/omap/sdp4430.c

	sdp4430_soc_init(void)

	sdp4430_snd_device = platform_device_alloc("soc-audio", -1);
	if (!sdp4430_snd_device) {
		printk(KERN_ERR "Platform device allocation failed\n");
		ret = -ENOMEM;
		goto device_err;
	}

	ret = snd_soc_register_dais(&sdp4430_snd_device->dev, dai, ARRAY_SIZE(dai));
	if (ret < 0)
		goto err;
	platform_set_drvdata(sdp4430_snd_device, &snd_soc_sdp4430);

	ret = platform_device_add(sdp4430_snd_device);
	if (ret)
		goto err;
	twl6040_codec = snd_soc_card_get_codec(&snd_soc_sdp4430,
					"twl6040-codec");

模块初始化时，注册一个名为soc-audio的platform设备，把snd\_soc\_sdp4430设到platform\_device结构的dev->p->driver_data中，snd\_soc\_sdp4430是snd\_soc\_card结构的实例

	static struct snd_soc_card snd_soc_sdp4430 = {
		.driver_name = "OMAP4",
		.long_name = "TI OMAP4 Board",
		.dai_link = sdp4430_dai,
		.num_links = ARRAY_SIZE(sdp4430_dai),
		.stream_event = sdp4430_stream_event,
	};

通过snd\_soc\_card结构引入

- snd_soc_dai_link
- snd_soc_ops

[参考 Linux ALSA声卡驱动之六：ASoC架构中的Machine](http://blog.csdn.net/DroidPhone/article/details/7231605)