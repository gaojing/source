title: ASOC Codec
categories: ALSA
tags: [ALSA]
---
# Codec简介
Codec作用主要有以下四种

- PCM等信号进行DA转换
- 对Mic等信号进行AD转换
- 对音频通路
- 对音频信号进行处理，音量控制等

# 1. ASOC对codec的抽象
主要由snd\_soc\_codec，snd\_soc\_codec\_driver，snd\_soc\_dai，snd\_soc\_dai\_driver四个结构体描述。

# 2. Codec的注册
Codec驱动的首要任务就是确定snd\_soc\_codec和snd\_soc\_dai的实例，并把它们注册到系统中，machine才能使用它。以twl6040为例，对应的代码位置在sound/soc/codecs/twl6040.c，注册了一个platform driver。

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
	module_init(twl6040_codec_init);

platform注册后，probe函数会被调用

	static int __devinit twl6040_codec_probe(struct platform_device *pdev)
	{
		return snd_soc_register_codec(&pdev->dev,
				&soc_codec_dev_twl6040, twl6040_dai, ARRAY_SIZE(twl6040_dai));
	}

soc\_codec\_dev\_twl6040的定义如下

	static struct snd_soc_codec_driver soc_codec_dev_twl6040 = {
		.probe = twl6040_probe,
		.remove = twl6040_remove,
		.suspend = twl6040_suspend,
		.resume = twl6040_resume,
		.read = twl6040_read_reg_cache,
		.write = twl6040_write,
		.set_bias_level = twl6040_set_bias_level,
		.reg_cache_size = ARRAY_SIZE(twl6040_reg),
		.reg_word_size = sizeof(u8),
		.reg_cache_default = twl6040_reg,
	};

twl6040\_dai的定义如下

	static struct snd_soc_dai_driver twl6040_dai[] = {
	{
		.name = "twl6040-ul",
		.capture = {
			.stream_name = "Capture",
			.channels_min = 1,
			.channels_max = 2,
			.rates = TWL6040_RATES,
			.formats = TWL6040_FORMATS,
		},
		.ops = &twl6040_dai_ops,
	},
	{
		.name = "twl6040-dl1",
		.playback = {
			.stream_name = "Headset Playback",
			.channels_min = 1,
			.channels_max = 2,
			.rates = TWL6040_RATES,
			.formats = TWL6040_FORMATS,
		},
		.ops = &twl6040_dai_ops,
	},
	{
		.name = "twl6040-dl2",
		.playback = {
			.stream_name = "Handsfree Playback",
			.channels_min = 1,
			.channels_max = 2,
			.rates = TWL6040_RATES,
			.formats = TWL6040_FORMATS,
		},
		.ops = &twl6040_dai_ops,
	},
	{
		.name = "twl6040-vib",
		.playback = {
			.stream_name = "Vibra Playback",
			.channels_min = 2,
			.channels_max = 2,
			.rates = SNDRV_PCM_RATE_CONTINUOUS,
			.formats = TWL6040_FORMATS,
		},
		.ops = &twl6040_dai_ops,
	},
	};

可见codec驱动的第一个步骤就是定义snd\_soc\_codec\_driver和snd\_soc\_dai\_driver的实例，然后调用snd\_soc\_register\_codec注册codec。

## 2.1 snd\_soc\_register\_codec流程
首先，申请一个snd\_soc\_codec实例

	codec = kzalloc(sizeof(struct snd_soc_codec), GFP_KERNEL);
	if (codec == NULL)
		return -ENOMEM;

确定codec的名字，machine驱动定义的snd\_soc\_dai\_link中指定的codec的名字就是和这个比较的。

	if (codec_drv == &null_codec_drv)
		codec->name = kstrdup("null-codec", GFP_KERNEL);
	else
		codec->name = fmt_single_name(dev, &codec->id);
	if (codec->name == NULL) {
		kfree(codec);
		return -ENOMEM;
	}

接着初始化它的各个字段，多数来自上面定义的snd_soc_codec_driver的实例soc\_codec\_dev\_twl6040。

	if (codec_drv->compress_type)
		codec->compress_type = codec_drv->compress_type;
	else
		codec->compress_type = SND_SOC_FLAT_COMPRESSION;

	codec->write = codec_drv->write;
	codec->read = codec_drv->read;
	codec->volatile_register = codec_drv->volatile_register;
	codec->readable_register = codec_drv->readable_register;
	codec->writable_register = codec_drv->writable_register;
	codec->dapm.bias_level = SND_SOC_BIAS_OFF;
	codec->dapm.dev = dev;
	codec->dapm.codec = codec;
	codec->dapm.seq_notifier = codec_drv->seq_notifier;
	codec->dapm.stream_event = codec_drv->stream_event;
	codec->dev = dev;
	codec->driver = codec_drv;
	codec->num_dai = num_dai;

然后初始化一些寄存器和做一些配置工作。

	/* allocate CODEC register cache */
	if (codec_drv->reg_cache_size && codec_drv->reg_word_size) {
		reg_size = codec_drv->reg_cache_size * codec_drv->reg_word_size;
		codec->reg_size = reg_size;
		/* it is necessary to make a copy of the default register cache
		 * because in the case of using a compression type that requires
		 * the default register cache to be marked as __devinitconst the
		 * kernel might have freed the array by the time we initialize
		 * the cache.
		 */
		if (codec_drv->reg_cache_default) {
			codec->reg_def_copy = kmemdup(codec_drv->reg_cache_default,
						      reg_size, GFP_KERNEL);
			if (!codec->reg_def_copy) {
				ret = -ENOMEM;
				goto fail;
			}
		}
	}

	if (codec_drv->reg_access_size && codec_drv->reg_access_default) {
		if (!codec->volatile_register)
			codec->volatile_register = snd_soc_default_volatile_register;
		if (!codec->readable_register)
			codec->readable_register = snd_soc_default_readable_register;
		if (!codec->writable_register)
			codec->writable_register = snd_soc_default_writable_register;
	}

	for (i = 0; i < num_dai; i++) {
		fixup_codec_formats(&dai_drv[i].playback);
		fixup_codec_formats(&dai_drv[i].capture);
	}

