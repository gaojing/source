title: Debugging Native Android Platform Code
categories: Android
tags: [Android]
---
# Table of Contents
- [1. Overview](#section1)
	- [1.1 Crash dumps](#section1.1)
	- [1.2 Getting a stack trace/tombstone from a running process](#section1.2)
- [2. Diagnosing Native Crashes](#section2)
	- [2.1 AudioPolicyService构造函数流程](#section2.1)
		- [2.1.1 加载policy module](#section2.1.1)

<a name="section1"></a>
# 1. Overview

<a name="section1.1"></a>
## 1.1 Crash dumps
当一个动态链接的可执行文件启动时，会在crash的时候写一个详细的tombstone文件。

可以通过stack获取backtrace的信息。

<a name="section1.2"></a>
## 1.2 Getting a stack trace/tombstone from a running process
可以通过debuggerd dump一个运行中的进程的信息。

<a name="section2"></a>
# 2. Diagnosing Native Crashes


