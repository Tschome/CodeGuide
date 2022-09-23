# Ubuntu下载使用FFmpeg

[【极简】ubuntu / linux 直接下载使用 FFmpeg 步骤（带 x265)](https://blog.csdn.net/weixin_43667077/article/details/122276284)

https://blog.csdn.net/weixin_43400806/article/details/124288676

# ffmpeg mp3转pcm指令



## 使用范例：

### 大段数据格式

```shell
ffmpeg -i test.mp3 -f s16be -ar 16000 -ac 1 -acodec pcm_s16be pcm16k.pcm
```



### 小端数据格式

```shell
ffmpeg -i test.mp3 -f s16le -ar 16000 -ac 1 -acodec pcm_s16le pcm16k.pcm
```

 

> 说明:
>
> 1. -acodec pcm_s16be：输出pcm格式，采用signed 16编码，字节序为大尾端（小尾端为le)；
> 2. -ar 16000: 采样率为16000
> 3. -ac 1: 声道数为1











如何播放录制各个音频格式的软件？

## g726编码音频：ffplay

```shell
#ffplay 可以播放。
ffplay -f g726le  -ar 8000 -ac 1 -code_size 5 -i  xxx.g726
#如果是大端序就写  -f g726
#如果不是40kbps的，比如是32kbps的就写 -code_size = 4  等等(取值是2~5)
```



[音视频学习之ffplay基础命令整理](https://blog.csdn.net/yun6853992/article/details/121870678)





安装编译特别麻烦



安装完还会出现这个问题



```
Could not initialize SDL - No available video device
```

https://www.dandelioncloud.cn/article/details/1441232425681801217

也就是上面链接最后出来的错误，针对这个错误需要重新编译

要重新编译SDL库

```shell
./configure --enable-sdl
```



# ffplay播放PCM

```shell
ffplay -ar 44100 -ac 1 -f s16le -i ./201904091310_test.pcm

-ar 表示采样率

-ac 表示音频通道数
	单声道是 1，Android 中为 AudioFormat.CHANNEL_IN_MONO
	双声道是 2，Android 中为 AudioFormat.CHANNEL_IN_STEREO

-f 表示 pcm 格式，sample_fmts + le(小端)或者 be(大端)
	sample_fmts可以通过ffplay -sample_fmts来查询

-i 表示输入文件，这里就是 pcm 文件
```





