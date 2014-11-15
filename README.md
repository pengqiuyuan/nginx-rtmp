nginx-rtmp
==========

nginx-rtmp 流媒体服务器的搭建 

在线Demo,直播自己的pc机桌面音视频
```
1、下载ffmpeg及脚本
http://download.csdn.net/detail/pqy15005917185/8160799
```
```
2、安装screen-capture-recorder(采集windows桌面、摄像头)
下载地址：
http://download.csdn.net/detail/pqy15005917185/8160801
直接安装就ok了，安装之后才可以使用bat脚本的
“-f dshow -i video=screen-capture-recorder -f dshow -i audio=virtual-audio-capturer”
```
![img](http://182.92.69.21/images/nginx-rtmp/10.png)
```
3、解压,点击"流服务器直播.bat",运行如下图
```
![img](http://182.92.69.21/images/nginx-rtmp/7.png)
```
4、vlc访问地址“rtmp://182.92.69.21/hls/test” 或者直接用iphone来访问“http://182.92.69.21/hls/test.m3u8”
```

**使用环境:**

- **linux**
- **nginx 安装nginx-rtmp-module模块**
- **客户端 ffmpeg 转换以及流化音视频**
- **采集桌面或摄像头 screen-capture-recorder-to-video-windows-free**

```
https://github.com/arut/nginx-rtmp-module  nginx-rtmp流媒体模块
https://github.com/rdp/screen-capture-recorder-to-video-windows-free  采集桌面、摄像头
```

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

## 环境部署及流媒体服务器搭建:
---	
#### 1.nignx及nginx-rtmp模块的安装
nginx-rtmp的下载,直接下源码解压就ok了,我放在这里/home/dev/pengqiuyuan/nginx-rtmp-module-master 
```
https://github.com/arut/nginx-rtmp-module
```
![img](http://182.92.69.21/images/nginx-rtmp/6.png)

在编译nginx的时候添加nginx-rtmp模块的路径
```
./configure --add-module=/home/dev/pengqiuyuan/nginx-rtmp-module-master --with-http_ssl_module
make
make install
```
#### 2.nginx平滑安装nginx-rtmp

#### 3.nginx.conf的修改(完整的流媒体配置)
> (1).chunk_size:流整合的最大的块大小。默认值为 4096。这个值设置的越大 CPU 负载就越小。这个值不能低于 128。

> (2).listen 1935:给NGINX添加一个监听端口以接收RTMP连接,iptables不需要开放1935端口。

> (3).application hls:HLS协议支持。hls_path(m3u8文件生产路径)、hls_fragment、hls_playlist_length(5个分片,每片11s      .注:这样设置实时直播延时在1分钟左右)

> (4).http请求地址(http://182.92.69.21/hls/test.m3u8)

> (5).vlc播放rtmp流(rtmp://182.92.69.21/hls/test)

> (6).hls_path和location /hls{alias}的路径保持一样如:/home/dev/pengqiuyuan/streaming ,目录下面保存的是客户端推送到流服务器的m3u8、ts文件

```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user              nginx;
worker_processes  1;

#error_log  /var/log/nginx/error.log;
#error_log  /var/log/nginx/error.log  notice;
#error_log  /var/log/nginx/error.log  info;

pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}

rtmp {
   server{
     listen 1935;
     chunk_size 4096;
     application hls { 
       live on;  
       hls on;  
       hls_path /home/dev/pengqiuyuan/streaming;  
       hls_fragment 11s;
       hls_playlist_length 55s;
       allow play all;
     }
   }
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include /etc/nginx/conf.d/*.conf;

    server{
       listen 80;
       server_name www.fjpqy.cn;
       access_log logs/redmine.access.log;
       error_log  logs/redmine.error.log info;
       client_body_buffer_size 128k;
       proxy_connect_timeout 600;
       proxy_read_timeout 600;
       proxy_send_timeout 600;
       proxy_buffer_size 256k;
       proxy_buffers   8 256k;
       proxy_busy_buffers_size 512k;
       proxy_temp_file_write_size 512k;
      location /stat{
         rtmp_stat all;
         rtmp_stat_stylesheet stat.xsl;
      }
      location /stat.xsl{
        root /home/dev/pengqiuyuan/nginx-rtmp-module-master/;
      }
      location /test{
       alias /home/dev/pengqiuyuan/nginx-rtmp-module-master/test/rtmp-publisher/;
      }
      location /hls{
        #server hls fragments
        types{
             application/vnd.apple.mpegurl m3u8;
             video/mp2t ts;
           }
        alias /home/dev/pengqiuyuan/streaming;
        expires -1;
      }
    }
}

```
#### 4.客户端部分,采集桌面视频以及ffmpeg转换以及流化音视频
之前遇到丢帧的问题,real-time buffer 276% full! frame dropped! 

```
问题：
https://github.com/rdp/screen-capture-recorder-to-video-windows-free/issues/37
```
桌面音视频采集成功的例子:
```
ffmpeg -f dshow -i video=screen-capture-recorder -f dshow -i audio=virtual-audio-capturer -vf scale=1280:720 -vcodec libx264 -r 60.97 -acodec libvo_aacenc -ac 2 -ar 44100 -ab 128 -pix_fmt yuv420p -tune zerolatency -preset ultrafast -f flv "rtmp://192.168.1.50/hls/test"
```
ffmpeg(带脚本)下载地址,流媒体服务器搭建好之后,修改bat脚本里的推送地址“rtmp://192.168.1.50/hls/test”,直接点击运行就可以向服务器推送了,之后“/home/dev/pengqiuyuan/streaming”目录下面会生产m3u8、ts文件就成功了
下载地址：
http://download.csdn.net/detail/pqy15005917185/8160799
```
阿里云就1M的带宽,被占的满满的,下载地址换到csdn上面好了
```
![img](http://182.92.69.21/images/nginx-rtmp/11.png)
![img](http://182.92.69.21/images/nginx-rtmp/7.png)
![img](http://182.92.69.21/images/nginx-rtmp/8.png)
![img](http://182.92.69.21/images/nginx-rtmp/9.png)

#### 5.视频直播+实时聊天部分,之后补充
整体思路是 nginx-rtmp(流媒体服务) + nodejs(即时聊天)
