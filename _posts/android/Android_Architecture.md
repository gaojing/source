title: Android_Architecture
categories: Android
tags: [Android]
---

# Table of Contents
- [1. Overview](#section1)
- [2. HAL](#section2)
	- [2.1 HAL modules](#section2.1)
	- [2.2 HAL devices](#section2.2)
	- [2.3 Building HAL modules](#section2.3)
- [3. HAL Types](#section3)
- [4. Treble](#section4)
- [5. Kernel](#section5)
	- [5.1 Overview](#section5.1)
- [6. HIDL](#section6)
	- [6.1 HIDL Overview](#section6.1)
	- [6.2 Interfaces & Packages](#section6.2)
	- [6.3 Interface Hashing](#section6.3)
- [7. HIDL C++](#section7)
	- [7.1 Overview](#section7.1)
	- [7.2 Packages](#section7.2)
	- [7.3 Interfaces](#section7.3)

<a name="section1"></a>
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

<a name="section2"></a>
# 2. HAL
![](/images/ape_fwk_hal.png)

HAL实现一般编译到共享库里(.so文件)，android没有强制HAL实现和设备驱动之间的标准交互。   
为了保证HALs包含定义好的结构，每个硬件相关的HAL interface在hardware/libhardware/include/hardware/hardware.h中定义了属性。   
一个HAL interface包含两个组件：modules和devices。

<a name="section2.1"></a>
## 2.1 HAL modules
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

<a name="section2.2"></a>
## 2.2 HAL devices
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

<a name="section2.3"></a>
## 2.3 Building HAL modules
HAL实现被编译进.so文件，需要时被动态链接进android。通过为每个HAL实现创建Android.mk文件。   
通常库名字遵循<module_type>.<device_name>规则。

<a name="section3"></a>
# 3. HAL Types
Android O重新架构了android os底层，运行Android O的设备必须支持binderized和passthrough HALs。

- Binderized HALs：用HAL interface definition language（HIDL）描述。
- Passthrough HALs:HIDL-wrapped onventional or legacy HAL。包装已有的HALs，serve HAL in binderized and same-process（passthrough）模式。

<a name="section4"></a>
# 4. Treble
重新架构了Android OS framework，方便手机厂商更快的更新android。

## About Android updates
Treble从Android OS framework中分离了vendor implementation， 通过新添加的vendor interface。

<a name="section5"></a>
# 5. Kernel

<a name="section5.1"></a>
## 5.1 Overview

## Modular Kernel Requirement
在Android O中，内核被分为SOC，device和board-specific。

### Loadable kernel modules

<a name="section6"></a>
# 6. HIDL
HAL interface definition language or HIDL is an interface description language (IDL) to specify the interface between a HAL and its users.   
HIDL用于IPC。   
HIDL specifies data structures and method signatures, organized in interfaces (similar to a class) that are collected into packages.

<a name="section6.1"></a>
## 6.1 HIDL Overview

### HIDL设计
HIDL的目标是framework可以单独被替换不需要编译HALs。由Vendors或者SOC厂商编译的HALs被放在/vendor分区，framework在单独的分区。

<a name="section6.2"></a>
## 6.2 Interfaces & Packages

### Packages

### Interface definition
除了types.hal外，其他的hal文件定义interface.

	interface IBar extends IFoo { // IFoo is another interface
	    // embedded types
	    struct MyStruct {/*...*/};
	
	    // interface methods
	    create(int32_t id) generates (MyStruct s);
	    close();
	};

所有没有extend的interface从android.hidl.base@1.0::IBase继承

### Importing

### Interface inheritance


<a name="section6.3"></a>
## 6.3 Interface Hashing

<a name="section7"></a>
## 7 HIDL C++


<a name="section7.1"></a>
## 7.1 Overview

### Creating the HAL client
在makefile中添加hal库

- Make: LOCAL_SHARED_LIBRARIES += android.hardware.nfc@1.0
- Soong: shared_libs: [ …, android.hardware.nfc@1.0 ]

然后include头文件使用

	#include <android/hardware/nfc/1.0/IFoo.h>
	…
	// in code:
	sp<IFoo> client = IFoo::getService();
	client->doThing();

### Creating the HAL server
创建实现hal必要的文件

	PACKAGE=android.hardware.nfc@1.0
	LOC=hardware/interfaces/nfc/1.0/default/
	m -j hidl-gen
	hidl-gen -o $LOC -Lc++-impl -randroid.hardware:hardware/interfaces \
	    -randroid.hidl:system/libhidl/transport $PACKAGE
	hidl-gen -o $LOC -Landroidbp-impl -randroid.hardware:hardware/interfaces \
	    -randroid.hidl:system/libhidl/transport $PACKAGE

然后设置daemon，示例代码

支持passthrough

	#include <hidl/LegacySupport.h>

	int main(int /* argc */, char* /* argv */ []) {
	    return defaultPassthroughServiceImplementation<INfc>("nfc");
	}

pure binderized

	int main(int /* argc */, char* /* argv */ []) {
	    // This function must be called before you join to ensure the proper
	    // number of threads are created. The threadpool will never exceed
	    // size one because of this call.
	    ::android::hardware::configureRpcThreadpool(1 /*threads*/, true /*willJoin*/);
	
	    sp nfc = new Nfc();
	    const status_t status = nfc->registerAsService();
	    if (status != ::android::OK) {
	        return 1; // or handle error
	    }
	
	    // Adds this thread to the threadpool, resulting in one total
	    // thread in the threadpool. We could also do other things, but
	    // would have to specify 'false' to willJoin in configureRpcThreadpool.
	    ::android::hardware::joinRpcThreadpool();
	    return 1; // joinRpcThreadpool should never return
	}

<a name="section7.2"></a>
## 7.2 Packages

<a name="section7.3"></a>
## 7.3 Interfaces
