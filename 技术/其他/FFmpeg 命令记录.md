**直播推流**

```bash
ffmpeg -re -i aaa.mp4 -vcodec copy -codec copy -f flv rtmp://192.168.23.241/hls/cctv
```

**转码分片**

```bash
ffmpeg -i input.mp4 -c:v libx264 -c:a aac -strict -2 -f hls -hls_list_size 0 -hls_time 60 /play/mp4/output.m3u8
```

