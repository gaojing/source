title: galaxy nexus asoc
categories: asoc
tags: [asoc]
---
记录galaxy nexus asoc驱动。

# 1. machine

## 1.1 sdp4430_soc_init初始化函数
## 1.2 建立soc-audio platform设备
建立一个名为soc-audio的平台设备

	sdp4430_snd_device = platform_device_alloc("soc-audio", -1);
	if (!sdp4430_snd_device) {
		printk(KERN_ERR "Platform device allocation failed\n");
		ret = -ENOMEM;
		goto device_err;
	}

将snd\_soc\_sdp4430设为sdp4430\_snd\_device设备的drvdata

	platform_set_drvdata(sdp4430_snd_device, &snd_soc_sdp4430);

## 1.3 snd\_soc\_card结构体
snd\_soc\_sdp4430是一个snd\_soc\_card结构体实例

	static struct snd_soc_card snd_soc_sdp4430 = {
		.driver_name = "OMAP4",
		.long_name = "TI OMAP4 Board",
		.dai_link = sdp4430_dai,
		.num_links = ARRAY_SIZE(sdp4430_dai),
		.stream_event = sdp4430_stream_event,
	};

### 1.3.1 snd\_soc\_dai\_link结构体
在snd\_soc\_sdp4430中包含了snd\_soc\_dai\_link的指针，描述了dai link的信息

# 2. machine注册的platform对应的驱动
对于machine建立的soc-audio平台设备，有对应的平台驱动，代码在sound/soc/soc-core.c中

## 2.1 注册平台驱动
可以看到注册平台驱动的name和上面建立的平台设备的name匹配

	static struct platform_driver soc_driver = {
		.driver		= {
			.name		= "soc-audio",
			.owner		= THIS_MODULE,
			.pm		= &snd_soc_pm_ops,
		},
		.probe		= soc_probe,
		.remove		= soc_remove,
		.shutdown	= soc_shutdown,
	};

	static int __init snd_soc_init(void)
	{
		...
		platform_driver_register(&soc_driver);
		...

	}

## 2.2 soc_probe
驱动匹配后会调用soc_probe回调



