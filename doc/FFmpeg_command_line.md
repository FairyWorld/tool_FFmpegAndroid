
FFmpeg多媒体库支持的命令行调用分为三个模块：ffmpeg、ffprobe、ffplay。
其中ffmpeg命令行常用于音视频剪切、转码、滤镜、拼接、混音、截图等；
ffprobe用于检测多媒体流格式；ffplay用于播放视频,包括本地与网络视频。
详情可查阅FFmpeg官方文档：ffmpeg Documentation。

# 一、ffmpeg命令行
## 1、命令行参数介绍
Print help / information / capabilities:
--help topic        show help
-version            show version
-formats            show available formats
-muxers             show available muxers
-demuxers           show available demuxers
-devices            show available devices
-codecs             show available codecs
-decoders           show available decoders
-encoders           show available encoders
-bsfs               show available bit stream filters
-protocols          show available protocols
-filters            show available filters
-pix_fmts           show available pixel formats
-sample_fmts        show available audio sample formats
-hwaccels           show available HW acceleration methods

Global options (affect whole program instead of just one file:
-v loglevel         set logging level
-y                  overwrite output files
-n                  never overwrite output files
-filter_threads     number of non-complex filter threads
-stats              print progress report during encoding
-vol volume         change audio volume (256=normal)

Per-file main options:
-f fmt              force format
-c codec            codec name
-codec codec        codec name
-t duration         record or transcode "duration" seconds of audio/video
-to time_stop       record or transcode stop time
-fs limit_size      set the limit file size in bytes
-ss time_off        set the start time offset
-sseof time_off     set the start time offset relative to EOF
-frames number      set the number of frames to output
-filter filter_graph  set stream filtergraph
-discard            discard
-disposition        disposition

Video options:
-vframes number     set the number of video frames to output
-r rate             set frame rate (Hz value, fraction or abbreviation)
-s size             set frame size (WxH or abbreviation)
-aspect aspect      set aspect ratio (4:3, 16:9 or 1.3333, 1.7777)
-bits_per_raw_sample number  set the number of bits per raw sample
-vn                 disable video
-vcodec codec       force video codec ('copy' to copy stream)
-timecode hh:mm:ss[:;.]ff  set initial TimeCode value.
-vf filter_graph    set video filters
-ab bitrate         audio bitrate (please use -b:a)
-b bitrate          video bitrate (please use -b:v)

Audio options:
-aframes number     set the number of audio frames to output
-aq quality         set audio quality (codec-specific)
-ar rate            set audio sampling rate (in Hz)
-ac channels        set number of audio channels
-an                 disable audio
-acodec codec       force audio codec ('copy' to copy stream)
-vol volume         change audio volume (256=normal)
-af filter_graph    set audio filters

Subtitle options:
-s size             set frame size
-sn                 disable subtitle
-scodec codec       force subtitle codec ('copy' to copy stream)

命令行整体格式：以ffmpeg开头，参数之间用空格隔开，每个参数key以"-"开头，后面跟着参数value，输出文件放在命令行最后。

## 2、音频常见操作
### 2.1 音频转码
直接转码：
ffmpeg -i input.mp3 output.m4a
指定编码器、采样率、声道数转码：
ffmpeg -i input.mp3 -acodec aac -ac 2 -ar 44100 output.m4a

### 2.2 音频剪切
-ss 10指定从第10秒开始，-t 20代表剪切20秒
ffmpeg -i input.mp3 -ss 10 -t 20 cut.mp3

### 2.3 音频拼接
ffmpeg -i concat:"hello.mp3|world.mp3" -acodec copy -vn concat.mp3

### 2.4 音频混音
使用amix，参数inputs代表输入流个数，duration有longest、shortest、first三种模式，weights设置每条流音量权重占比：
ffmpeg -i hello.mp3 -i world.mp3 -filter_complex amix=inputs=2:duration=first -vn mix.mp3
使用amerge，合并成多声道输出：
ffmpeg -i hello.mp3 -i world.mp3 -filter_complex [0:a][1:a]amerge=inputs=2[aout] -map [aout] merge.mp3

### 2.5 空灵音效
aecho接收4个参数：in_gain (0, 1]；out_gain (0, 1]；delays (0 - 90000]；decays (0 - 1.0]
ffmpeg -i input.mp3 -af aecho=0.8:0.8:1000:0.5 echo.mp3

### 2.6 惊悚音效
tremolo接收2个参数：frequency [0.1, 20000.0]；depth (0, 1]
ffmpeg -i input.mp3 -af tremolo=5:0.9 tremolo.mp3

### 2.7 搞笑音效
搞笑音效通过调节音速实现，使用atempo：
ffmpeg -i input.mp3 -filter_complex atempo=0.5 atempo.mp3

### 2.8 静音检测
ffmpeg -i input.mp3 -af silencedetect=noise=0.0001 -f null -

### 2.9 修改音量
ffmpeg -i input.mp3 -af volume=0.5 volume.mp3

### 2.10 抽取音频
从视频抽取音频，也就是disable视频流、拷贝音频流：
ffmpeg -i input.mp4 -acodec copy -vn out.mp3

## 3、视频常见操作
### 3.1 视频剪切
基本剪切，指定起始时间、剪切时长：
ffmpeg -i input.mp4 -ss 10 -t 20 -codec copy cut.mp4
精确剪切，包含多音轨，-map 0代表所有track流进行剪切，-accurate_seek代表精确seek：
ffmpeg -i input.mp4 -ss 10 -accurate_seek -t 20 -map 0 -codec copy cut.mp4

### 3.2 视频转码
使用-vcodec指定视频编码，-acodec指定音频编码，-s 640x480指定视频分辨率，
-b 200k指定码率，-r 20指定帧率，这样达到视频压缩效果：
ffmpeg -i input.mp4 -vcodec libx264 -acodec aac -s 640x480 -b 200k -r 20 transcode.mp4

### 3.3 视频截图
使用-vframes指定截图数量，-ss指定起始时间放在-i前面，这样保证先seek到指定位置再截图。
如果是先-i指定输入文件再-ss，是从时间0开始解码，直到指定时间再截图，这样效率太低：
ffmpeg -ss 10 -i input.mp4 -f image2 -vframes 1 -an screenshot.jpg

### 3.4 图片水印
使用-filter_complex指定位置overlay=x:y，如下所示：
ffmpeg -i input.mp4 -i logo.png -filter_complex overlay=10:20 pic_watermark.mp4
如果要灵活设置左上角、右上角、左下角、右下角位置，可以使用case语句，其中main_w代表视频宽度，
main_h代表视频高度，overlay_w代表水印宽度，overlay_h代表水印高度：

public static String obtainOverlay(int offsetX, int offsetY, int location) {
    switch (location) {
        case 2:
            return "overlay='(main_w-overlay_w)-" + offsetX + ":" + offsetY + "'";
        case 3:
            return "overlay='" + offsetX + ":(main_h-overlay_h)-" + offsetY + "'";
        case 4:
            return "overlay='(main_w-overlay_w)-" + offsetX + ":(main_h-overlay_h)-" + offsetY + "'";
        case 1:
        default:
            return "overlay=" + offsetX + ":" + offsetY;
    }
}

public static String[] addWaterMarkImg(String inputPath, String imgPath, int location,
                                           int offsetXY, String outputPath) {
    String overlay = obtainOverlay(offsetXY, offsetXY, location);
    String waterMarkCmd = "ffmpeg -i %s -i %s -filter_complex %s -preset:v superfast %s";
    waterMarkCmd = String.format(waterMarkCmd, inputPath, imgPath, overlay, outputPath);
    return waterMarkCmd.split(" ");
}

### 3.5 GIF水印
使用-ignore_loop 0代表GIF循环显示，其他操作与图片水印一致：
ffmpeg -i input.mp4 -ignore_loop 0 -i logo.gif -filter_complex overlay=10:20 gif_mark.mp4

### 3.6 去除水印
使用delogo命令，然后指定水印位置：
ffmpeg -i input.mp4 -filter_complex delogo=x=10:y=20:w=90:h=30 delogo.mp4

### 3.7 视频拼接
视频拼接分为垂直拼接、水平拼接，还有前后拼接，这里主要介绍垂直与水平拼接，其中使用hstack做水平拼接，vstack做垂直拼接：
ffmpeg -i input1.mp4 -i input2.mp4 -filter_complex hstack out.mp4

### 3.8 视频翻转
使用reverse实现视频翻转，但是处理比较耗时：
ffmpeg -i input.mp4 -vf reverse -an output.mp4

### 3.9 视频降噪
使用-nr代表noise reduction，进行视频降噪：
ffmpeg -i in.mp4 -nr 500 out.mp4

### 3.10 视频抽帧
ffmpeg -ss 20 -accurate_seek -t 10 -i input.mp4 -an -r 5 %3d.jpg

### 3.11 播放速度
使用setpts设置视频速度，atempo设置音频速度：
ffmpeg -i in.mp4 -filter_complex [0:v]setpts=%.2f*PTS[v];[0:a]atempo=%.2f[a] -map [v] -map [a] out.mp4

### 3.12 插入封面
其中-disposition:v:1 attached_pic代表封面：
ffmpeg -i in.mp4 -i photo.png -map 0 -map 1 -c copy -c:v:1 png -disposition:v:1 attached_pic out.mp4

### 3.13 旋转视频
ffmpeg -i in.mp4 -c copy -metadata:s:v:0 rotate=90 out.mp4

# 二、ffplay命令行
ffplay主要用于播放视频，也可以播放网络流，示例如下：
ffplay -i beyond.mp4

# 三、ffprobe命令行
ffprobe主要用于检测多媒体信息，包括时长、视频分辨率、帧率、音频采样率、声道数、每个stream流信息等等。
支持显示形式包括：json和xml，例如-print_format json
ffprobe -i input.mp4 -show_streams -show_format -print_format json

如果要显示每帧数据信息，使用-show_frames可以显示pts、packet_size、duration、frame_type等信息：
ffprobe -i input.mp4 -show_frames

如果只要显示视频流，使用-select_streams v。其中v代表video，a代表audio，s代表subtitle。
ffprobe -i input.mp4 -show_frames -select_streams v