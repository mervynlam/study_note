**转码分片、截取封面**

```bash
ffmpeg -i video.mp4 -c:v libx264 -hls_time 120 -hls_list_size 0 -c:a aac -strict -2 -f hls /transcoding/video.m3u8 
```

```bash
ffmpeg -i video.mp4 -y -f mjpeg -ss 3 -t 1 -s 900x500 /transcoding/video.jpg 
```

**nginx设置**

在`server`节点下添加

```conf
location /play {
	alias  /transcoding/;
}
```

**访问视频**

`http://192.168.23.241/play/video.m3u8`