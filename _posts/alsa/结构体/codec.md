title: codec相关结构体
categories: ALSA
tags: [ALSA]
---
# control

## kcontrol
描述control的结构体

	struct snd_kcontrol_new {
		snd_ctl_elem_iface_t iface;	/* interface identifier */
		unsigned int device;		/* device/client number */
		unsigned int subdevice;		/* subdevice (substream) number */
		unsigned char *name;		/* ASCII name of item */
		unsigned int index;		/* index of item */
		unsigned int access;		/* access rights */
		unsigned int count;		/* count of same elements */
		snd_kcontrol_info_t *info;
		snd_kcontrol_get_t *get;
		snd_kcontrol_put_t *put;
		union {
			snd_kcontrol_tlv_rw_t *c;
			const unsigned int *p;
		} tlv;
		unsigned long private_value;
	};

## 方便定义kcontrol的宏
### 简单型控件
#### SOC_SINGLE
用来定义一个简单的control，例如volume。

	#define SOC_SINGLE_VALUE(xreg, xshift, xmax, xinvert) \
		((unsigned long)&(struct soc_mixer_control) \
		{.reg = xreg, .shift = xshift, .rshift = xshift, .max = xmax, \
		.platform_max = xmax, .invert = xinvert})

	#define SOC_SINGLE(xname, reg, shift, max, invert) \
	{	.iface = SNDRV_CTL_ELEM_IFACE_MIXER, .name = xname, \
		.info = snd_soc_info_volsw, .get = snd_soc_get_volsw,\
		.put = snd_soc_put_volsw, \
		.private_value =  SOC_SINGLE_VALUE(reg, shift, max, invert) }

#### SOC\_SINGLE\_TLV
主要定义有增益控制的空间，如音量控制器，用户空间可以通过对声卡的control设备发起ioctl来访问tlv字段

SNDRV\_CTL\_IOCTL\_TLV\_READ   
SNDRV\_CTL\_IOCTL\_TLV\_WRITE   
SNDRV\_CTL\_IOCTL\_TLV\_COMMAND   

主要描述寄存器设置的值和实际意义间的映射

	#define SOC_SINGLE_TLV(xname, reg, shift, max, invert, tlv_array) \
	{	.iface = SNDRV_CTL_ELEM_IFACE_MIXER, .name = xname, \
		.access = SNDRV_CTL_ELEM_ACCESS_TLV_READ |\
			 SNDRV_CTL_ELEM_ACCESS_READWRITE,\
		.tlv.p = (tlv_array), \
		.info = snd_soc_info_volsw, .get = snd_soc_get_volsw,\
		.put = snd_soc_put_volsw, \
		.private_value =  SOC_SINGLE_VALUE(reg, shift, max, invert) }

DECLARE\_TLV\_DB\_SCALE用于定义一个db值映射的tlv_array

	#define TLV_DB_SCALE_ITEM(min, step, mute)			\
		SNDRV_CTL_TLVT_DB_SCALE, 2 * sizeof(unsigned int),	\
		(min), ((step) & TLV_DB_SCALE_MASK) | ((mute) ? TLV_DB_SCALE_MUTE : 0)

	#define DECLARE_TLV_DB_SCALE(name, min, step, mute) \
		unsigned int name[] = { TLV_DB_SCALE_ITEM(min, step, mute) }


#### SOC\_DOUBLE
用来定义一个同时控制一个寄存器里两个相似变量的control，比如立体声左右声道

	#define SOC_DOUBLE(xname, xreg, shift_left, shift_right, xmax, xinvert) \
	{	.iface = SNDRV_CTL_ELEM_IFACE_MIXER, .name = (xname),\
		.info = snd_soc_info_volsw, .get = snd_soc_get_volsw, \
		.put = snd_soc_put_volsw, \
		.private_value = (unsigned long)&(struct soc_mixer_control) \
			{.reg = xreg, .shift = shift_left, .rshift = shift_right, \
			 .max = xmax, .platform_max = xmax, .invert = xinvert} }

#### SOC\_DOUBLE\_R
左右声道控制寄存器不一样
#### SOC\_DOUBLE\_TLV
带tlv
#### SOC\_DOUBLE\_R\_TLV 

### Mixer控件
由一个输出多个输入组成

	static const struct snd_kcontrol_new left_speaker_mixer[] = {  
		SOC_SINGLE("Input Switch", WM8993_SPEAKER_MIXER, 7, 1, 0),  
		SOC_SINGLE("IN1LP Switch", WM8993_SPEAKER_MIXER, 5, 1, 0),  
		SOC_SINGLE("Output Switch", WM8993_SPEAKER_MIXER, 3, 1, 0),  
		SOC_SINGLE("DAC Switch", WM8993_SPEAKER_MIXER, 6, 1, 0),  
	}; 

### Mux控件
mux控件用soc_enum描述

	struct soc_enum {
		unsigned short reg;
		unsigned short reg2;
		unsigned char shift_l;
		unsigned char shift_r;
		unsigned int max;
		unsigned int mask;
		const char * const *texts;
		char **dtexts;
		const unsigned int *values;
		void *dapm;
	};

#### 定义mux控件
1. 定义字符串和数值数组

		static const char *twl6040_headset_power_texts[] = {
			"Low-Power", "High-Performance",
		};

2. 利用soc_enum，描述寄存器

		static const struct soc_enum twl6040_headset_power_enum =
			SOC_ENUM_SINGLE_EXT(ARRAY_SIZE(twl6040_headset_power_texts),
					twl6040_headset_power_texts);

3. 利用asoc的宏，定义snd\_kcontrol\_new控件

		SOC_ENUM_EXT("Headset Power Mode", twl6040_headset_power_enum,
			twl6040_headset_power_get_enum,
			twl6040_headset_power_put_enum),

