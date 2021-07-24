# FFmpeg-ext

基于FFmpeg 4.4版本的源码进行扩展

FFmpeg源码官网下载地址：[Download FFmpeg](http://ffmpeg.org/download.html#releases)

### 扩展内容

#### libavfilter/mpdecimate_area

在libavfilter中添加一个新的filter：mpdecimate_area，用于检测视频中指定区域的卡顿情况。

mpdecimate_area是在mpdecimate的基础上实现的。mpdecimate只能检测视频的整个画面的卡顿情况，而不能指定某个区域。

利用现有的ffmpeg命令，如果我们希望检测视频中某个区域 (x, y, w, h) 的卡顿情况，就需要先用crop命令把这个区域的视频裁剪出来，再用mpdecimate命令分析裁剪后的视频。

显然，裁剪这一过程需要花费很多时间。而且，裁剪后的视频质量会对卡顿检测的结果造成影响。虽然可以用参数控制裁剪后的视频质量，但是质量越高，生成视频的时间就越长。

为了提高卡顿分析的效率，我们实现了mpdecimate_area，可以直接对视频中的指定区域进行卡顿检测，省去了视频裁剪。

mpdecimate_area的参数为 (max, hi, lo, frac, x, y, w, h)。其中，(max, hi, lo, frac) 与mpdecimate命令的参数是一致的，添加的参数 (x, y, w, h) 则用于指定一个区域的位置，(x, y) 为左上角的坐标，(w, h) 为宽度、高度。

使用mpdecimate_area时，如果不指定 (x, y, w, h) 参数，则mpdecimate_area的表现与mpdecimate一致。

### 使用说明

安装重新编译生成的ffmpeg程序后，用以下命令查看mpdecimate_area的用法：

`ffmpeg -h filter=mpdecimate_area`

mpdecimate_area的使用示例如下：

`ffmpeg -hide_banner -i D:/Videos/video_1.mp4 -vf mpdecimate_area=max=0:hi=768:lo=320:frac=0.33:x=3:y=257:w=694:h=362 -loglevel debug -an -f null - 2>&1`

运行该命令，利用输出的信息可以分析视频的卡顿情况。命令中的参数含义如下：

`-hide_banner`：隐藏横幅（横幅是直接运行ffmpeg命令看到的一些提示信息）

`-i D:/Videos/video_1.mp4`：输入的视频

 `-vf mpdecimate_area=max=0:hi=768:lo=320:frac=0.33:x=3:y=257:w=694:h=362`：选择的filter及其参数

`-loglevel debug`：日志级别为debug。我们利用输出信息来分析视频的卡顿情况，如果不使用该选项，则输出信息较为简略，无法从中分析出卡顿情况

`-an`：去除音频

`-f null`：不生成输出视频

`- 2>&1`：将标准错误输出重定向到标准输出

