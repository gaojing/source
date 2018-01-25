title: Building an Audio App
categories: Android
tags: [Android]
---

# Building a Media Browser Service

	<service android:name=".MediaPlaybackService">
	  <intent-filter>
	    <action android:name="android.media.browse.MediaBrowserService" />
	  </intent-filter>
	</service>

## Initialize the media session

## Manage client connections
MediaBrowerService有两个方法来处理client connection

- onGetRoot()来控制访问service
- onLoadChildren()提供给client用来获取service的媒体结构




![](/img/android/media_apps/Audio_app.png)



### 参考
[Media Apps](https://developer.android.com/guide/topics/media-apps/index.html)
