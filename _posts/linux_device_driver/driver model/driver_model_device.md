title: The Basic Device Structure
categories: Linux Device Driver Model
tags: [Linux Device Driver Model]
---
See the kerneldoc for the struct device.

# Programming Interface
发现device的bus driver通过函数注册device到core中

	int device_register(struct device * dev);

bus应该初始化下面域

	- parent
    - name
    - bus_id
    - bus

当device的reference count变成0时device从core总删除

	struct device * get_device(struct device * dev);
	void put_device(struct device * dev);

lock

	void lock_device(struct device * dev);
	void unlock_device(struct device * dev);

# Attributes
	struct device_attribute {
		struct attribute	attr;
		ssize_t (*show)(struct device *dev, struct device_attribute *attr,
				char *buf);
		ssize_t (*store)(struct device *dev, struct device_attribute *attr,
				 const char *buf, size_t count);
	};

device的attribute通过device driver导出到sysfs

宏

	#define DEVICE_ATTR(name,mode,show,store)


