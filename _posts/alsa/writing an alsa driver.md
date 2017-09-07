title: writing an alsa driver
categories: ALSA
tags: [ALSA]
---

# Chapter 1 File Tree Structure

# Chapter 2 Basic Flow for PCI Drivers
PCI声卡最小的流程

- 定义PCI ID表
- probe()
- remove()
- 创建包含指向上面三个内容的指针的结构体pci_driver
- 创建init()函数调用pci_register_driver()注册pci_driver
- 创建exit()函数调用pci_unregister_driver()

## Driver Constructor
PCI Driver的实际构造函数是probe回调。在probe函数中

1. 检查并增加device index

		static int dev;
		....
		if (dev >= SNDRV_CARDS)
		        return -ENODEV;
		if (!enable[dev]) {
		        dev++;
		        return -ENOENT;
		}

2. 创建card instance

		struct snd_card *card;
		int err;
		....
		err = snd_card_new(&pci->dev, index[dev], id[dev], THIS_MODULE,
		                   0, &card);

3. 创建一个main component

		struct mychip *chip;
		....
		err = snd_mychip_create(card, pci, &chip);
		if (err < 0) {
		        snd_card_free(card);
		        return err;
		}

4. 设置driver id和名称

		strcpy(card->driver, "My Chip");
		strcpy(card->shortname, "My Own Chip 123");
		sprintf(card->longname, "%s at 0x%lx irq %i",
		        card->shortname, chip->ioport, chip->irq);

5. Create other components, such as mixer, MIDI, etc.

6. Register the card instance

		err = snd_card_register(card);
		if (err < 0) {
		        snd_card_free(card);
		        return err;
		}

7. Set the PCI driver data and return zero.

		pci_set_drvdata(pci, card);
		dev++;
		return 0;

# Chapter 3 Management of Cards and Components

## Card Instance
对于每个声卡， 必须分配一个card record。

一个card record是声卡的中心。管理声卡上的所有设备(组件)，例如PCM，mixer，MIDI，synthesizer等。Card record包含声卡ID和name字符串，管理proc文件，控制电源管理状态和热插拔。设备列表用来在最后正确的释放资源。

创建声卡instance(card record)

	struct snd_card *card;
	int err;
	err = snd_card_new(&pci->dev, index, id, module, extra_size, &card);

index是card-index number，id string， module pointer， extra-data size， 返回指向card instance的指针。

## Components
在ALSA驱动中，一个component用snd_device结构体表示。可以是PCM instance， control interface， raw MIDI interface等。每个instance有一个component entry。

创建component

	snd_device_new(card, SNDRV_DEV_XXX, chip, &ops);

card指向card instance的指针， device-level， chip是data指针，ops回调函数指针。device-level定义了component的类型以及注册和删除的顺序。

## chip-specific data
chip-specific相关的信息保存在chip-specific record中，例如

	struct mychip {
		...
	}

有两种分配chip record的方法

- 通过snd\_card\_create()函数 

		err = snd_card_create(index[dev], id[dev], THIS_MODULE, sizeof(struct mychip), &card);

	然后用下面方式访问

		struct mychip *chip = card->private_data;

- 分配额外的设备   
通过snd\_card\_create()(第四个参数设成0)

		struct snd_card *card;

		struct mychip *chip;
		err = snd_card_create(index[dev], id[dev], THIS_MODULE, 0, &card);
			.....
		chip = kzalloc(sizeof(*chip), GFP_KERNEL);
		
		struct mychip {
			struct snd_card *card;
			....
		};	

		chip->card = card;

		static struct snd_device_ops ops = {
			.dev_free = snd_mychip_dev_free,
		};
		....
		snd_device_new(card, SNDRV_DEV_LOWLEVEL, chip, &ops);

## registration and release
当所有component都分配了，通过snd\_card\_register()注册card instance。这个时候可以访问设备文件。

出错的话，调用snd\_card\_free()然后退出probe函数。

# Chapter 4. PCI Resource Management
PCI资源在probe函数中分配。

## 4.1 分配资源

# Chapter 5. PCM interface

## general
ALSA的PCM中间层只需要实现low-level function来访问硬件。

访问PCM层需要添加<sound/pcm.h>，添加<sound/pcm_params.h>来访问与hw_param相关的函数。

每个pcm instance对应一个pcm设备文件, 一个pcm instance由playback和capture stream组成, 每个stream包含一个到多个substream。

## PCM Constructor
一个PCM instance由snd\_pcm\_new()函数分配，第一个参数指向card instance，第二个参数ID字符串，第三个参数pcm的index，第四第五个参数是playbcak和capture中substreams的个数。

如果芯片支持多个playback或者capture时，在open/close函数中通过传入的struct snd\_pcm\_substream获得是哪个substream。

