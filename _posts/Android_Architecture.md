title: Android_Architecture
categories: Android
tags: [Android]
---

# 1. Overview
![](/images/ape_fwk_all.png)

## Application framework
主要供应用程序开发者使用。对于硬件开发人员来说，一些API直接映射到HAL的interface上。

## Binder IPC
Binder IPC允许application framwork调用android system services代码。

## Hardware abstraction layer(HAL)
HAL定义了一套标准的interface供硬件厂商去实现。

## Linux kernel
开发设备驱动和实现linux驱动类似。android使用的linux kernel增加了wake locks，binder ipc driver和一些其他的功能。

# 2. HAL
![](/images/ape_fwk_hal.png)

HAL实现一般编译到共享库里(.so文件)，android没有强制HAL实现和设备驱动之间的标准交互。   
为了保证HALs包含定义好的结构，每个硬件相关的HAL interface在hardware/libhardware/include/hardware/hardware.h中定义了属性。   
一个HAL interface包含两个组件：modules和devices。

## HAL modules
一个module包含打包好的HAL实现，保存在共享库里(.so文件)。   
hardware/libhardware/include/hardware/hardware.h中的hw_module_结构体代表了一个module，包含version，name等metadata。  
hw\_module\_t包含一个指向hw\_module\_methods\_t的指针，hw\_module\_methods\_t中包含一个指向open函数的指针。open函数用来初始化与硬件的通信。
每个具体的硬件通常扩展hw\_module\_t，例如camera HAL

	typedef struct camera_module {
    	hw_module_t common;
    	int (*get_number_of_cameras)(void);
    	int (*get_camera_info)(int camera_id, struct camera_info *info);
	} camera_module_t;

当实现HAL创建module结构体时，必须命名为HAL\_MODULE\_INFO\_SYM

	struct audio_module HAL_MODULE_INFO_SYM = {
	    .common = {
	        .tag = HARDWARE_MODULE_TAG,
	        .module_api_version = AUDIO_MODULE_API_VERSION_0_1,
	        .hal_api_version = HARDWARE_HAL_API_VERSION,
	        .id = AUDIO_HARDWARE_MODULE_ID,
	        .name = "NVIDIA Tegra Audio HAL",
	        .author = "The Android Open Source Project",
	        .methods = &hal_module_methods,
	    },
	};

## HAL devices
devices是硬件的抽象。
一个device有hw\_device\_t结构体表达。每种类型的设备扩展了hw\_device\_t结构体。

	struct audio_hw_device {
    struct hw_device_t common;
	
	    /**
	     * used by audio flinger to enumerate what devices are supported by
	     * each audio_hw_device implementation.
	     *
	     * Return value is a bitmask of 1 or more values of audio_devices_t
	     */
	    uint32_t (*get_supported_devices)(const struct audio_hw_device *dev);
	  ...
	};
	typedef struct audio_hw_device audio_hw_device_t;

[HAL Reference](https://source.android.com/reference/hal/)

## Building HAL modules
HAL实现被编译进.so文件，需要时被动态链接进android。通过为每个HAL实现创建Android.mk文件。   
通常库名字遵循<module_type>.<device_name>规则。

# 3. HAL Types
Android O重新架构了android os底层，运行Android O的设备必须支持binderized和passthrough HALs。

- Binderized HALs：用HAL interface definition language（HIDL）描述。
- Passthrough HALs:HIDL-wrapped onventional or legacy HAL。包装已有的HALs，serve HAL in binderized and same-process（passthrough）模式。

# 4. Treble
重新架构了Android OS framework，方便手机厂商更快的更新android。

## About Android updates
Treble从Android OS framework中分离了vendor implementation， 通过新添加的vendor interface。

# 5. Kernel

## Linux kernel development

## Modular Kernel Requirement
在Android O中，内核被分为SOC，device和board-specific。

### Loadable kernel modules
 
