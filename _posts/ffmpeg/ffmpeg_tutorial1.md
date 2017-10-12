title: An ffmpeg and SDL Tutorial
categories: ffmpeg
tags: [ffmpeg]
---
# tutorial1 : Making Screencaps
## overview
最基本的流程

	10 OPEN video_stream FROM video.avi
	20 READ packet FROM video_stream INTO frame
	30 IF frame NOT COMPLETE GOTO 20
	40 DO SOMETHING WITH frame
	50 GOTO 20

