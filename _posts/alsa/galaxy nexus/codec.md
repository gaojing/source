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

