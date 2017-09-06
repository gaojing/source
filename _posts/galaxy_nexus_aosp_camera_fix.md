title: 解决galaxy nexus aosp camera无法使用问题
categories: android
---

adb pull /system/vendor/etc/sirfgps.conf

Next, add the following to vendor/samsung/maguro/proprietary/Android.mk

###################################
include $(CLEAR_VARS)
LOCAL_MODULE := sirfgps
LOCAL_MODULE_OWNER := samsung
LOCAL_SRC_FILES := sirfgps.conf
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_SUFFIX := .conf
LOCAL_MODULE_CLASS := SHARED_LIBRARIES
LOCAL_MODULE_PATH := $(TARGET_OUT_VENDOR)/etc
include $(BUILD_PREBUILT)

include $(CLEAR_VARS)
LOCAL_MODULE := gps.omap4
LOCAL_MODULE_OWNER := samsung
LOCAL_SRC_FILES := gps.omap4.so
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_SUFFIX := .so
LOCAL_MODULE_CLASS := SHARED_LIBRARIES
LOCAL_MODULE_PATH := $(TARGET_OUT_VENDOR)/lib/hw
include $(BUILD_PREBUILT)

include $(CLEAR_VARS)
LOCAL_MODULE := ducati-m3
LOCAL_MODULE_OWNER := samsung
LOCAL_SRC_FILES := ducati-m3.bin
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_SUFFIX := .bin
LOCAL_MODULE_CLASS := SHARED_LIBRARIES
LOCAL_MODULE_PATH := $(TARGET_OUT_VENDOR)/firmware
include $(BUILD_PREBUILT)
###################################

in the same file, change the LOCAL_MODULE_PATH for fRom to install into $(TARGET_OUT)/bin rather than $(TARGET_VENDOR_OUT)/bin.

Lastly, change your packages list in vendor/samsung/maguro/device-partial.mk to:

###################################
PRODUCT_PACKAGES := \
fRom \
libsec-ril \
libsecril-client \
sirfgps \
ducati-m3 \
gps.omap4

[引用](https://anders.com/cms/435/Google.Galaxy.Nexus/Android/AOSP/camera/GPS/driver)