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

# Chapter 3 Management of Cards andComponents

## Card Instance
对于每个声卡， 必须分配一个card record。

一个card record是声卡的中心。管理声卡上的所有设备(组件)，例如PCM，mixer，MIDI，synthesizer等。Card record包含声卡ID和name字符串，管理proc文件，控制电源管理状态和热插拔。设备列表用来在最后正确的释放资源。

创建声卡instance(card record)

	struct snd_card *card;
	int err;
	err = snd_card_create(index, id, module, extra_size, &card);

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

- 通过snd_card_create()函数 

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




[111](https://www.alsa-project.org/main/index.php/ALSA_Driver_Documentation)