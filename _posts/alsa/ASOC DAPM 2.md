title: ASOC DAPM - widget-具备路径和电源管理信息的kcontrol
categories: ALSA
tags: [ALSA]
---
kcontrol有一些不足，例如无法描述各个kcontrol的连接关系。

# DAPM的基本单元 widget
widget把kcontrol和动态电源管理进行了有机的结合，同时还具备音频路径的连结功能，用结构体snd_soc_dapm_widget来描述。

# widget的种类
snd_soc_dapm_type描述

# widget之间的连接器：path
path用来将widget连接到一起。

# widget的连接关系：route
一个路径包含起始端widget，路径path，到达端widget，在DAPM中用snd\_soc\_dapm\_route表示。

	struct snd_soc_dapm_route {
		const char *sink;
		const char *control;
		const char *source;
	
		/* Note: currently only supported for links where source is a supply */
		int (*connected)(struct snd_soc_dapm_widget *source,
				 struct snd_soc_dapm_widget *sink);
	};

