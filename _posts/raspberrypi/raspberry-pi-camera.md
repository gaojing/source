title: 树莓派使用摄像头
---
如何在树莓派上使用摄像头。

### enable camera in raspi-config

### 拍摄照片
	raspistill -o cam.jpg

### 拍摄视频
	raspivid -o vid.h264

### python3控制摄像头
picamera

### raspistill
水平垂直翻转

	raspistill -vf -hf -o cam.jpg


### 参考
[https://www.raspberrypi.org/learning/addons-guide/picamera](https://www.raspberrypi.org/learning/addons-guide/picamera)   
[Getting started with picamera](https://www.raspberrypi.org/learning/getting-started-with-picamera)   
[picamera reference](http://picamera.readthedocs.io/en/release-1.10/)
