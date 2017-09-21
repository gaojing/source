title: Device Drivers
categories: Linux Device Driver Model
tags: [Linux Device Driver Model]
---
See the kerneldoc for the struct device_driver.

# Allocation
Device drivers是静态分配的，对于多个driver支持的设备，struct device_driver代表了整个driver。

# Initialization
driver必须初始化name和bus域。

# Declaration

# Registration
	int driver_register(struct device_driver * drv);

