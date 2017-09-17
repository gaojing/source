title: ASOC DAPM - kcontrol
categories: ALSA
tags: [ALSA]
---
DAPM为了使基于linux的嵌入式设备的音频子系统，工作在最小功耗状态。

# snd\_kcontrol\_new结构
在ALSA中，一个control设备由snd\_kcontrol\_new结构定义。其中有get，put，private_value主要的字段。

# 简单型控件
## SOC_SINGLE
通过SOC_SINGLE宏，创建简单控件，例如一个开关或者一个数值变量。

	#define SOC_SINGLE(xname, reg, shift, max, invert) \
	{	.iface = SNDRV_CTL_ELEM_IFACE_MIXER, .name = xname, \
		.info = snd_soc_info_volsw, .get = snd_soc_get_volsw,\
		.put = snd_soc_put_volsw, \
		.private_value =  SOC_SINGLE_VALUE(reg, shift, max, invert) }

对应的参数name为控件的名字，reg寄存器地址，shift控制为在寄存器的位移，max可设置的最大值，invert逻辑取反。

	#define SOC_SINGLE_VALUE(xreg, xshift, xmax, xinvert) \
	((unsigned long)&(struct soc_mixer_control) \
	{.reg = xreg, .shift = xshift, .rshift = xshift, .max = xmax, \
	.platform_max = xmax, .invert = xinvert})

SOC\_SINGLE\_VALUE定义了一个soc\_mixer\_control赋值给private\_value。

	struct soc_mixer_control {
		int min, max, platform_max;
		unsigned int reg, rreg, shift, rshift, invert;
	};

## SOC\_SINGLE\_TLV

	#define SOC_SINGLE_TLV(xname, reg, shift, max, invert, tlv_array) \
	{	.iface = SNDRV_CTL_ELEM_IFACE_MIXER, .name = xname, \
		.access = SNDRV_CTL_ELEM_ACCESS_TLV_READ |\
			 SNDRV_CTL_ELEM_ACCESS_READWRITE,\
		.tlv.p = (tlv_array), \
		.info = snd_soc_info_volsw, .get = snd_soc_get_volsw,\
		.put = snd_soc_put_volsw, \
		.private_value =  SOC_SINGLE_VALUE(reg, shift, max, invert) }

增加了tvl.p字段的赋值，用户控件可以对声卡的control设备调用ioctl来访问数组：

	SNDRV_CTL_IOCTL_TLV_READ
    SNDRV_CTL_IOCTL_TLV_WRITE
    SNDRV_CTL_IOCTL_TLV_COMMAND

tlv_array数组代表寄存器设定值和实际意义之间的映射，例如音量设定值和db间的映射。

	static DECLARE_TLV_DB_SCALE(hs_tlv, -3000, 200, 0);
	
	SOC_DOUBLE_EXT_TLV("Headset Playback Volume",
		TWL6040_REG_HSGAIN, 0, 4, 0xF, 1,
		twl6040_get_volsw, twl6040_put_volsw, hs_tlv),

DECLARE_TLV_DB_SCALE定义db值映射的tlv_array，例子表明该tlv是SNDRV_CTL_TLVT_DB_SCALE类型，寄存器最小值对应-30db，寄存器每增加一个值，db值加2。

## SOC\_DOUBLE
可以控制同一个寄存器的两个变量

	#define SOC_DOUBLE(xname, xreg, shift_left, shift_right, xmax, xinvert) \
	{	.iface = SNDRV_CTL_ELEM_IFACE_MIXER, .name = (xname),\
		.info = snd_soc_info_volsw, .get = snd_soc_get_volsw, \
		.put = snd_soc_put_volsw, \
		.private_value = (unsigned long)&(struct soc_mixer_control) \
			{.reg = xreg, .shift = shift_left, .rshift = shift_right, \
			 .max = xmax, .platform_max = xmax, .invert = xinvert} }

## SOC\_DOUBLE\_R
左右声道寄存器不一样的情况

## SOC\_DOUBLE\_TLV

## SOC\_DOUBLE\_R\_TLV

# Mixer控件
由一个输出多个输入组成

	static const struct snd_kcontrol_new left_speaker_mixer[] = {  
		SOC_SINGLE("Input Switch", WM8993_SPEAKER_MIXER, 7, 1, 0),  
		SOC_SINGLE("IN1LP Switch", WM8993_SPEAKER_MIXER, 5, 1, 0),  
		SOC_SINGLE("Output Switch", WM8993_SPEAKER_MIXER, 3, 1, 0),  
		SOC_SINGLE("DAC Switch", WM8993_SPEAKER_MIXER, 6, 1, 0),  
	};  

# Mux控件
与mixer类似，但mux输入只有一个能被选中。ASOC用soc_enum来描述mux控件寄存器信息。

	struct soc_enum {  
        unsigned short reg;  
        unsigned short reg2;  
        unsigned char shift_l;  
        unsigned char shift_r;  
        unsigned int max;  
        unsigned int mask;  
        const char * const *texts;  
        const unsigned int *values;  
	};  

其中texts指向用于描述输入端的，values指向寄存器可以选择的值，每个值对应一个输入端。

## 定义mux控件
1. 定义字符串和数值数组

		static const char *twl6040_hs_texts[] = {
			"Off", "HS DAC", "Line-In amp"
		};

2. 利用soc_enum，描述寄存器

		static const struct soc_enum twl6040_hs_enum[] = {
			SOC_ENUM_SINGLE(TWL6040_REG_HSLCTL, 5, ARRAY_SIZE(twl6040_hs_texts),
					twl6040_hs_texts),
			SOC_ENUM_SINGLE(TWL6040_REG_HSRCTL, 5, ARRAY_SIZE(twl6040_hs_texts),
					twl6040_hs_texts),
		};

3. 利用asoc的宏，定义snd\_kcontrol\_new控件

		static const struct snd_kcontrol_new hsl_mux_controls =
		SOC_DAPM_ENUM("Route", twl6040_hs_enum[0]);

		#define SOC_DAPM_ENUM(xname, xenum) \
		{	.iface = SNDRV_CTL_ELEM_IFACE_MIXER, .name = xname, \
			.info = snd_soc_info_enum_double, \
		 	.get = snd_soc_dapm_get_enum_double, \
		 	.put = snd_soc_dapm_put_enum_double, \
		  	.private_value = (unsigned long)&xenum }

# 参考
[ALSA声卡驱动中的DAPM详解之一：kcontrol](http://blog.csdn.net/DroidPhone/article/details/12793293)

