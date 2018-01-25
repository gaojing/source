title: audio policy service
categories: Android
tags: [Android]
---
# Table of Contents
- [1. AudioPolicy Service概述](#section1)
- [2. AudioPolicy Service启动过程](#section2)
	- [2.1 AudioPolicyService构造函数流程](#section2.1)
		- [2.1.1 加载policy module](#section2.1.1)
		- [2.1.2 打开audio policy设备](#section2.1.2)
		- [2.1.3 生成policy](#section2.1.3)
	- [2.2 AudioPolicyService与音频设备](#section2.2)
- [3. 音频流的回放 AudioTrack](#section3)

<a name="section1"></a>
# 1. AudioPolicy Service概述

![](/img/android/audio/audio_policy001.jpg)

- 一个PlaybackThread的输出对应一种设备
- 特定时间，同一类型的音频对应的输出设备是统一的
- AudioPolicyService起到路由器的作用

<a name="section2"></a>
# 2. AudioPolicy Service启动过程
audiopolicy service驻留在mediaserver进程中，mediaserver代码位于frameworks\av\media\mediaserver\main_mediaserver.cpp中。

	int main(int argc, char** argv)
	{
		...
		AudioPolicyService::instantiate();
		...
	}

android编译会生成mediaserver可执行程序，参考frameworks\av\media\mediaserver\Android.mk

	LOCAL_MODULE:= mediaserver

	include $(BUILD_EXECUTABLE)

mediaserver会在系统启动时运行，参考system\core\rootdir\init.rc

	service media /system/bin/mediaserver
	    class main
	    user media
	    group audio camera inet net_bt net_bt_admin net_bw_acct drmrpc mediadrm
	    ioprio rt 4

<a name="section2.1"></a>
## 2.1 AudioPolicyService构造函数流程

<a name="section2.1.1"></a>
## 2.1.1 加载policy module

	/* instantiate the audio policy manager */
    rc = hw_get_module(AUDIO_POLICY_HARDWARE_MODULE_ID, &module);

   AUDIO\_POLICY\_HARDWARE\_MODULE\_ID定义为"audio_policy"，在galaxy nexus代码中最终会去加载audio\_policy.default.so，对应的库是libaudiopolicy\_legacy，代码位于hardware\libhardware\_legacy\audio中.

<a name="section2.1.2"></a>
## 2.1.2 打开audio policy设备

	rc = audio_policy_dev_open(module, &mpAudioPolicyDev);

调用的是audio\_policy\_hal.cpp中的legacy\_ap\_dev\_open函数，最终生成的policy设备是legacy\_ap\_device。

<a name="section2.1.3"></a>
## 2.1.3 生成policy

	rc = mpAudioPolicyDev->create_audio_policy(mpAudioPolicyDev, &aps_ops, this,
                                               &mpAudioPolicy);

生成一个策略，对应的函数是audio\_policy\_hal.cpp中的create\_legacy\_ap函数

	struct legacy_audio_policy {
	    struct audio_policy policy;
	
	    void *service;
	    struct audio_policy_service_ops *aps_ops;
	    AudioPolicyCompatClient *service_client;
	    AudioPolicyInterface *apm;
	};

aps_ops是AudioPolicyService提供的函数指针，是AudioPolicyService与外界沟通的接口。
apm是AudioPolicyManager的简写，AudioPolicyInterface是它的基类，默认情况对应AudioPolicyManagerDefault.cpp中的实现。

	lap->apm = createAudioPolicyManager(lap->service_client);

<a name="section2.2"></a>
## 2.2 AudioPolicyService与音频设备
AuidoPolicyService通过AudioPolicyManagerDefault的父类AudioPolicyManagerBase的构造函数去加载音频设备。

	if (loadAudioPolicyConfig(AUDIO_POLICY_VENDOR_CONFIG_FILE) != NO_ERROR) {
        if (loadAudioPolicyConfig(AUDIO_POLICY_CONFIG_FILE) != NO_ERROR) {
            ALOGE("could not load audio policy configuration file, setting defaults");
            defaultAudioPolicyConfig();
        }
    }

首先读取audio_policy.conf配置文件得到audio interface以及信息。

	mHwModules[i]->mHandle = mpClientInterface->loadHwModule(mHwModules[i]->mName);
	...
	audio_io_handle_t output = mpClientInterface->openOutput(
                                                outProfile->mModule->mHandle,
                                                &outputDesc->mDevice,
                                                &outputDesc->mSamplingRate,
                                                &outputDesc->mFormat,
                                                &outputDesc->mChannelMask,
                                                &outputDesc->mLatency,
                                                outputDesc->mFlags);

通过binder rpc调用audioflinger的loadHwModule和openOutput函数。

<a name="section3"></a>
# 3. 音频流的回放 AudioTrack
<a name="section3.1"></a>
# 3.1 AudioTrack应用实例
AudioTrack用于PCM音频流的回放，有两种数据传输方式

- 调用write把音频数据push到AudioTrack中
- 数据接收方主动索取

AudioTrack支持两种数据模式

- static   
数据一次性交给接收方。
- streaming   
数据不断的传递给接收方

参考MediaAudioTrackTest.java中的testSetStereoVolumnMax函数对AudioTrack的应用

<a name="section3.2"></a>
# 3.2 AudioTrack
创建AudioTrack

	AudioTrack track = new AudioTrack(TEST_STREAM_TYPE, TEST_SR, TEST_CONF, TEST_FORMAT, 
                minBuffSize, TEST_MODE);

AudioTrack构造函数中调用native代码

	int initResult = native_setup(new WeakReference<AudioTrack>(this),
                mStreamType, mSampleRate, mChannels, mAudioFormat,
                mNativeBufferSizeInBytes, mDataLoadMode, session);

native_setup对应frameworks\base\core\Jni\android_media_AudioTrack.cpp中的android_media_AudioTrack_native_setup函数

	sp<AudioTrack> lpTrack = new AudioTrack();

创建一个本地的AudioTrack对象

	AudioTrackJniStorage* lpJniStorage = new AudioTrackJniStorage();

创建存储音频数据的地方

	lpTrack->set(
            atStreamType,// stream type
            sampleRateInHertz,
            format,// word length, PCM
            nativeChannelMask,
            frameCount,
            AUDIO_OUTPUT_FLAG_NONE,
            audioCallback, &(lpJniStorage->mCallbackData),//callback, callback data (user)
            0,// notificationFrames == 0 since not using EVENT_MORE_DATA to feed the AudioTrack
            0,// shared mem
            true,// thread can call Java
            sessionId);// audio session ID

