title: galaxy nexus audio hal
categories: android
tags: [android audio hal]
---

# android hal   
- 实现audio.h和audio_effect.h中定义的接口
- 创建audio policy配置文件
	- 描述audio topology，将HAL实现封装成动态库
- 配置pre-processing effect

# main function   
- hardware/libhardware/include/hardware/audio.h

# struct pcm_config
在external/tinyalsa/include/tinyalsa中定义，描述stream的配置

# audio flinger加载audio module
在load\_audio\_interface()函数中

	rc = hw_get_module_by_class(AUDIO_HARDWARE_MODULE_ID, if_name, &mod);

其中AUDIO\_HARDWARE\_MODULE\_ID在hardware/libhardware/include/hardware中定义

	#define AUDIO_HARDWARE_MODULE_ID "audio"

# audio module struct
	struct audio_module {
	    struct hw_module_t common;
	};

具体实现

	struct audio_module HAL_MODULE_INFO_SYM = {
	    .common = {
	        .tag = HARDWARE_MODULE_TAG,
	        .module_api_version = AUDIO_MODULE_API_VERSION_0_1,
	        .hal_api_version = HARDWARE_HAL_API_VERSION,
	        .id = AUDIO_HARDWARE_MODULE_ID,
	        .name = "Tuna audio HW HAL",
	        .author = "The Android Open Source Project",
	        .methods = &hal_module_methods,
	    },
	};

其中包含的hw\_module\_methods\_t成员

	static struct hw_module_methods_t hal_module_methods = {
	    .open = adev_open,
	};

# tuna audio device
结构体定义

	struct tuna_audio_device {
	    struct audio_hw_device hw_device;
	
	    pthread_mutex_t lock;       /* see note below on mutex acquisition order */
	    struct mixer *mixer;
	    struct mixer_ctls mixer_ctls;
	    audio_mode_t mode;
	    int out_device;
	    int in_device;
	    struct pcm *pcm_modem_dl;
	    struct pcm *pcm_modem_ul;
	    int in_call;
	    float voice_volume;
	    struct tuna_stream_in *active_input;
	    struct tuna_stream_out *outputs[OUTPUT_TOTAL];
	    bool mic_mute;
	    int tty_mode;
	    struct echo_reference_itfe *echo_reference;
	    bool bluetooth_nrec;
	    bool device_is_toro;
	    int wb_amr;
	    bool screen_off;
	
	    /* RIL */
	    struct ril_handle ril;
	};