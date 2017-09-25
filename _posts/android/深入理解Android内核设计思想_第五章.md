title: 深入理解Android内核设计思想 第五章
categories: 深入理解Android内核设计思想
tags: [深入理解Android内核设计思想]
---
# 5.1 Android进程和线程
activity创建的主线程是ActivityThread   
对于同一个AndroidManifest.xml中定义的四大组件，都运行于同一个进程中，均由主线程来处理事件。

# 5.2 Handler, MessageQueue, Runnable与Looper
## Handler
代码路径: frameworks/base/core/java/android/os/Handler.java   

	public class Handler {
		final MessageQueue mQueue;
		final Looper mLooper;
		final Callback mCallback;

- 每个Thread对应一个Looper
- 每个Looper对应一个MessageQueue;
- 每个MessageQueue有N个Message
- 每个Message中最多指定一个Handler来处理事件

Handler有两个方面的作用
- 处理Message
- 将某个Message压入MessageQueue中

处理Message

	public void dispatchMessage(Message msg)
	public void handleMessage(Message msg)

Looper从MessageQueue中取出一个Message后，首先调用Handler.dispatchMessage进行消息派发

- Message.callback(Runnable对象)是否为空
- Handler.mCallback是否为空
- Handler.handlerMessage

将Message压入MessageQueue

	final boolean post(Runnable r);
	final boolean postAtTime(Runnable r, long uptimeMillis);

	final boolean sendEmptyMessage(int what)
	...

## MessageQueue
源码路径: frameworks/base/core/java/android/os
### 新建队列
由构造函数和本地方法nativeInit组成
### 元素入列
	final boolean enqueueMessage(Message msg, long when)

## Looper

## UI主线程 ActivityThread

# 5.4 Thread类


