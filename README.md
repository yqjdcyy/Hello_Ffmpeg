# 指令
- `ffmpeg`
- `ffprobe -i <resource>`
- `ffplay <resource>`

# 场景
## Audio.Cut
- `ffmpeg -ss 0 -t 3 -i audio/bgm.mp3 -acodec copy -y audio/bgm-min.mp3`
    ```
    audio[0:3] cut to mp3.new
    ```

## Video.Split2Img
- `ffmpeg -i dest/png_mp3.mp4 dest/png_mp3-mp4/%d.jpg`
    ```
    MUST create folder[dest/png_mp3-mp4] first
    ```

- `ffmpeg -i dest/png_mp3_loop.avi dest/png_mp3_loop-mp4/%d.jpg`
    ```
    MUST create folder[dest/png_mp3_loop-mp4] first
    ```

## Video.Gif
- `ffmpeg -i video/bike.mp4 -ss 0 -t 2 -vf "fps=15,scale=320:-1" -y dest/bike-vf.gif`
    ```
    file[fps=15,scale=320:-1].size= 825K
    ```

- `ffmpeg -i video/bike.mp4 -ss 0 -t 2 -r 24 -s 320:260 -y dest/bike-rs.gif`
    ```
    file[-s 320:260].size= 2,252K
    file[-r 24 -s 320:260].size= 1,824K
    ```


## IMG+ Audio= Video
- `ffmpeg -i img/bg.png -i audio/bgm-min.mp3 -y dest/png_mp3.mp4`
    ```
    convert:    success
    play:
        potplay:    success
        ffplay:     success
    ```

- `ffmpeg -loop 1 -shortest -y -i img/bg.png -i audio/bgm-min.mp3 -acodec copy dest/png_mp3.avi`
    ```
    convert:    fail
        Option shortest (finish encoding within shortest input) cannot be applied to input url img/bg.png
    ```

- `ffmpeg -loop 1 -y -i img/bg.png -i audio/bgm-min.mp3 -acodec copy -y dest/png_mp3_loop.avi`
    ```
    convert:    fail.not stop
    play:       
        potplay:    success
            audio.duration= 3
            video.duration= 19
        ffplay:     success
    ```

- `ffmpeg -i img/bg.png -i audio/bgm-min.mp3 -acodec copy -y dest/png_mp3.mp4`
    ```
    convert:    success
    play:       
        potplay:    success
        ffplay:     success
    ```

## IMGs+ Audio= Video
- `ffmpeg -i dest/png_mp3_loop-mp4/%d.jpg -i audio/bgm-min.mp3 -y dest/jpgs_mp3.mp4`

- `ffprobe -i dest/jpgs_mp3.mp4`
    ```
    Metadata:
        major_brand     : isom
        minor_version   : 512
        compatible_brands: isomiso2avc1mp41
        encoder         : Lavf57.82.101
    Duration: 00:00:13.28, start: 0.000000, bitrate: 199 kb/s
    Stream #0:0(und): Video: h264 (High) (avc1 / 0x31637661), yuvj420p(pc), 1150x650 [SAR 1:1 DAR 23:13], 179 kb/s, 25 fps, 25 tbr, 12800 tbn, 50 tbc (default)
    Metadata:
        handler_name    : VideoHandler
    Stream #0:1(und): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, mono, fltp, 69 kb/s (default)    Metadata:
        handler_name    : SoundHandler
    ```

- `ffmpeg -i img/loop/%d.png -r 24 -i audio/bgm-min.mp3 -y dest/pngs_r24.mp4`
    ```
    imgs(25)+ audio(3s)= video(3s)
    ```

- `ffmpeg -framerate 6 -i img/loop/%d.png -i audio/bgm-min.mp3 -y dest/pngs_framerate6.mp4`
    ```
    imgs(25)+ audio(3s)= video(4s, 6 frame/second)
    ```


## Video+ Audio= Video
- `ffmpeg -i video/bike.mp4 -i audio/bgm-min.mp3 -vcodec copy -acodec copy 0 -map 0.0:0 -map 1.0:1 dest/video_audio.mp4`
    ```
    video(5s)+ audio(3s)= video(5s)
    ```
- `ffmpeg -ss 0 -t 3 -i video/bike.mp4 -i audio/bgm-min.mp3 -y dest/video_audio.mp4`
    ```
    video[0:3]+ audio(3s)= video(3s)
    ```

## M3U8= Video
- `ffmpeg -i index.m3u8 [-c copy -bsf:a aac_adtstoasc] -y output.mp4`
    ```sh
    Result
        ts.time:    603.386| 10:03
        mp4.time:   04:50.55
        **实际时长与转换后时长不一致**
            部分情况下出现

    Error
        Non-monotonous DTS in output stream 0:0; previous: 26142599, current: 24141600; changing to 26142600. This may result in incorrect timestamps in the output file.
        ResultDuration is less than real duration

    Duration
        ffmpeg -i index.m3u8 -y output.mp4
            1:53.81
        ffmpeg -i index.m3u8 -c copy -bsf:a aac_adtstoasc -y output.mp4
            0:05.51
    ```

