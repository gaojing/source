title: 深入理解Android内核设计思想 第六章 进程间通信 binder
categories: 深入理解Android内核设计思想
tags: [深入理解Android内核设计思想]
---
Binder
- Binder驱动
- Service Manager
- Binder Client
- Binder Server

Binder的各个组成元素和TCP/IP网络相似
- Binder驱动  -- 路由器
- Service Manager -- DNS
- Binder Client -- 客户端
- Binder Server -- 服务器

## 6.1 智能指针

### 6.1.3 弱指针
主要解决两个对象相互引用导致无法释放的情况。

## 6.3 Binder驱动与协议
Binder驱动会将自己注册成一个misc device，并向上层提供一个/dev/binder节点。

### 6.3.1 打开binder驱动-binder_open
上层进程访问binder驱动时，先要打开/dev/binder节点，体现在binder_open中

### 6.3.2 binder_mmap
上层用户调用mmap系统调用时最终调用binder_mmap。

## 6.4 DNS服务器-ServiceManager[Binder Server]

### 6.4.1 ServiceManager的启动
在init解析init.rc时启动。ServiceManager用C/C++编写，在/frameworks/native/cmds/servicemanager目录中。

### 6.4.2 ServiceManager的构建
main函数

	int main(int argc, char **argv)
	{
	    struct binder_state *bs;
	    void *svcmgr = BINDER_SERVICE_MANAGER;
	
	    bs = binder_open(128*1024);
	
	    if (binder_become_context_manager(bs)) {
	        ALOGE("cannot become context manager (%s)\n", strerror(errno));
	        return -1;
	    }
	
	    svcmgr_handle = svcmgr;
	    binder_loop(bs, svcmgr_handler);
	    return 0;
	}

- 打开binder设备，初始化
- 设置为binder大管家
- 进入主循环

		void binder_loop(struct binder_state *bs, binder_handler func)
		{
		    int res;
		    struct binder_write_read bwr;
		    unsigned readbuf[32];
		
		    bwr.write_size = 0;
		    bwr.write_consumed = 0;
		    bwr.write_buffer = 0;
		    
		    readbuf[0] = BC_ENTER_LOOPER;
		    binder_write(bs, readbuf, sizeof(unsigned));
		
		    for (;;) {
		        bwr.read_size = sizeof(readbuf);
		        bwr.read_consumed = 0;
		        bwr.read_buffer = (unsigned) readbuf;
		
		        res = ioctl(bs->fd, BINDER_WRITE_READ, &bwr);
		
		        if (res < 0) {
		            ALOGE("binder_loop: ioctl failed (%s)\n", strerror(errno));
		            break;
		        }
		
		        res = binder_parse(bs, 0, readbuf, bwr.read_consumed, func);
		        if (res == 0) {
		            ALOGE("binder_loop: unexpected reply?!\n");
		            break;
		        }
		        if (res < 0) {
		            ALOGE("binder_loop: io error %d %s\n", res, strerror(errno));
		            break;
		        }
		    }
		}


