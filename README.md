nginx-rtmp
==========

nginx-rtmp 流媒体服务器的搭建 

**使用环境:**

- **linux**
- **nginx 安装nginx-rtmp-module模块**
- **客户端 ffmpeg 转换以及流化音视频**
- **采集桌面或摄像头 screen-capture-recorder-to-video-windows-free**

## 主要的使用场景:
---
#### 1. 小米盒子提供视频源,通过设备(桌面或者摄像头)采集音视频,通过rtmp等协议,经过ffmpeg转换以及流化音视频到nginx-rtmp流媒体服务器。
#### 2.拓扑图
![img](http://182.92.69.21/images/nginx-rtmp/5.png)
## 展示(这里展示采集桌面视频)
---
#### 1. pc机桌面
![img](http://182.92.69.21/images/nginx-rtmp/1.png)
#### 2. vlc播放直播桌面视频(rtmp://182.92.69.21/hls/test)
![img](http://182.92.69.21/images/nginx-rtmp/2.png)
#### 3. 手机播放m3u8流(182.92.69.21/hls/test.m3u8)
![img](http://182.92.69.21/images/nginx-rtmp/3.PNG)
![img](http://182.92.69.21/images/nginx-rtmp/4.PNG)

	
