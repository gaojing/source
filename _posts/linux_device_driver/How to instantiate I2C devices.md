title: How to instantiate I2C devices
categories: linux device driver
tags: [linux device driver]
---
# How to instantiate I2C devices
对于I2C设备，软件必须知道设备的I2C地址，连接到哪个I2C总线上，必须显示的实例化I2C设备。

# Method 1a: Declare the I2C devices by bus number
对于embedded system来说一般I2C是系统总线，I2C总线的地址提前知道，可以直接定义I2C设备。通过定义struct i2c_board_info数组，然后调用i2c_register_board_info()。

例如galaxy nexus

	static struct i2c_board_info __initdata tuna_i2c1_boardinfo[] = {
		{
			I2C_BOARD_INFO("twl6030", 0x48),
			.flags = I2C_CLIENT_WAKE,
			.irq = OMAP44XX_IRQ_SYS_1N,
			.platform_data = &tuna_twldata,
		},
	};

	tuna_i2c_init
	
	omap_register_i2c_bus(1, 400, tuna_i2c1_boardinfo,
			      ARRAY_SIZE(tuna_i2c1_boardinfo));

	int __init omap_register_i2c_bus(int bus_id, u32 clkrate,
			  struct i2c_board_info const *info,
			  unsigned len)
	{
		int err;
	
		BUG_ON(bus_id < 1 || bus_id > omap_i2c_nr_ports());
	
		if (info) {
			err = i2c_register_board_info(bus_id, info, len);
			if (err)
				return err;
		}
	
		if (!i2c_pdata[bus_id - 1].clkrate)
			i2c_pdata[bus_id - 1].clkrate = clkrate;
	
		i2c_pdata[bus_id - 1].clkrate &= ~OMAP_I2C_CMDLINE_SETUP;
	
		return omap_i2c_add_bus(bus_id);
	}

定义了一个I2C设备，包括地址和platform_data，当对应的总线被注册时，i2c-core会实例化i2c设备。