- `ffmpeg -i all.ts -codec copy -y output.mp4`
    - all.ts
        
        ```sh
        ls -v *.ts | grep "[0-9]" | xargs cat > all.ts
        cat index.m3u8 | grep ".ts" | xargs cat > all.ts
        ```
    - 优点
        - 可 **兼容异常** 音视频 ts 片数据
    - 缺点
        - 阿里云文件服务中，出现文件刚创建时，使用 `ls` 无法获取所有文件的异常

- `ffmpeg -f concat -safe 0 -i list -codec copy -y output.mp4`
    - list

        ```sh
        cat index.m3u8 | grep ".ts" | awk '{printf "file %s\n", $0}' > list 
        ```
    - 缺点

        ```sh
        转换时间
            相较 all.ts 方式，降低了一倍左右
        转换兼容性
            相较 all.ts 存在部分无法正常转换异常
            如若 `ffprobe -i 0.ts` 时报 `0.ts: Invalid data found when processing input` 导致无法正常转换
        ```

# 比较

| 类型 |                     名称                    |         分辨率         |     时长    | 大小 | 转换方式 |    转换时长   |
|------|---------------------------------------------|------------------------|-------------|------|----------|---------------|
| MP3  |                                             |                        |             |      |          |               |
|      | 74434125-45a7-4c37-a3a8-5156b327bda1        | 24 kb/s                | 00:14:29.69 | 2.5M |          |               |
|      |                                             |                        |             |      | all.ts   | 10.741287936s |
|      |                                             |                        |             |      | ts.list  | 5.829118866s  |
|      | 5c311087-4a3f-4fb3-bb6e-af7ed403a009        | 24 kb/s                | 00:06:43.91 | 1.2M |          |               |
|      |                                             |                        |             |      | all.ts   | 4.82727274s   |
|      |                                             |                        |             |      | ts.list  | 1.04991862s   |
|      | b036b618-f9c9-4165-8c09-65c1fa301ccd        | 24 kb/s                | 00:01:23.57 | 245K |          |               |
|      |                                             |                        |             |      | all.ts   | 1.198884141s  |
|      |                                             |                        |             |      | ts.list  | 1.07304459s   |
| MP4  |                                             |                        |             |      |          |               |
|      | 171c8282-5291-4727-9e39-35ceb54588e9_camera | 640x360 <br>442 kb/s   | 00:14:50.64 | 48M  |          |               |
|      |                                             |                        |             |      | all.ts   | 3.116673884s  |
|      |                                             |                        |             |      | ts.list  | 5.82908783s   |
|      | 6fb3a20a-e2aa-4bbf-b5d1-3b24fe3b9f53_camera | 1920x1080 <br>585 kb/s | 00:04:54.44 | 21M  |          |               |
|      |                                             |                        |             |      | all.ts   | 894.732724ms  |
|      |                                             |                        |             |      | ts.list  | 4.38981284s   |
|      | 7335cb91-a56e-49e4-92d4-0f561d4f6cef_camera | 640x360 <br>356 kb/s   | 00:00:46.88 | 2.0M |          |               |
|      |                                             |                        |             |      | all.ts   | 291.281715ms  |
|      |                                             |                        |             |      | ts.list  | 416.529161ms  |
|      | 24631476-9976-4c10-9193-10f522a16992_camera | 640x360 <br>339 kb/s   | 01:36:38.71 | 235M |          |               |
|      |                                             |                        |             |      | all.ts   | 22.508821932s |
|      |                                             |                        |             |      | ts.list  | 38.972474909s |

# 参考
## 官网
- [ffmpeg](https://www.ffmpeg.org/ffmpeg.html)

## IMG-> Video
- [Combine one image + one audio file to make one video using FFmpeg](https://superuser.com/questions/1041816/combine-one-image-one-audio-file-to-make-one-video-using-ffmpeg)
- [Useful ‘FFmpeg’ Commands for Video, Audio and Image ](https://www.tecmint.com/ffmpeg-commands-for-video-audio-and-image-conversion-in-linux/)
- [How do I convert a video to GIF using ffmpeg, with reasonable quality?](https://superuser.com/questions/556029/how-do-i-convert-a-video-to-gif-using-ffmpeg-with-reasonable-quality)


## M3U8-> Video
- [FFMPEG mp4 from http live streaming m3u8 file?](https://stackoverflow.com/questions/32528595/ffmpeg-mp4-from-http-live-streaming-m3u8-file)
- [Proper command to convert m3u8 to mp4](https://apple.stackexchange.com/questions/285635/proper-command-to-convert-m3u8-to-mp4)
- [利用ffmpeg合併m3u8串流影片，並且轉成MP4格式](https://shimeche.github.io/2017/04/13/%E5%88%A9%E7%94%A8ffmpeg%E5%90%88%E4%BD%B5m3u8%E4%B8%B2%E6%B5%81%E5%BD%B1%E7%89%87%EF%BC%8C%E4%B8%A6%E4%B8%94%E8%BD%89%E6%88%90MP4%E6%A0%BC%E5%BC%8F/)