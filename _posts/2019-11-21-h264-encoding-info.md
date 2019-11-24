---
layout: post
title: H.264 编码基础知识
categories: H.264
description: H.264 编码基础知识
keywords: H.264
---

说明，以下内容很多参考 [从零了解H.264结构](http://www.iosxxx.com/blog/2017-08-09-%E4%BB%8E%E9%9B%B6%E4%BA%86%E8%A7%A3H.264%E7%BB%93%E6%9E%84.html)


## 基本资料
H.264，又称为MPEG-4第10部分，高级视频编码（英语：MPEG-4 Part 10, Advanced Video Coding，缩写为MPEG-4 AVC）是一种面向块，基于运动补偿的视频编码标准。由[ITU-T](https://zh.wikipedia.org/wiki/ITU-T)视频编码专家组与[ISO](https://zh.wikipedia.org/wiki/ISO)/[IEC](https://zh.wikipedia.org/wiki/IEC)联合工作组开发。

## H.264 协议标准
- [ITU-T H.264](https://www.itu.int/ITU-T/recommendations/rec.aspx?rec=13189)
- [H.264 : Advanced video coding for generic audiovisual services](https://www.itu.int/rec/dologin_pub.asp?lang=e&id=T-REC-H.264-201704-S!!PDF-E&type=items)

## 音视频基础知识
在线视频兴起之前，互联网上看片的主要方式是下载到电脑后观看。比如下载了一个 ```test.mp4``` 的视频，这个 ```mp4``` 是一个容器，又叫封装格式,就是把已经编码封装好的视频、音频按照一定的规范放到一起。常见的容器格式有： avi, mp4, mov, ts, mkv, rmvb/rm, wmv, flv, 3gp, asf, webm 。
裸视频是非常大的，所以需要对视频做编码压缩，常见的视频编码格式有： mpeg-1, mpeg-2, mpeg-4, H.264/AVC/mpeg-4part 10, h.265/hevc, vc-1, RealVideo, AVS 。

<img src="/images/posts/h264/01.png">


## H.264 视频编码
视频编码主要是利用空间、时间的冗余信息进行编码达到压缩的目的。空间冗余信息压缩是做每一帧图片内的压缩，跟 jpeg 图片压缩类似，可能会是降采样、联合周边像素信息等方式来进行编码。 时间冗余信息主要是参考前后帧的信息来进行编码，比如前后两帧的图片通常是有很多像素点是一样的，比如要表示两针图片的信息，能想到最简单的方式就是用第一帧的全量信息+第二帧相对第一帧有变化的变量信息来表示。H.264 的编码很复杂，但是大体的原理就是利用空间、时间信息来编码。
因为有利用时间信息来编码，所以必然要利用多帧的图片来编码，所以需要缓存多帧信息，给编解码以及视频流带来一些需要权衡的问题，比如要利用更多前后帧信息来进行编码，能够提高压缩比，但是相应的延迟就会比较高，对于实时视频推流是需要权衡的；另外解码顺序和渲染顺序也需要注意。

### 简介
H.264 编码功能分为两层，VCL(视频编码层/Video Coding Layer)和 NAL(网络提取层/Network Abstraction Layer)。VCL 主要分为 5 部分： 帧间和帧内预测（Estimation）、变换（Transform）和反变换、量化（Quantization）和反量化、环路滤波（Loop Filter）、熵编码（Entropy Coding）。VCL 负责怎么高效的进行编码，编码后的数据怎么进行存储和传输由 NAL 来负责，存到了NAL单元（NALU）。一帧图片经过 H.264 编码器之后，就被编码为一个或多个片（slice），而装载着这些片（slice）的载体，就是 NALU 了。片（slice）是 H.264 中提出的新概念，是通过编码图片后切分通过高效的方式整合出来的概念，一张图片至少有一个或多个片（slice）。除了装载编码数据的slice，有些NALU装载了编码的一些 meta 信息，比如 SPS(Sequence parameter set), PPS(Picture parameter set)。

<img src="/images/posts/h264/h264-frametoNALU.png">

<img src="/images/posts/h264/h264-frametoNALU-2.png">

- 1 Frame (帧) = 1..n个Slice (片)1 Slice (片) = 1..n个Marcoblock(宏块)1 Marcoblock(宏块) = 16x16yuv数据
- 1 Slice (片) = Slice Header + Slice Data
- 1 NALU = 一组对应于视频编码的NALU头部信息 + 一个原始字节序列负荷(RBSP,Raw Byte Sequence Payload).


### NALU
H.264的结构全部都是以 NALU 为主，理解了 NALU，就理解了 H.264 的结构。一个 NALU 由 NAL头+RBSP组成。一个原始的 H.264 NALU 单元常由 ```[StartCode] [NALU Header] [NALU Payload]``` 三部分组成，其中 Start Code 用于标示这是一个 NALU 单元的开始，必须是 ```00 00 00 01``` 或 ```00 00 01``` 。

<img src="/images/posts/h264/minimal_yuv420_hex.png">

### NALU Header
NAL Header 由三部分组成，F/forbidden_bit(1bit)，NRI/nal_reference_bit(2bits)（优先级），Type/nal_unit_type(5bits)（类型）。
```bash
+---------------+ 
|0|1|2|3|4|5|6|7| 
+-+-+-+-+-+-+-+-+ 
|F|NRI| Type    |
+---------------+ 
```

<img src="/images/posts/h264/h264-nalheader-02.png">

比较经常会用到的 NALU 类型有：
```bash
00 00 00 01 06:  SEI信息   
00 00 00 01 67:  0x67&0x1f = 0x07 :SPS
00 00 00 01 68:  0x68&0x1f = 0x08 :PPS
00 00 00 01 65:  0x65&0x1f = 0x05: IDR Slice
00 00 00 01 41:  0x41&0x1f = 0x01: Coded slice of a non-IDR picture
```
参数集是 H.264 标准的一个新概念，是一种通过改进视频码流结构增强错误恢复能力的方法。
SPS序列参数集 （包括一个图像序列的所有信息，即两个 IDR 图像间的所有图像信息，如图像尺寸、视频格式等）。
PPS图像参数集 （包括一个图像的所有分片的所有相关信息， 包括图像类型、序列号等，解码时某些序列号的丢失可用来检验信息包的丢失与否）


### 分片(Slice)
我们可以理解为一 张/帧 图片可以包含一个或多个分片(Slice)，而每一个分片(Slice)包含整数个宏块(Macroblock)，即每片（Slice）至少一个宏块(Macroblock)。 ```Slice = Slice Header + Slice Data```

<img src="/images/posts/h264/slice.png">

1. 分片头中包含着分片类型、分片中的宏块类型、分片帧的数量、分片属于那个图像以及对应的帧的设置和参数等信息。
2. 分片数据中则是宏块，这里就是我们要找的存储像素数据的地方。

**有五种分片类型**：  

| Slice | 内容 | 
| :----- | :----- | 
| I Slice | 只包含I宏块 |
| P Slice | 包含P和I宏块 |
| B Slice | 包含B和I宏块 |
| SP Slice | 包含P 和/或 I宏块,用于不同码流之间的切换 |
| SI Slice | 一种特殊类型的编码宏块 |


### 宏块(Macroblock)
宏块是视频信息的主要承载者。一个宏块由一个16×16亮度像素和附加的一个8×8 Cb和一个 8×8 Cr 彩色像素块组成。每个图象中，若干宏块被排列成片的形式。
<img src="/images/posts/h264/macroblock.png">

**宏块分类**：  
| 宏块分类 | 说明 | 
| :----- | :----- | 
| I Macroblock | 利用从当前片中已解码的像素作为参考进行帧内预测 |
| P Macroblock | 利用前面已编码图像作为参考进行帧内预测，一个帧内编码的宏块可进一步作宏块的分割:即16×16.16×8.8×16.8×8亮度像素块。如果选了8×8的子宏块，则可再分成各种子宏块的分割，其尺寸为8×8，8×4，4×8，4×4 |
| B Macroblock | 利用双向的参考图像(当前和未来的已编码图像帧)进行帧内预测 |

### I,P,B帧与 PTS/DTS  
| 帧的分类 | 名称 | 说明 | 
| :----- | :----- | :----- | 
| I帧 | 帧内编码帧,又称intra picture | 自身可以通过视频解压算法解压成一张单独的完整的图片 |
| P帧 | 前向预测编码帧,又称predictive-frame | 需要参考其前面的一个I frame 或者B frame来生成一张完整的图片 |
| I帧 | 双向预测帧,又称bi-directional interpolated prediction frame | 则要参考其前一个I或者P帧及其后面的一个P帧来生成一张完整的图片 |
  
| 名称 | 说明 | 
| :----- | :----- | 
| PTS(Presentation Time Stamp) | PTS主要用于度量解码后的视频帧什么时候被显示出来 |
| DTS(Decode Time Stamp) | DTS主要是标识内存中的bit流再什么时候开始送入解码器中进行解码 |

### GOP(Group Of Pictures)
GOP （图像组）主要用作形容一个 i 帧 到下一个 i 帧之间的间隔了多少个帧。一个I帧所占用的字节数大于一个P帧，一个P帧所占用的字节数大于一个B帧。在码率不变的前提下，GOP值越大，P、B帧的数量会越多，平均每个I、P、B帧所占用的字节数就越多，也就更容易获取较好的图像质量。在遇到场景切换的情况时，H.264编码器会自动强制插入一个I帧，此时实际的GOP值被缩短了。另一方面，在一个GOP中，P、B帧是由I帧预测得到的，当I帧的图像质量比较差时，会影响到一个GOP中后续P、B帧的图像质量，直到下一个GOP开始才有可能得以恢复，所以GOP值也不宜设置过大。由于P、B帧的复杂度大于I帧，所以过多的P、B帧会影响编码效率，使编码效率降低。

### IDR(Instantaneous Decoding Refresh)即时解码刷新
一个序列的第一个图像叫做 IDR 图像（立即刷新图像），IDR 图像都是 I 帧图像。
I和IDR帧都使用帧内预测。I帧不用参考任何帧，但是之后的P帧和B帧是有可能参考这个I帧之前的帧的。IDR就不允许这样。
比如这种情况:
IDR1 P4 B2 B3 P7 B5 B6 I10 B8 B9 P13 B11 B12 P16 B14 B15 这里的B8可以跨过I10去参考P7

核心作用：
H.264 引入 IDR 图像是为了解码的重同步，当解码器解码到 IDR 图像时，立即将参考帧队列清空，将已解码的数据全部输出或抛弃，重新查找参数集，开始一个新的序列。这样，如果前一个序列出现重大错误，在这里可以获得重新同步的机会。IDR图像之后的图像永远不会使用IDR之前的图像的数据来解码。




## 附录

### 将一张图片转成 H.264 视频
```bash
ffmpeg -i minimal.png -pix_fmt yuv420p minimal_yuv420.h264
```

### 下载测试视频
要学习 H.264 编码，就需要有用 H.264 编码的视频 demo 来测试，可以用 [you-get](https://github.com/soimort/you-get) 工具到主流的一些视频网站下载个视频来测试。

```bash
➜  test you-get -i https://www.bilibili.com/video/av2650963\?from\=search\&seid\=15782581299335183738
site:                Bilibili
title:               Tribute to Hayao Miyazaki（致敬宫崎骏）
streams:             # Available quality and codecs
    [ DEFAULT ] _________________________________
    - format:        flv
      container:     flv
      quality:       高清 1080P
      size:          42.4 MiB (44452950 bytes)
    # download-with: you-get --format=flv [URL]

    - format:        flv720
      container:     flv
      quality:       高清 720P
      size:          42.4 MiB (44452999 bytes)
    # download-with: you-get --format=flv720 [URL]

    - format:        flv360
      container:     flv
      quality:       流畅 360P
      size:          9.0 MiB (9452072 bytes)
    # download-with: you-get --format=flv360 [URL]
➜  test you-get --format=flv360 https://www.bilibili.com/video/av2650963\?from\=search\&seid\=15782581299335183738
```

### 视频格式转换
视频格式转换大部分都可以用 ffmpeg 来实现。

```bash
ffmpeg -i input.flv output.mp4
```
此过程需要对视频进行重新编码，比较耗费 CPU 等资源。如果不做转码，可以用如下的方式：
```bash
ffmpeg -i input.flv -vcodec copy -acodec copy output.mp4
```


#### 将视频转换为非压缩的裸视频
将视频 ```input.mp4``` 转换为 ```pix_fmt nv12``` yuv 的裸视频 ```output.yuv``` ， 裸视频可以作为后续视频编码的 input 。
```bash
ffmpeg -i input.mp4 -vcodec rawvideo -vframes 100 -pix_fmt nv12 -an output.yuv
```

#### flv 转 h264
将视频 ```input.flv``` 转换为 ```output.h264```
```bash
ffmpeg -i input.flv -vcodec copy output.h264
```

#### mp4 转 mkv
将视频 ```input.mp4``` 转换为 ```output.mkv```
```bash
ffmpeg -i input.mp4 -vcodec copy -acodec copy output.mkv
```


## 参考资料
- [H.264 : Advanced video coding for generic audiovisual services](https://www.itu.int/rec/dologin_pub.asp?lang=e&id=T-REC-H.264-201704-S!!PDF-E&type=items)
- [从零了解H.264结构](http://www.iosxxx.com/blog/2017-08-09-%E4%BB%8E%E9%9B%B6%E4%BA%86%E8%A7%A3H.264%E7%BB%93%E6%9E%84.html)
