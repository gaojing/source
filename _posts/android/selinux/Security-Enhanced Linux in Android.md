title: Security-Enhanced Linux in Android
categories: Android
tags: [Android]
---
# Table of Contents
- [1. Overview](#section1)
	- [1.1 Background](#section1.1)
- [2. Selinux Concepts](#section2)
	- [2.1 Mandatory access control](#section2.1)
	- [2.2 Enforcement levels](#section2.2)
	- [2.3 Labels, rules and domains](#section2.3)
- [3. Implementing SELinux](#section3)
	- [3.1 Summary of steps](#section3.1)
	- [3.2 Key files](#section3.2)
	- [3.3 Steps in detail](#section3.3)
- [4. Customizing SELinux](#section4)
- [5. Validating SELinux](#section5)
	- [5.1 Reading denials](#section5.1)

<a name="section1"></a>
# 1. Overview

<a name="section1.1"></a>
## 1.1 Background
Selinux可以工作在两种模式下

- Permissive mode, 违反权限只被log记录
- Enforcing mode, 违反权限同时被log记录以及执行

Selinux还支持per-domain permissive mode，在这种模式下特定domain的process工作在permissive模式下，而其他的工作在enforcing mode。

<a name="section2"></a>
# 2. Selinux Concepts

<a name="section2.1"></a>
## 2.1 Mandatory access control
Selinux 是一个mandatory access control(MAC)系统。

<a name="section2.2"></a>
## 2.2 Enforcement levels

- Permissive
- Enforcing

<a name="section2.3"></a>
## 2.3 Labels, rules and domains
Selinux通过label来匹配action和policy，lable格式user:role:type:mls_level。

policy rules格式allow domains types:classes permissions;

- Domain: 进程的label
- Type: object的label
- Class: object的类型
- Permission: operation

rule的格式，
	
	RULE_VARIANT SOURCE_TYPES TARGET_TYPES : CLASSES PERMISSIONS


<a name="section3"></a>
# 3. Implementing SELinux

<a name="section3.1"></a>
## 3.1 Summary of steps
1. 在内核和配置中增加selinux支持
2. 从init开始为每个服务(进程或者daemon)分配domain
3. 通过下面方法识别service 
	- init.<device\>.rc文件
	- 在dmesg寻找提示信息
	- 通过ps -Z | grep init检查哪些service是通过init domain运行的
4. 给新的进程打label
5. 定义rule

<a name="section3.2"></a>
## 3.2 Key files
kernel和system/sepolicy中的包含selinux相关的文件。   
设备相关的policy文件应该添加到/device/manufacturer/device-name/sepolicy中。

下面是实现selinux必须的文件

- 新的selinux source(*.te)文件，放到/device/manufacturer/device-name/sepolicy下面，在启动定义domain和label
- 更新BoardConfig.mk包含进selinux文件夹
- file_contexts： 给文件定义label。
- genfs_contexts: 
- property_contexts: 
- service_contexts: 给binder service定义label。
- seapp_contexts: 
- mac_permissions.xml: 

<a name="section3.3"></a>
## 3.3 Steps in detail
1. 内核中启用selinux： CONFIG\_SECURITY\_SELINUX=y
2. 修改内核启动参数BOARD\_KERNEL\_CMDLINE := androidboot.selinux=permissive，当开发好后删除
3. 启动系统，抓取log分析denied信息   
	
		adb shell su -c dmesg | grep denied | audit2allow -p out/target/product/BOARD/root/sepolicy
4. Evaluate the output
5. 分析device和新的需要label的文件
6. ...


<a name="section4"></a>
# 4. Customizing SELinux

<a name="section5"></a>
# 5. Validating SELinux
获取selinux全局模式
	
	getenforce

selinux工具

	sepolicy-analyze

<a name="section5.1"></a>
## 5.1 Reading denials

<a name="section5.2"></a>
## 5.2 Switching to permissive
userdebug和eng编译的android可以通过adb修改selinux的模式。

1. adb root
2. 设置permissive
   
		adb shell setenforce 0

或者通过内核启动参数修改

	androidboot.selinux=permissive
	androidboot.selinux=enforcing

<a name="section5.3"></a>
## 5.3 Using audit2allow

<a name="section6"></a>
# 6. Writing SELinux Policy



