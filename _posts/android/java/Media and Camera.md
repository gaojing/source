title: Media and Camera
categories: Android
tags: [Android]
---
# Table of Contents
- [1. MediaPlayer](#section1)
	- [1.1 The basics](#section1.1) 
	- [1.2 Manifest declarations](#section1.2)
	- [1.3 Using MediaPlayer](#section1.3) 
		- [1.3.1 播放例子](#section1.3.1) 
- [2. AudioPolicy Service启动过程](#section2)
	- [2.1 获取权限](#section2.1)
	- [2.2 创建运行MediaRecorder](#section2.2)
	- [2.3 创建运行MediaRecorder](#section2.3)

Android提供播放和录制以及控制camera的api。

<a name="section1"></a>
# 1. MediaPlayer

<a name="section1.1"></a>
# 1.1 The basics
和播放audio与video相关的类。

- MediaPlayer   
- AudioManager   
管理audio source和设备上的audio output

<a name="section1.2"></a>
# 1.2 Manifest declarations
- Internet Permission   
如果用MediaPlayer来播放网络流时，要申请internet权限

		<uses-permission android:name="android.permission.INTERNET" />

- Wake Lock Permission   
保持屏幕常亮

		<uses-permission android:name="android.permission.WAKE_LOCK" />

<a name="section1.3"></a>
# 1.3 Using MediaPlayer
MediaPlayer对象的实例可以获取，解码和播放audio与video。支持的媒体来源有

- Local resources
- Internal URIs
- External URIs

<a name="section1.3.1"></a>
## 1.3.1 播放例子
播放本地资源

	MediaPlayer mediaPlayer = MediaPlayer.create(context, R.raw.sound_file_1);
	mediaPlayer.start();

播放internal URIs

	Uri myUri = ....; // initialize Uri here
	MediaPlayer mediaPlayer = new MediaPlayer();
	mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
	mediaPlayer.setDataSource(getApplicationContext(), myUri);
	mediaPlayer.prepare();
	mediaPlayer.start();

<a name="section2"></a>
# 2. MediaRecorder

Android多媒体框架支持录制和压缩音视频。

<a name="section2.1"></a>
# 2.1 获取权限
	<uses-permission android:name="android.permission.RECORD_AUDIO" />

<a name="section2.2"></a>
# 2.2 创建运行MediaRecorder

- 设置audio source: setAudioSource()
- 设置output格式: setOutputFormat()
- 设置输出文件名: setOutputFile()
- 设置audio encoder: setAudioEncoder()
- 完成初始化: prepare()

然后通过start和stop来开始和停止，使用完后调用release释放。

<a name="section2.3"></a>
# 2.3 创建运行MediaRecorder