通过snd\_soc\_register\_dais对本code的dai进行注册。

	/* register any DAIs */
	if (num_dai) {
		ret = snd_soc_register_dais(dev, dai_drv, num_dai);
		if (ret < 0)
			goto fail;
	}

最后把codec实例添加到全局链表codec_list中。调用snd\_soc\_instantiate\_cards进行一次匹配操作。

	mutex_lock(&client_mutex);
	list_add(&codec->list, &codec_list);
	snd_soc_instantiate_cards();
	mutex_unlock(&client_mutex);

snd\_soc\_register\_dais函数和snd\_soc\_register\_codec类似。

# 3. mfd设备
codec驱动把自己注册为一个platform driver，对应的platform device在drivers/mfd/twl6040-codec.c中。

	static struct platform_driver twl6040_driver = {
		.probe		= twl6040_probe,
		.remove		= __devexit_p(twl6040_remove),
		.driver		= {
			.owner	= THIS_MODULE,
			.name	= "twl6040-audio",
		},
	};

	static int __devinit twl6040_init(void)
	{
		return platform_driver_register(&twl6040_driver);
	}

注册platform驱动会调用probe函数twl6040_probe

首先分配一个twl6040结构体实例

	twl6040 = kzalloc(sizeof(struct twl6040), GFP_KERNEL);
	if (!twl6040)
		return -ENOMEM;

	platform_set_drvdata(pdev, twl6040);

进行一些初始化

	twl6040->icrev = twl6040_reg_read(twl6040, TWL6040_REG_ASICREV);
	if (twl6040->icrev < 0) {
		ret = twl6040->icrev;
		goto gpio1_err;
	}

	if (pdata && (twl6040_get_icrev(twl6040) > TWL6040_REV_1_0))
		audpwron = pdata->audpwron_gpio;
	else
		audpwron = -EINVAL;

	if (pdata)
		naudint = pdata->naudint_irq;
	else
		naudint = 0;

	twl6040->audpwron = audpwron;
	twl6040->powered = 0;
	twl6040->irq = naudint;
	twl6040->irq_base = pdata->irq_base;
	init_completion(&twl6040->ready);

...

添加mfd设备

	if (pdata->audio) {
		cell = &twl6040->cells[children];
		cell->name = "twl6040-codec";
		cell->platform_data = pdata->audio;
		cell->pdata_size = sizeof(*pdata->audio);
		children++;
	}

	if (pdata->vibra) {
		cell = &twl6040->cells[children];
		cell->name = "twl6040-vibra";
		cell->platform_data = pdata->vibra;
		cell->pdata_size = sizeof(*pdata->vibra);
		children++;
	}

	if (children) {
		ret = mfd_add_devices(&pdev->dev, pdev->id, twl6040->cells,
				      children, NULL, 0);
		if (ret)
			goto mfd_err;
	} else {
		dev_err(&pdev->dev, "No platform data found for children\n");
		ret = -ENODEV;
		goto mfd_err;
	}

# 4. codec相关
启动宏

	MACHINE_START(TUNA, "Tuna")
		/* Maintainer: Google, Inc */
		.boot_params	= 0x80000100,
		.reserve	= tuna_reserve,
		.map_io		= tuna_map_io,
		.init_early	= tuna_init_early,
		.init_irq	= gic_init_irq,
		.init_machine	= tuna_init,
		.timer		= &omap_timer,
	MACHINE_END

系统启动时调用tuna_init，在函数中一次调用

	tuna_audio_init();
	tuna_i2c_init();

涉及到的实例，twl6040_codec全局变量

	static struct twl4030_codec_data twl6040_codec = {
		.audio		= &twl6040_audio,
		.naudint_irq	= OMAP44XX_IRQ_SYS_2N,
		.irq_base	= TWL6040_CODEC_IRQ_BASE,
		.get_ext_clk32k	= tuna_twl6040_get_ext_clk32k,
		.put_ext_clk32k	= tuna_twl6040_put_ext_clk32k,
		.set_ext_clk32k	= tuna_twl6040_set_ext_clk32k,
	};
	
	static struct twl4030_platform_data tuna_twldata = {
		.irq_base	= TWL6030_IRQ_BASE,
		.irq_end	= TWL6030_IRQ_END,
	
		/* Regulators */
		.vmmc		= &tuna_vmmc,
		.vpp		= &tuna_vpp,
		.vusim		= &tuna_vusim,
		.vana		= &tuna_vana,
		.vcxio		= &tuna_vcxio,
		.vdac		= &tuna_vdac,
		.vusb		= &tuna_vusb,
		.vaux1		= &tuna_vaux1,
		.vaux2		= &tuna_vaux2,
		.vaux3		= &tuna_vaux3,
		.clk32kg	= &tuna_clk32kg,
		.clk32kaudio	= &tuna_clk32kaudio,
	
		/* children */
		.codec		= &twl6040_codec,
		.madc		= &twl6030_madc,
	
		/* SMPS */
		.vdd3		= &tuna_vdd3,
		.vmem		= &tuna_vmem,
		.v2v1		= &tuna_v2v1,
	};

在tuna_audio_init函数中设置codec data的poweron引脚

	twl6040_codec.audpwron_gpio = aud_pwron;
