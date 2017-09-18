title: ASOC DAPM - 如何定义各种widget
categories: ALSA
tags: [ALSA]
---
snd\_soc\_dapm\_widget和snd\_soc\_dapm\_route需要我们自己定义，snd\_soc\_dapm\_path在注册route时会动态生成。

# 定义widget
DAPM提供了大量的辅助宏来定义widget，根据widget的类型，按照电源所在的域，分为以下几个域

- codec域，例如VREF和VMID等提供参考电压的widget，通常在codec的probe/remove回调中控制。
- platform域，通常针对平台或板子上的物理接口，例如耳机，扬声器等。
- 音频连接域，一般指codec内部mixer，mux等控制音频路径的widget。
- 音频数据域，指需要处理音频数据流的widget，例如ADC，DAC等。

## codec域widget定义
	#define SND_SOC_DAPM_VMID(wname) \
	{	.id = snd_soc_dapm_vmid, .name = wname, .kcontrol_news = NULL, \
		.num_kcontrols = 0}

## platform域widget定义
	#define SND_SOC_DAPM_INPUT(wname) \
	{	.id = snd_soc_dapm_input, .name = wname, .kcontrol_news = NULL, \
		.num_kcontrols = 0, .reg = SND_SOC_NOPM }
	#define SND_SOC_DAPM_OUTPUT(wname) \
	{	.id = snd_soc_dapm_output, .name = wname, .kcontrol_news = NULL, \
		.num_kcontrols = 0, .reg = SND_SOC_NOPM }
	#define SND_SOC_DAPM_MIC(wname, wevent) \
	{	.id = snd_soc_dapm_mic, .name = wname, .kcontrol_news = NULL, \
		.num_kcontrols = 0, .reg = SND_SOC_NOPM, .event = wevent, \
		.event_flags = SND_SOC_DAPM_PRE_PMU | SND_SOC_DAPM_POST_PMD}
	#define SND_SOC_DAPM_HP(wname, wevent) \
	{	.id = snd_soc_dapm_hp, .name = wname, .kcontrol_news = NULL, \
		.num_kcontrols = 0, .reg = SND_SOC_NOPM, .event = wevent, \
		.event_flags = SND_SOC_DAPM_POST_PMU | SND_SOC_DAPM_PRE_PMD}
	#define SND_SOC_DAPM_SPK(wname, wevent) \
	{	.id = snd_soc_dapm_spk, .name = wname, .kcontrol_news = NULL, \
		.num_kcontrols = 0, .reg = SND_SOC_NOPM, .event = wevent, \
		.event_flags = SND_SOC_DAPM_POST_PMU | SND_SOC_DAPM_PRE_PMD}
	#define SND_SOC_DAPM_LINE(wname, wevent) \
	{	.id = snd_soc_dapm_line, .name = wname, .kcontrol_news = NULL, \
		.num_kcontrols = 0, .reg = SND_SOC_NOPM, .event = wevent, \
		.event_flags = SND_SOC_DAPM_POST_PMU | SND_SOC_DAPM_PRE_PMD}

widget分别对应信号发生器，输入引脚，输出引脚，麦克风，耳机，扬声器，线路输入接口。

## path域widget封装

对普通kcontrol再封装，增加音频路径和电源管理的功能，包含一个或多个kcontrol，需要使用damp的宏定义kcontrol。

# 建立widget和route
1. 定义widget需要的damp kcontrol   

		static const char *twl6040_hs_texts[] = {
			"Off", "HS DAC", "Line-In amp"
		};
		
		static const struct soc_enum twl6040_hs_enum[] = {
			SOC_ENUM_SINGLE(TWL6040_REG_HSLCTL, 5, ARRAY_SIZE(twl6040_hs_texts),
					twl6040_hs_texts),
			SOC_ENUM_SINGLE(TWL6040_REG_HSRCTL, 5, ARRAY_SIZE(twl6040_hs_texts),
					twl6040_hs_texts),
		};

		static const struct snd_kcontrol_new hsl_mux_controls =
			SOC_DAPM_ENUM("Route", twl6040_hs_enum[0]);
		
		static const struct snd_kcontrol_new hsr_mux_controls =
			SOC_DAPM_ENUM("Route", twl6040_hs_enum[1]);
		

2. 定义widget   

		static const struct snd_soc_dapm_widget twl6040_dapm_widgets[] = {
			SND_SOC_DAPM_MUX("HS Left Playback",
					SND_SOC_NOPM, 0, 0, &hsl_mux_controls),
			SND_SOC_DAPM_MUX("HS Right Playback",
					SND_SOC_NOPM, 0, 0, &hsr_mux_controls),

3. 定义连接路径   

		static const struct snd_soc_dapm_route intercon[] = {
			{"HS Left Playback", "HS DAC", "HSDAC Left"},
			{"HS Left Playback", "Line-In amp", "AFMAmpL"},

			{"HS Right Playback", "HS DAC", "HSDAC Right"},
			{"HS Right Playback", "Line-In amp", "AFMAmpR"},

4. 在codec驱动的probe注册这些widget和path

[ALSA声卡驱动中的DAPM详解之三：如何定义各种widget](http://blog.csdn.net/DroidPhone/article/details/12978287)