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

	主循环:
	
	- 从消息队列中读取消息
	- 如果消息是退出命令，结束循环，消息为空，继续读取或者等段时间再读取，负责根据具体情况进行处理。
	- 循环知道退出。

	binder_parse解析消息

	- 1. BR_TRANSACTION   
	对BR\_TRANSACTION的处理主要由func来完成，将结果返回给binder驱动。   
	
			- 注册   
			当一个binder server创建后，他们要将[名称，Binder句柄】告诉SM进行备案。
			- 查询   
			应用程序向binder发送查询请求。
			- 其他信息查询
		
### 6.4.3 获取ServiceManager服务  -- 设计思考
访问SM(Binder Server)的流程

- 打开binder设备
- 执行mmap
- 通过binder驱动向SM发送请求
- 获得结果

设计

- 发请求可能是APK应用，需提供java接口
- 封装流程方便更好的使用
- 每个进程只允许打开一次binder设备，做一次内存映射，防止消耗越来越多的系统资源。

#### ProcessState和IPCThreadState
创建一个类专门来管理每个应用进程的binder操作，ProcessState.   
与Binder驱动进行实际命令通信的是IPCThreadState。

#### proxy
ServiceManagerProxy和ServiceManager。

#### IBinder和BpBinder
BpBinder，native层IBinder接口类的具体实现类。

#### ProcessState和IPCThreadState
实现ProcessState：

- 一个进程只有一个ProcessState实例
- 向上层提供IPC服务
- 与IPCThreadState分工

 
			
	
