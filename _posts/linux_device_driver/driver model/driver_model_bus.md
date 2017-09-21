title: bus
categories: Linux Device Driver Model
tags: [Linux Device Driver Model]
---
# Definition
See the kerneldoc for the struct bus_type

int bus\_register(struct bus_type * bus);

# Declaration
每个内核中的bus type(pci, usb, etc)应该定义一个静态的对象。需要初始化name域，可能初始化match回调。

	struct bus_type pci_bus_type = {
	       .name	= "pci",
	       .match	= pci_bus_match,
	};

在头文件中导出给driver

	extern struct bus_type pci_bus_type;

# Registration
当bus driver初始化时，调用bus_register。初始化其他的域并且插入到一个全局的表中。

# Callbacks
## match(): Attaching Drivers to Devices
当一个driver注册时，遍历bus上的设备列表，为每个没有driver与其关联的设备上调用match回调。

# Device and Driver Lists
struct devices和struct device_drivers的列表。

	int bus_for_each_dev(struct bus_type * bus, struct device * start, void * data,
		     int (*fn)(struct device *, void *));

	int bus_for_each_drv(struct bus_type * bus, struct device_driver * start, 
			     void * data, int (*fn)(struct device_driver *, void *));

遍历设备或驱动列表，每个都调用回调。所有列表访问都是同步的。

# sysfs
有一个顶级的bus目录，每个bus在bus目录中有单独的目录，里面包含两个默认的子目录

	/sys/bus/pci/
	|-- devices
	`-- drivers

bus上注册的driver在drivers下有相应的目录

	/sys/bus/pci/
	|-- devices
	`-- drivers
	    |-- Intel ICH
	    |-- Intel ICH Joystick
	    |-- agpgart
	    `-- e100

bus上发现的设备在devices目录下有链接到sys目录下的device里

# Exporting Attributes

	struct bus_attribute {
		struct attribute	attr;
		ssize_t (*show)(struct bus_type *, char * buf);
		ssize_t (*store)(struct bus_type *, const char * buf, size_t count);
	};

bus driver可能导出属性

	static BUS_ATTR(debug,0644,show_debug,store_debug);

然后通过函数添加进sysfs或者从sysfs中删除

	int bus_create_file(struct bus_type *, struct bus_attribute *);
	void bus_remove_file(struct bus_type *, struct bus_attribute *);