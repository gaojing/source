title: Device Classes
categories: Linux Device Driver Model
tags: [Linux Device Driver Model]
---
# Introduction
device class描述一种类型的device，想audio或者network。

# Programming Interface

	typedef int (*devclass_add)(struct device *);
	typedef void (*devclass_remove)(struct device *);

See the kerneldoc for the struct class.

通常device class定义像下面这样

	struct device_class input_devclass = {
	        .name		= "input",
	        .add_device	= input_add_device,
		.remove_device	= input_remove_device,
	};

注册

	int devclass_register(struct device_class * cls);
	void devclass_unregister(struct device_class * cls);

# Devices

# Device Drivers

# sysfs directory structure
有个top level的class目录


