title: ASoC Supporting Audio on an Embedded Board
categories: ALSA
tags: [ALSA]
---
![](/images/alsa/asoc_anatomy.png)

# Machine driver
## 注册soc card
machine driver注册一个struct snd\_soc\_card   
头文件include/sound/soc.h

	int snd_soc_register_card(struct snd_soc_card *card);
	int snd_soc_unregister_card(struct snd_soc_card *card);
	int devm_snd_soc_register_card(struct device *dev, struct snd_soc_card *card);
	[...]
	/* SoC card */
	struct snd_soc_card {
	const char *name;
	const char *long_name;
	const char *driver_name;
	struct device *dev;
	struct snd_card *snd_card;
	[...]
	/* CPU <--> Codec DAI links */
	struct snd_soc_dai_link *dai_link; /* predefined links only */
	int num_links; /* predefined links only */
	struct list_head dai_link_list; /* all links */
	int num_dai_links;
	[...]
	};

## CPU DAI和codec DAI连接
struct snd\_soc\_dai\_link用于创建CPU DAI和codec DAI之间的连接

	

[ref](http://events.linuxfoundation.org/sites/events/files/slides/belloni-alsa-asoc_0.pdf)

