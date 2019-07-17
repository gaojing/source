title: ASOC Platform
categories: ALSA
tags: [ALSA]
---

# Table of Contents
- [1. Overview](#section1)
- [2. HAL](#section2)
	- [2.1 HAL modules](#section2.1)



# 1. Platform驱动在ASoC中的作用
Platform驱动的主要作用是完成音频数据的管理，最终通过CPU的数字音频接口（DAI）把音频数据传送给Codec进行处理，最终由Codec输出驱动耳机或者是喇叭的音信信号。在具体实现上，ASoC有把Platform驱动分为两个部分：snd\_soc\_platform\_driver和snd\_soc\_dai\_driver。其中，platform\_driver负责管理音频数据，把音频数据通过dma或其他操作传送至cpu dai中，dai\_driver则主要完成cpu一侧的dai的参数配置，同时也会通过一定的途径把必要的dma等参数与snd\_soc\_platform\_driver进行交互。

# 2.  snd\_soc\_platform\_driver的注册
ASoC把snd\_soc\_platform\_driver注册为一个系统的platform\_driver

- 定义一个snd\_soc\_platform\_driver结构的实例；
- 在platform\_driver的probe回调中利用ASoC的API：snd\_soc\_register\_platform()注册上面定义的实例；
- 实现snd\_soc\_platform\_driver中的各个回调函数；
	
		static struct snd_soc_platform_driver omap_soc_platform = {
			.ops		= &omap_pcm_ops,
			.pcm_new	= omap_pcm_new,
			.pcm_free	= omap_pcm_free_dma_buffers,
		};
		
		static __devinit int omap_pcm_probe(struct platform_device *pdev)
		{
			return snd_soc_register_platform(&pdev->dev,
					&omap_soc_platform);
		}
		
		static int __devexit omap_pcm_remove(struct platform_device *pdev)
		{
			snd_soc_unregister_platform(&pdev->dev);
			return 0;
		}
		
		static struct platform_driver omap_pcm_driver = {
			.driver = {
					.name = "omap-pcm-audio",
					.owner = THIS_MODULE,
			},
		
			.probe = omap_pcm_probe,
			.remove = __devexit_p(omap_pcm_remove),
		};

# 3.  cpu的snd_soc_dai driver驱动的注册
dai驱动通常对应cpu的一个或几个I2S/PCM接口，与snd\_soc\_platform一样，dai驱动也是实现为一个platform driver，实现一个dai驱动大致可以分为以下几个步骤

- 定义一个snd\_soc\_dai\_driver结构的实例
- 在对应的platform\_driver中的probe回调中通过API：snd\_soc\_register\_dai或者snd\_soc\_register\_dais，注册snd\_soc\_dai实例；
- 实现snd\_soc\_dai\_driver结构中的probe、suspend等回调；
- 实现snd\_soc\_dai\_driver结构中的snd\_soc\_dai\_ops字段中的回调函数；

# 4.  snd_soc_dai_driver中的ops字段
ops字段指向一个snd_soc_dai_ops结构，该结构实际上是一组回调函数的集合，dai的配置和控制几乎都是通过这些回调函数来实现的



[参考 Linux ALSA声卡驱动之八：ASoC架构中的Platform](http://blog.csdn.net/DroidPhone/article/details/7316061)