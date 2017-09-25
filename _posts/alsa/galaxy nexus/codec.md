title: galaxy nexus asoc codec
categories: galaxy nexus
tags: [galaxy nexus]
---
# codec模块init

	static struct platform_driver twl6040_codec_driver = {
		.driver = {
			.name = "twl6040-codec",
			.owner = THIS_MODULE,
		},
		.probe = twl6040_codec_probe,
		.remove = __devexit_p(twl6040_codec_remove),
	};

	static int __init twl6040_codec_init(void)
	{
		return platform_driver_register(&twl6040_codec_driver);
	}

twl6040\_codec\_probe被调用

	static int __devinit twl6040_codec_probe(struct platform_device *pdev)
	{
		return snd_soc_register_codec(&pdev->dev,
				&soc_codec_dev_twl6040, twl6040_dai, ARRAY_SIZE(twl6040_dai));
	}

通过snd\_soc\_register\_codec注册codec driver和codec dai driver

# codec定义的kcontrol
根据twl6040 codec定义了以下control

- capture前置放大音量
- capture音量
- Aux FM音量
- HeadSet播放音量
- Handsfree播放音量
- Earphone播放音量

	static const struct snd_kcontrol_new twl6040_snd_controls[] = {
		/* Capture gains */
		SOC_DOUBLE_TLV("Capture Preamplifier Volume",
			TWL6040_REG_MICGAIN, 6, 7, 1, 1, mic_preamp_tlv),
		SOC_DOUBLE_TLV("Capture Volume",
			TWL6040_REG_MICGAIN, 0, 3, 4, 0, mic_amp_tlv),
	
		/* AFM gains */
		SOC_DOUBLE_TLV("Aux FM Volume",
			TWL6040_REG_LINEGAIN, 0, 3, 7, 0, afm_amp_tlv),
	
		/* Playback gains */
		SOC_DOUBLE_EXT_TLV("Headset Playback Volume",
			TWL6040_REG_HSGAIN, 0, 4, 0xF, 1,
			twl6040_get_volsw, twl6040_put_volsw, hs_tlv),
		SOC_DOUBLE_R_EXT_TLV("Handsfree Playback Volume",
			TWL6040_REG_HFLGAIN, TWL6040_REG_HFRGAIN, 0, 0x1D, 1,
			twl6040_get_volsw_2r, twl6040_put_volsw_2r_vu, hf_tlv),
		SOC_SINGLE_EXT_TLV("Earphone Playback Volume",
			TWL6040_REG_EARCTL, 1, 0xF, 1,
			twl6040_get_volsw, twl6040_put_volsw, ep_tlv),
	
		SOC_ENUM_EXT("Headset Power Mode", twl6040_headset_power_enum,
			twl6040_headset_power_get_enum,
			twl6040_headset_power_put_enum),
	};

### Headset Power Mode
定义了一个类似mux的控件，

	static const char *twl6040_headset_power_texts[] = {
		"Low-Power", "High-Performance",
	};
	
	static const struct soc_enum twl6040_headset_power_enum =
		SOC_ENUM_SINGLE_EXT(ARRAY_SIZE(twl6040_headset_power_texts),
				twl6040_headset_power_texts);

	SOC_ENUM_EXT("Headset Power Mode", twl6040_headset_power_enum,
			twl6040_headset_power_get_enum,
			twl6040_headset_power_put_enum),