nginx-rtmp
==========

nginx-rtmp 流媒体服务器的搭建 

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


