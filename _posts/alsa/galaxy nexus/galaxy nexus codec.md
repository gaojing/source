title: galaxy nexus twl6040定义的dapm widget
categories: 
tags: []
---
# twl6040 codec定义的dapm widget
代码位于sound/soc/codecs/twl6040.c

## 1. 左模拟麦克风输入选择

### 1.1 定义mux kcontrol texts
有三个输入源，主麦克风，耳机麦克风或者Aux/FM Left输入

	static const char *twl6040_amicl_texts[] =
		{"Headset Mic", "Main Mic", "Aux/FM Left", "Off"};

### 1.2 定义soc\_enum结构体

	static const struct soc_enum twl6040_enum[] = {
		SOC_ENUM_SINGLE(TWL6040_REG_MICLCTL, 3, 4, twl6040_amicl_texts),
		SOC_ENUM_SINGLE(TWL6040_REG_MICRCTL, 3, 4, twl6040_amicr_texts),
	};

### 1.3 定义snd\_kcontrol\_new结构体

	static const struct snd_kcontrol_new amicl_control =
		SOC_DAPM_ENUM("Route", twl6040_enum[0]);

#### 1.4 定义snd\_soc\_dapm\_widget

	SND_SOC_DAPM_MUX("Analog Left Capture Route",
			SND_SOC_NOPM, 0, 0, &amicl_control),

## 2. 左模拟麦克风的route

### 2.1 左模拟麦克风mux的输入路径
定义的INPUT连接到mux上对应的输入端

	static const struct snd_soc_dapm_route intercon[] = {

		{"Analog Left Capture Route", "Headset Mic", "HSMIC"},
		{"Analog Left Capture Route", "Main Mic", "MAINMIC"},
		{"Analog Left Capture Route", "Aux/FM Left", "AFML"},

mux的输出接到MicAmpL上

		{"MicAmpL", NULL, "Analog Left Capture Route"},

### 2. 右模拟麦克风输入选择
和左模拟麦克风类似

