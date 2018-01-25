title: Managing Audio Focus
categories: Android
tags: [Android]
---
# Table of Contents
- [1. android8.0之前](#section1)
- [2. AudioPolicy Service启动过程](#section2)
	- [2.1 AudioPolicyService构造函数流程](#section2.1)
		- [2.1.1 加载policy module](#section2.1.1)

为了避免多个music app同时播放，android引入audio focus的概念，同一时间只有一个app能得到audio focus。

一个规范的audio app应该遵守下面的规则

- 播放前调用requestAudioFocus()获取焦点，确认返回AUDIOFOCUS\_REQUEST\_GRANTED。
- 当另外的程序获取到audio focus时停止、暂停播放或者降低音量播放。
- 播放完成后，释放audio focus。

<a name="section1"></a>
# 1. android8.0之前