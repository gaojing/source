title: writing-clients
categories: linux device driver
tags: [linux device driver]
---
# writing-clients
描述了为I2C和SMBUS设备编写驱动，linux作为host/master设备。

# The driver structure
实现一个driver structure，然后通过它实例化client。driver structure包含通过的access函数，应该zero-initialized。client structure包含设备相关的数据，例如driver model device node和它的i2c地址。

	static struct i2c_device_id foo_idtable[] = {
		{ "foo", my_id_for_foo },
		{ "bar", my_id_for_bar },
		{ }
	};
	
	MODULE_DEVICE_TABLE(i2c, foo_idtable);
	
	static struct i2c_driver foo_driver = {
		.driver = {
			.name	= "foo",
			.pm	= &foo_pm_ops,	/* optional */
		},
	
		.id_table	= foo_idtable,
		.probe		= foo_probe,
		.remove		= foo_remove,
		/* if device autodetection is needed: */
		.class		= I2C_CLASS_SOMETHING,
		.detect		= foo_detect,
		.address_list	= normal_i2c,
	
		.shutdown	= foo_shutdown,	/* optional */
		.command	= foo_command,	/* optional, deprecated */
	}

# Extra client data
每个client structure包含一个data field用来保存device-specific数据。

	/* store the value */
	void i2c_set_clientdata(struct i2c_client *client, void *data);

	/* retrieve the value */
	void *i2c_get_clientdata(const struct i2c_client *client);

# Accessing the client
封装读写

	int foo_read_value(struct i2c_client *client, u8 reg)
	{
		if (reg < 0x10)	/* byte-sized register */
			return i2c_smbus_read_byte_data(client, reg);
		else		/* word-sized register */
			return i2c_smbus_read_word_data(client, reg);
	}
	
	int foo_write_value(struct i2c_client *client, u8 reg, u16 value)
	{
		if (reg == 0x10)	/* Impossible to write - driver error! */
			return -EINVAL;
		else if (reg < 0x10)	/* byte-sized register */
			return i2c_smbus_write_byte_data(client, reg, value);
		else			/* word-sized register */
			return i2c_smbus_write_word_data(client, reg, value);
	}

# Probing and attaching
# Device/Driver Binding
In linux，提供probe函数用来bind device，remove函数来unbind

	static int foo_probe(struct i2c_client *client,
			     const struct i2c_device_id *id);
	static int foo_remove(struct i2c_client *client);

probe函数在id_table name成员与device name匹配时调用。

# Device Creation
# Device Detection
# Device Deletion
	i2c_unregister_device()

# Initializing the driver
	
	static int __init foo_init(void)
	{
		return i2c_add_driver(&foo_driver);
	}
	module_init(foo_init);
	
	static void __exit foo_cleanup(void)
	{
		i2c_del_driver(&foo_driver);
	}
	module_exit(foo_cleanup);

# Sending and receiving
## Plain I2C communication
	int i2c_master_send(struct i2c_client *client, const char *buf,
			    int count);
	int i2c_master_recv(struct i2c_client *client, char *buf, int count);
	int i2c_transfer(struct i2c_adapter *adap, struct i2c_msg *msg,
			 int num);