在pcm instance创建后，通过snd\_pcm\_set\_ops为每个pcm stream设置operators。operators定义

	static struct snd_pcm_ops snd_mychip_playback_ops = {
        .open =        snd_mychip_pcm_open,
        .close =       snd_mychip_pcm_close,
        .ioctl =       snd_pcm_lib_ioctl,
        .hw_params =   snd_mychip_pcm_hw_params,
        .hw_free =     snd_mychip_pcm_hw_free,
        .prepare =     snd_mychip_pcm_prepare,
        .trigger =     snd_mychip_pcm_trigger,
        .pointer =     snd_mychip_pcm_pointer,
	}

然后可以提前分配buffer

	snd_pcm_lib_preallocate_pages_for_all(pcm, SNDRV_DMA_TYPE_DEV,
                                      snd_dma_pci_data(chip->pci),
                                      64*1024, 64*1024);

设置额外的信息

	pcm->info_flags = SNDRV_PCM_INFO_HALF_DUPLEX;

## running pointer
当pcm substream打开时，一个pcm runtime instance被分配给substream，通过substream->runtime访问。runtime包含用来控制PCM的信息：hw\_params和sw\_params配置的拷贝，buffer指针，mmap record， spinlocks等。

## hardware description
struct snd\_pcm\_hardware包含硬件的信息，在pcm open callback中定义。

## PCM Configurations
PCM configuration保存在runtime instance中，在应用程序通过alsa-lib发送hw_params后。

## running status
The running status can be referred via runtime->status. This is the pointer to the struct snd_pcm_mmap_status record.

## private data
通常在pcm open callback中分配并保存在runtime->private_data。

## Operators
callback参数最少有一个指向struct snd_pcm_substream的指针，通过它可以获取chip record。

	int xxx() {
        struct mychip *chip = snd_pcm_substream_chip(substream);
        ....
	}

### pcm open callback
	static int snd_xxx_open(struct snd_pcm_substream *substream);
最少需要初始化runtime->hw record。

### pcm close callback
	static int snd_xxx_close(struct snd_pcm_substream *substream);

### ioctl callback
通常可以赋值snd_pcm_lib_ioctl()。

### hw_params callback
	static int snd_xxx_hw_params(struct snd_pcm_substream *substream, struct snd_pcm_hw_params *hw_params);
在应用程序设置hw_params时调用，参数通过params_xxx()宏获取。

### hw_free callback
	static int snd_xxx_hw_free(struct snd_pcm_substream *substream);

### prepare callback
	static int snd_xxx_prepare(struct snd_pcm_substream *substream);

### trigger callback
当pcm被start，stop或者pause时调用

	static int snd_xxx_trigger(struct snd_pcm_substream *substream, int cmd);

### pointer callback
当PCM middle layer查询buffer位置时调用。

	static snd_pcm_uframes_t snd_xxx_pointer(struct snd_pcm_substream *substream)

## PCM interrupt handler
PCM终端处理函数更新buffer position并且通知PCM中间层当buffer位置超过规定的period size时。调用snd_pcm_period_elapsed()函数。

# Control interface
## general
## 定义controls
创建一个control，需要定义三种callback：info，get和put。然后定义一个struct snd\_kcontrol\_new record。

	static struct snd_kcontrol_new my_control = {
        .iface = SNDRV_CTL_ELEM_IFACE_MIXER,
        .name = "PCM Playback Switch",
        .index = 0,
        .access = SNDRV_CTL_ELEM_ACCESS_READWRITE,
        .private_value = 0xffff,
        .info = my_control_info,
        .get = my_control_get,
        .put = my_control_put
	};

iface指定control的类型，SNDRV_CTL_ELEM_IFACE_XXX,通常是MIXER。如果是global control的话使用CARD。如果与特定的device相关的话使用HWDEP, PCM, RAWMIDI, TIMER, or SEQUENCER并且通过device和subdevice指定device number。

## Control Names
用"SOURCE DIRECTION FUNCTION"三部分定义。

SOURCE，指定control的source。

DIRECTION，“Playback”, “Capture”, “Bypass Playback” and “Bypass Capture”中的一个。

FUNCTION，“Switch”, “Volume” and “Route”中的一个。

## Global capture and playback
“Capture Source”, “Capture Switch” and “Capture Volume”
“Playback Switch” and “Playback Volume”

## Tone-controls
“Tone Control - XXX”

## 3D controls
## Mic boost

## Access Flags

## Control Callbacks
### info callback
获取control的信息。
### get callback
### put callback

## Control Constructor
通过snd_ctl_new1() and snd_ctl_add()创建control。

## Metadata
提供mixer control的db值，使用DECLARE_TLV_xxx宏。

# Buffer and memory management
## Buffer Types

[111](https://www.alsa-project.org/main/index.php/ALSA_Driver_Documentation)   
[writing an alsa driver](https://www.kernel.org/doc/html/v4.10/sound/kernel-api/writing-an-alsa-driver.html)