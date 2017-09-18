title: ASOC DAPM - 在驱动程序中初始化并注册widget和route
categories: ALSA
tags: [ALSA]
---
从代码入手，分析如何注册widget，如何连接连个widget，一个widget的状态如何传到整个音频路径的

# dapm context
dapm把整个音频系统，按照功能和偏置电压级别，划分为若干个电源域，每个域包含各自的widget，每个域中的所有widget通常都处于同一个偏置电压级别上，而一个电源域就是一个dapm context，通常会有以下几种dapm context：

- 属于codec中的widget位于一个dapm context中
- 属于platform的widget位于一个dapm context中
- 属于整个声卡的widget位于一个dapm context中

ASOC使用snd\_soc\_dapm\_context来表示一个damp context

	struct snd_soc_dapm_context {
		int n_widgets; /* number of widgets in this context */
		enum snd_soc_bias_level bias_level;
		enum snd_soc_bias_level suspend_bias_level;
		struct delayed_work delayed_work;
		unsigned int idle_bias_off:1; /* Use BIAS_OFF instead of STANDBY */
	
		struct snd_soc_dapm_update *update;
	
		void (*seq_notifier)(struct snd_soc_dapm_context *,
				     enum snd_soc_dapm_type, int);
	
		struct device *dev; /* from parent - for debug */
		struct snd_soc_codec *codec; /* parent codec */
		struct snd_soc_platform *platform; /*parent platform */
		struct snd_soc_card *card; /* parent card */
	
		/* used during DAPM updates */
		int dev_power;
		struct list_head list;
	
		int num_valid_paths;
	
		int (*stream_event)(struct snd_soc_dapm_context *dapm);
	
	#ifdef CONFIG_DEBUG_FS
		struct dentry *debugfs_dapm;
	#endif
	};

snd\_soc\_bias\_level的取值范围是以下几种：
- SND_SOC_BIAS_OFF
- SND_SOC_BIAS_STANDBY
- SND_SOC_BIAS_PREPARE
- SND_SOC_BIAS_ON

snd\_soc\_dapm\_context被内嵌到代表codec、platform、card、dai的结构体中：

	struct snd_soc_codec {  
	        ......  
	        /* dapm */  
	        struct snd_soc_dapm_context dapm;  
	        ......  
	};  
	  
	struct snd_soc_platform {  
	        ......  
	        /* dapm */  
	        struct snd_soc_dapm_context dapm;  
	        ......  
	};  
	  
	struct snd_soc_card {  
	        ......  
	        /* dapm */  
	        struct snd_soc_dapm_context dapm;  
	        ......  
	};  
	:  
	struct snd_soc_dai {  
	        ......  
	        /* dapm */  
	        struct snd_soc_dapm_widget *playback_widget;  
	        struct snd_soc_dapm_widget *capture_widget;  
	        struct snd_soc_dapm_context dapm;  
	        ......  
	};  

widget结构中有snd\_soc\_damp\_context的指针，指向所属的codec、platform、card、或dai的dapm结构。同时，所有的dapm结构，通过它的list字段，链接到代表声卡的snd_soc_card结构的dapm_list链表头字段。

# 创建和注册widget
# 注册音频路径

# dai widget
dai按所在的位置，分为cpu dai和codec dai，在硬件在，通常一个cpu dai会连接一个codec dai，在machine驱动上，要在snd\_soc\_card里指定一个snd\_soc\_dai\_link的结构，定义了使用哪一个cpu dai和codec dai连接。

	struct snd_soc_dai {
		const char *name;
		int id;
		struct device *dev;
		void *ac97_pdata;	/* platform_data for the ac97 codec */
	
		/* driver ops */
		struct snd_soc_dai_driver *driver;
	
		/* DAI runtime info */
		unsigned int capture_active:1;		/* stream is in use */
		unsigned int playback_active:1;		/* stream is in use */
		unsigned int symmetric_rates:1;
		struct snd_pcm_runtime *runtime;
		unsigned int active;
		unsigned char pop_wait:1;
		unsigned char probed:1;
	
		/* DAI DMA data */
		void *playback_dma_data;
		void *capture_dma_data;
	
		/* parent platform/codec */
		union {
			struct snd_soc_platform *platform;
			struct snd_soc_codec *codec;
		};
		struct snd_soc_card *card;
	
		struct list_head list;
		struct list_head card_list;
	};

dai由codec驱动和platform驱动接口注册，machine负责绑定。

## codec dai widget
codec注册驱动时，会传入该codec支持的dai个数和dai信息的结构体指针

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

这回使得ASoc把codec的dai注册到系统中，并把这些dai都挂在全局链表变量dai\_list中，然后，在codec被machine驱动匹配后，soc\_probe\_codec函数会被调用，他会通过全局链表变量dai\_list查找所有属于该codec的dai，调用snd\_soc\_dapm\_new\_dai\_widgets函数来生成该dai的播放流widget和录音流widget

## cpu dai widget 

# 端点widget
一条完整的dapm音频路径，必然有起点和终点，我们把位于这些起点和终点的widget称之为端点widget。以下这些类型的widget可以成为端点widget：

codec的输入输出引脚：

- snd\_soc\_dapm\_output
- snd\_soc\_dapm\_input
- 
外接的音频设备：

- snd\_soc\_dapm\_hp
- snd\_soc\_dapm\_spk
- snd\_soc\_dapm\_line
- 
音频流（stream domain）：

- snd\_soc\_dapm\_adc
- snd\_soc\_dapm\_dac
- snd\_soc\_dapm\_aif\_out
- snd\_soc\_dapm\_aif\_in
- snd\_soc\_dapm\_dai\_out
- snd\_soc\_dapm\_dai\_in
- 
电源、时钟和其它：

- snd\_soc\_dapm\_supply
- snd\_soc\_dapm\_regulator\_supply
- snd\_soc\_dapm\_clock\_supply
- snd\_soc\_dapm\_kcontrol
- 
当声卡上的其中一个widget的状态发生改变时，从这个widget开始，dapm框架会向前和向后遍历路径上的所有widget，判断每个widget的状态是否需要跟着变更，到达这些端点widget就会认为它是一条完整音频路径的开始和结束，从而结束一次扫描动作。

[ALSA声卡驱动中的DAPM详解之四：在驱动程序中初始化并注册widget和route](http://blog.csdn.net/DroidPhone/article/details/13756651)