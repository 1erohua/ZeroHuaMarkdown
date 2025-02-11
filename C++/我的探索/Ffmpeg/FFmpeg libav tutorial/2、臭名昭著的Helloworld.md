```cpp
/*
 * FFmpeg视频解码示例 - 详细注释版
 * 主要功能：读取视频文件，解码视频流，并将前8帧保存为灰度PGM图像
 * 核心组件说明：
 *   - 容器格式（Format）：处理封装格式（如MP4/MKV），负责解复用
 *   - 流（Stream）：连续的媒体数据流（视频/音频/字幕）
 *   - 编解码器（Codec）：定义数据编码/解码方式
 *   - 数据包（Packet）：压缩后的数据单元（解码前的输入）
 *   - 帧（Frame）：解码后的原始数据（输出）
 */

extern "C"{
    #include <libavcodec/avcodec.h>   // 编解码相关函数
    #include  <libavformat/avformat.h>
    #include <cstdio>
}


// 日志输出函数
static void logging(const char *fmt, ...);

// 数据包解码函数
static int decode_packet(const AVPacket *pPacket,
                        AVCodecContext *pCodecContext,
                        AVFrame *pFrame);

// 保存灰度帧为PGM文件
static void save_gray_frame(const unsigned char *buf,
                           int wrap,
                           int xsize,
                           int ysize,
                           const char *filename);

int main(const int argc, const char *argv[]) {
    // 参数检查
    if (argc < 2) {
        printf("用法：%s <媒体文件>\n", argv[0]);
        return -1;
    }

    /* 第一部分：初始化FFmpeg组件 */
    logging("初始化容器、编解码器和协议");

    // 分配格式上下文（存储文件头信息）
    AVFormatContext *pFormatContext = avformat_alloc_context();
    if (!pFormatContext) {
        logging("错误：无法分配格式上下文内存");
        return -1;
    }

    /* 第二部分：打开输入文件 */
    logging("打开文件 %s 并加载容器头信息", argv[1]);
    if (avformat_open_input(&pFormatContext, argv[1], nullptr, nullptr) != 0) {
        logging("错误：无法打开文件");
        return -1;
    }// 华：应该是将输入信息全部送给了pFormatContext

    // 显示基础信息
    logging("容器格式：%s，时长：%lld微秒，比特率：%lld",
           pFormatContext->iformat->name,
           pFormatContext->duration,
           pFormatContext->bit_rate);

    /* 第三部分：获取流信息 */
    logging("从容器中获取流信息");
    if (avformat_find_stream_info(pFormatContext, nullptr) < 0) {
        logging("错误：无法获取流信息");
        return -1;
    }

    /* 第四部分：查找视频流并初始化解码器 */
    AVCodec *pCodec = nullptr;
    AVCodecParameters *pCodecParameters = nullptr;
    int video_stream_index = -1;

    // 遍历所有流，寻找视频流
    for (int i = 0; i < pFormatContext->nb_streams; i++) {
        AVCodecParameters *pLocalCodecParameters = pFormatContext->streams[i]->codecpar;

        // 查找对应的解码器
        AVCodec *pLocalCodec = avcodec_find_decoder(pLocalCodecParameters->codec_id);
        if (!pLocalCodec) {
            logging("警告：找不到解码器，跳过流#%d", i);
            continue;
        }

        // 记录第一个视频流
        if (pLocalCodecParameters->codec_type == AVMEDIA_TYPE_VIDEO) {
            if (video_stream_index == -1) { // 只取第一个视频流
                video_stream_index = i;
                pCodec = pLocalCodec;
                pCodecParameters = pLocalCodecParameters;
                logging("找到视频流#%d，分辨率：%dx%d",
                       i,
                       pLocalCodecParameters->width,
                       pLocalCodecParameters->height);
            }
        }
    }

    if (video_stream_index == -1) {
        logging("错误：文件中没有视频流");
        return -1;
    }

    /* 第五部分：配置解码器上下文 */
    AVCodecContext *pCodecContext = avcodec_alloc_context3(pCodec);
    if (!pCodecContext) {
        logging("错误：无法分配编解码器上下文内存");
        return -1;
    }

    // 将流参数复制到编解码器上下文
    if (avcodec_parameters_to_context(pCodecContext, pCodecParameters) < 0) {
        logging("错误：无法复制编解码器参数");
        return -1;
    }

    // 打开解码器
    if (avcodec_open2(pCodecContext, pCodec, nullptr) < 0) {
        logging("错误：无法打开解码器");
        return -1;
    }

    /* 第六部分：准备解码资源 */
    AVFrame *pFrame = av_frame_alloc();    // 存储解码后的帧
    AVPacket *pPacket = av_packet_alloc(); // 存储压缩数据包
    if (!pFrame || !pPacket) {
        logging("错误：无法分配帧/包内存");
        return -1;
    }

    /* 第七部分：主解码循环 */
    int how_many_packets_to_process = 8; // 限制处理8个数据包（演示用）
    while (av_read_frame(pFormatContext, pPacket) >= 0) {
        if (pPacket->st ream_index == video_stream_index) {
            logging("处理视频包，PTS：%lld", pPacket->pts);
            if (decode_packet(pPacket, pCodecContext, pFrame) < 0)
                break;
 
            // 达到处理限制后退出
            if (--how_many_packets_to_process <= 0) break;
        }
        av_packet_unref(pPacket); // 释放包内存
    }

    /* 第八部分：资源清理 */
    logging("释放所有资源");
    avformat_close_input(&pFormatContext);
    av_packet_free(&pPacket);
    av_frame_free(&pFrame);
    avcodec_free_context(&pCodecContext);
    return 0;
}

//-----------------------------------------
// 函数实现部分
//-----------------------------------------

/* 日志输出实现 */
static void logging(const char *fmt, ...) {
    va_list args;
    fprintf(stderr, "LOG: ");
    va_start(args, fmt);
    vfprintf(stderr, fmt, args);
    va_end(args);
    fprintf(stderr, "\n");
}

/* 数据包解码实现 */
static int decode_packet(const AVPacket *pPacket,
                        AVCodecContext *pCodecContext,
                        AVFrame *pFrame) {
    // 发送压缩数据包到解码器
    int response = avcodec_send_packet(pCodecContext, pPacket);

    if (response < 0) {
        char err_buf[AV_ERROR_MAX_STRING_SIZE] = {0};
        av_strerror(response, err_buf, sizeof(err_buf));
        logging("解码错误：%s", err_buf);

        return response;
    }


    // 循环接收解码后的帧
    while (response >= 0) {
        response = avcodec_receive_frame(pCodecContext, pFrame);
        if (response == AVERROR(EAGAIN) || response == AVERROR_EOF) {
            break; // 需要更多数据或结束
        } else if (response < 0) {
            char err_buf[AV_ERROR_MAX_STRING_SIZE] = {0};
            av_strerror(response, err_buf, sizeof(err_buf));
            logging("接受帧错误：%s", err_buf);
            return response;
        }

        // 成功解码帧时的处理
        logging("解码帧#%d（类型：%c，大小：%d字节，格式：%d）",
               pCodecContext->frame_number,
               av_get_picture_type_char(pFrame->pict_type),
               pFrame->pkt_size,
               pFrame->format);

        // 生成文件名并保存帧
        char frame_filename[1024];
        snprintf(frame_filename, sizeof(frame_filename),
                "frame-%d.pgm", pCodecContext->frame_number);

        // 格式检查（本示例仅处理YUV420P格式）
        if (pFrame->format != AV_PIX_FMT_YUV420P) {
            logging("警告：非YUV420P格式可能导致非灰度图像");
        }

        // 保存Y分量（灰度图像）
        save_gray_frame(pFrame->data[0],  // Y分量数据
                      pFrame->linesize[0], // 行跨度
                      pFrame->width,
                      pFrame->height,
                      frame_filename);
    }
    return 0;
}

/* 保存灰度帧实现（PGM格式） */
static void save_gray_frame(const unsigned char *buf,
                           int wrap,  // 行跨度（可能包含填充字节）
                           int xsize, // 实际宽度
                           int ysize,
                           const char *filename) {
    FILE *f = fopen(filename, "wb");
    // 写入PGM文件头
    fprintf(f, "P5\n%d %d\n255\n", xsize, ysize);

    // 逐行写入Y分量数据（跳过可能的填充字节）
    for (int i = 0; i < ysize; i++) {
        fwrite(buf + i * wrap, 1, xsize, f);
    }
    fclose(f);
}

```



# 随笔
```cpp
/* 第二部分：打开输入文件 */logging("打开文件 %s 并加载容器头信息", argv[1]);  
if (avformat_open_input(&pFormatContext, argv[1], nullptr, nullptr) != 0) {  
    logging("错误：无法打开文件");  
    return -1;  
}// 华：应该是将输入信息全部送给了pFormatContext
```

打开输入文件，应该是由pformatContext存储文件内的信息

```cpp
// 显示基础信息
    logging("容器格式：%s，时长：%lld微秒，比特率：%lld",
           pFormatContext->iformat->name,
           pFormatContext->duration,
           pFormatContext->bit_rate);

```
`pFormatContext`中有不少的其他信息，能存挺多东西

```cpp
/* 第三部分：获取流信息 */
    logging("从容器中获取流信息");
    if (avformat_find_stream_info(pFormatContext, nullptr) < 0) {
        logging("错误：无法获取流信息");
        return -1;
    }
```
看得出来一件事，应该不是一下子给`pFormatContext`灌注全部信息的
应该是通过多个函数按照需求来给`pFormatContext`灌注信息
**所以avformat_find_stream_info能获取该视频的所有流信息！
可以使用循环来获取所有流数据：

```c
for (int i = 0; i < pFormatContext->nb_streams; i++)
{
  //
}
```



**每个流都维护了一个AVCodeParameters, 该结构体描述了被编码流的各种信息**
通过codec id和 [`avcodec_find_decoder`](https://ffmpeg.org/doxygen/trunk/group__lavc__decoding.html#ga19a0ca553277f019dd5b0fec6e1f9dca) 函数可以找到对应已经注册的解码器，返回 [`AVCodec`](https://ffmpeg.org/doxygen/trunk/structAVCodec.html) 指针，**该组件能让我们知道如何编解码这个流。**

```c
AVCodec *pLocalCodec = avcodec_find_decoder(pLocalCodecParameters->codec_id);
```



```cpp
    if (pLocalCodecParameters->codec_type == AVMEDIA_TYPE_VIDEO) {
            if (video_stream_index == -1) { // 只取第一个视频流
                video_stream_index = i;
                pCodec = pLocalCodec;
                pCodecParameters = pLocalCodecParameters;
                logging("找到视频流#%d，分辨率：%dx%d",
                       i,
                       pLocalCodecParameters->width,
                       pLocalCodecParameters->height);
     
```
可以发现
![[Pasted image 20250209154627.png]]
有不少关于视频中的流的宏
它们都是通过 `codec_type`而存储的，存放在了`AVCodeParameters`，所以应该是AVCodeParameters存放所有关于解码的信息，然后再由avcodec_find_decoder获取到它的解码器的型号AVCodec


获取解码器型号就能配置解码器上下文了
```cpp
/* 第五部分：配置解码器上下文 */AVCodecContext *pCodecContext = avcodec_alloc_context3(pCodec);  
if (!pCodecContext) {  
    logging("错误：无法分配编解码器上下文内存");  
    return -1;  
}
```
	


**利用刚刚获取的 `AVCodec` 为 [`AVCodecContext`](https://ffmpeg.org/doxygen/trunk/structAVCodecContext.html) 分配内存，它将维护解码/编码过程的上下文。 然后需要使用 [`avcodec_parameters_to_context`](https://ffmpeg.org/doxygen/trunk/group__lavc__core.html#gac7b282f51540ca7a99416a3ba6ee0d16)和被编码流的参数(`AVCodecParameters`) 来填充 `AVCodecContext`。** 就如同对待AVFormatContext一样。

完成上下文填充后，使用 [`avcodec_open2`](https://ffmpeg.org/doxygen/trunk/group__lavc__core.html#ga11f785a188d7d9df71621001465b0f1d) 来打开解码器。
```cpp
// 配置解码器上下文
AVCodecContext *pCodecContext = avcodec_alloc_context3(pCodec);

// 将流参数复制到编解码器上下文
avcodec_parameters_to_context(pCodecContext, pCodecParameters);

// 打开解码器
avcodec_open2(pCodecContext, pCodec, NULL);
```
	

现在我们将**从流中读取数据包并将它们解码为帧**。但首先，需要为 [`AVPacket`](https://ffmpeg.org/doxygen/trunk/structAVPacket.html) 和 [`AVFrame`](https://ffmpeg.org/doxygen/trunk/structAVFrame.html) 分配内存。
```cpp
AVPacket *pPacket = av_packet_alloc();
AVFrame *pFrame = av_frame_alloc();
```


使用函数 [`av_read_frame`](https://ffmpeg.org/doxygen/trunk/group__lavf__decoding.html#ga4fdb3084415a82e3810de6ee60e46a61) 读取帧数据来填充数据包。
```c
while (av_read_frame(pFormatContext, pPacket) >= 0) {
  //...
}
```

太好了，我逐渐理解一切：
- AVPacket（数据包）：相当于未拆封的快递包裹，**存储着压缩后的编码数据（如H.264/H.265视频数据）**
- AVFrame（帧）：相当于拆封后的物品，存储解码后的原始数据（如YUV/RGB像素数据或PCM音频数据）
[[AVPacket和AVFrame]]

步骤流程大概就是：
1. `AVFormatContext` 通过 `avformat_alloc_context` 获取 到音视频频文件的上下文信息
2. 利用 `avformat_open_input` **打开文件**、利用 `avformat_open_input`**获取流信息**，都是为了在之后**获取到对应流的编解码器** (`avcodec_find_decoder`)
3. 获取到编解码器之后就配置 编解码器的上下文内容，复制流参数，然后**打开编解码器**
4. 初始化包和帧，**从文件头不停读取信息到包中，将包发送到解码器，解码器对包依次解码返回给帧**，再对帧进行处理





``





























































# AI分析
以下是对代码中涉及的FFmpeg关键函数、结构体及操作的详细解析：

---

### **一、核心结构体解析**
5. **AVFormatContext**
   - **作用**：容器格式的上下文，存储媒体文件的封装信息（如MP4/MKV头信息、流信息、时长、比特率等）
   - **关键字段**：
     - `nb_streams`：流的数量
     - `streams[]`：流指针数组
     - `duration`：文件总时长（微秒）
     - `bit_rate`：全局比特率

6. **AVCodecContext**
   - **作用**：编解码器上下文，存储编解码器参数和状态
   - **关键字段**：
     - `codec_type`：媒体类型（视频/音频等）
     - `width/height`：视频分辨率
     - `pix_fmt`：像素格式（如YUV420P）
     - `time_base`：时间基准（用于时间戳计算）

7. **AVCodec**
   - **作用**：描述编解码器的能力（如H.264解码器）
   - **关键方法**：
     - `avcodec_find_decoder()`：通过ID查找解码器

8. **AVPacket**
   - **作用**：存储压缩后的数据包（解码前）
   - **关键字段**：
     - `data`：压缩数据指针
     - `size`：数据大小
     - `pts/dts`：显示/解码时间戳
     - `stream_index`：所属流的索引

9. **AVFrame**
   - **作用**：存储解码后的原始数据（如视频帧）
   - **关键字段**：
     - `data[]`：各分量数据指针（如YUV的三个平面）
     - `linesize[]`：每行的字节数（考虑内存对齐）
     - `width/height`：帧的宽高
     - `format`：像素格式

---

### **二、核心函数解析**
10. **初始化与文件操作**
   - `avformat_alloc_context()`: 创建AVFormatContext
   - `avformat_open_input(&ctx, filename, ...)`: 打开媒体文件并读取头信息
   - `avformat_find_stream_info()`: 探测流信息（可能读取部分数据）

11. **流与编解码器处理**
   - `avcodec_find_decoder(codec_id)`: 根据编码ID查找解码器
   - `avcodec_parameters_to_context()`: 将流参数复制到编解码器上下文
   - `avcodec_open2()`: 初始化编解码器上下文

12. **数据读取与解码**
   - `av_read_frame()`: 读取下一个数据包（可能来自任意流）
   - `avcodec_send_packet()`: 发送压缩包到解码器
   - `avcodec_receive_frame()`: 从解码器接收解码后的帧

13. **资源管理**
   - `av_packet_unref()`: 释放数据包引用
   - `avformat_close_input()`: 关闭输入文件并释放上下文
   - `avcodec_free_context()`: 释放编解码器上下文

---

### **三、关键操作流程**
14. **初始化阶段**
   ```cpp
   AVFormatContext *pFormatContext = avformat_alloc_context();
   avformat_open_input(&pFormatContext, filename, NULL, NULL);
   avformat_find_stream_info(pFormatContext, NULL);
   ```
   - 创建格式上下文 → 打开文件 → 探测流信息

15. **查找视频流**
   ```cpp
   for (遍历所有流) {
       if (流类型为视频 && 找到第一个视频流) {
           video_stream_index = i;
           pCodec = avcodec_find_decoder(codec_id);
       }
   }
   ```
   - 遍历所有流，筛选出视频流并记录索引

16. **初始化解码器**
   ```cpp
   AVCodecContext *pCodecContext = avcodec_alloc_context3(pCodec);
   avcodec_parameters_to_context(pCodecContext, codecpar);
   avcodec_open2(pCodecContext, pCodec, NULL);
   ```
   - 创建编解码器上下文 → 复制参数 → 打开解码器

17. **解码循环**
   ```cpp
   while (av_read_frame() >= 0) {
       if (是视频流数据包) {
           avcodec_send_packet();
           while (avcodec_receive_frame() >= 0) {
               // 处理解码后的帧
           }
       }
       av_packet_unref();
   }
   ```
   - 读取数据包 → 发送到解码器 → 循环接收帧

18. **帧处理与保存**
   ```cpp
   save_gray_frame(pFrame->data[0], ...); // 仅保存Y分量
   ```
   - 对YUV420P格式，Y分量即亮度（灰度）数据
   - PGM文件格式：P5头部 + 二进制灰度数据

---

### **四、技术细节与注意事项**
19. **时间戳处理**
   - `pPacket->pts` 存储的是显示时间戳，需结合 `time_base` 转换为秒
   - 示例代码未处理时间戳转换，实际应用可能需要

20. **解码器状态管理**
   - `avcodec_send_packet()` 返回 `EAGAIN` 表示需要先接收帧
   - `avcodec_receive_frame()` 返回 `EAGAIN` 表示需要更多输入数据

21. **内存对齐**
   - `AVFrame->linesize` 可能大于实际宽度（因内存对齐优化）
   - 写入文件时需要按 `linesize` 逐行读取数据

22. **格式兼容性**
   - 示例仅处理YUV420P，实际视频可能使用其他格式（如NV12、RGB）
   - 需添加像素格式转换逻辑（如使用 `sws_scale()`）

23. **错误处理**
   - 所有FFmpeg函数调用后应检查返回值
   - 使用 `av_strerror()` 将错误码转换为可读信息

---

### **五、扩展知识**
24. **硬件加速解码**
   - 可通过设置 `AVCodecContext->hw_device_ctx` 启用
   - 支持CUDA、VAAPI、DXVA2等

25. **多线程解码**
   - 设置 `AVCodecContext->thread_count` 开启多线程
   - FFmpeg支持帧级/切片级多线程

26. **时间基转换**
   ```cpp
   double seconds = packet.pts * av_q2d(stream->time_base);
   ```
   - 使用 `av_q2d()` 将分数时间基转换为浮点数

---

通过以上分析，可以清晰理解FFmpeg解码视频的核心流程和关键API的使用方式。实际开发中需根据需求扩展错误处理、格式转换等功能。


# 流信息可不止两种


在FFmpeg中，**流（Stream）的类型远不止视频流和音频流两种**。以下是完整的流类型分类和说明：

---

### **一、FFmpeg流类型（AVMediaType枚举）**
通过 `AVStream->codecpar->codec_type` 可判断流类型：

| 类型常量                  | 值 | 说明                                                                 |
|---------------------------|---|--------------------------------------------------------------------|
| `AVMEDIA_TYPE_VIDEO`      | 0 | **视频流**（包含压缩的视频数据，如H.264/HEVC/VP9等）                     |
| `AVMEDIA_TYPE_AUDIO`      | 1 | **音频流**（包含压缩的音频数据，如AAC/MP3/Opus等）                       |
| `AVMEDIA_TYPE_SUBTITLE`   | 3 | **字幕流**（文本字幕：SRT/ASS，或图形字幕：DVD SUB/PGS）                  |
| `AVMEDIA_TYPE_DATA`       | 4 | **数据流**（附加数据，如时间码、章节信息等）                               |
| `AVMEDIA_TYPE_ATTACHMENT` | 5 | **附件流**（嵌入文件，如字体、封面图等）                                   |
| `AVMEDIA_TYPE_NB`         | 6 | 类型总数（非实际类型）                                                     |

---

### **二、各类型流的典型特征**
#### 1. **视频流**
```c
if (codecpar->codec_type == AVMEDIA_TYPE_VIDEO) {
    // 处理视频流
    printf("分辨率：%dx%d\n", codecpar->width, codecpar->height);
}
```
- 必含字段：`width`, `height`, `format`（像素格式）
- 常见编码格式：H.264 (AVC), HEVC (H.265), VP9, AV1

#### 2. **音频流**
```c
if (codecpar->codec_type == AVMEDIA_TYPE_AUDIO) {
    // 处理音频流
    printf("采样率：%dHz，声道数：%d\n", 
           codecpar->sample_rate, 
           codecpar->channels);
}
```
- 必含字段：`sample_rate`, `channels`, `format`（采样格式）
- 常见编码格式：AAC, MP3, Opus, FLAC

#### 3. **字幕流**
```c
if (codecpar->codec_type == AVMEDIA_TYPE_SUBTITLE) {
    // 处理字幕流
    printf("字幕格式：%s\n", avcodec_get_name(codecpar->codec_id));
}
```
- 文本字幕：SRT、ASS、SSA（编码ID如 `AV_CODEC_ID_ASS`）
- 图形字幕：DVD SUB（`AV_CODEC_ID_DVD_SUBTITLE`）、PGS（`AV_CODEC_ID_HDMV_PGS_SUBTITLE`）

#### 4. **数据流**
```c
if (codecpar->codec_type == AVMEDIA_TYPE_DATA) {
    // 处理数据流（如时间码）
    printf("数据流标识：%s\n", avcodec_get_name(codecpar->codec_id));
}
```
- 常见用途：携带时间码（`AV_CODEC_ID_TIMECODE`）、章节信息等

#### 5. **附件流**
```c
if (codecpar->codec_type == AVMEDIA_TYPE_ATTACHMENT) {
    // 提取附件（如字体文件）
    printf("附件文件名：%s\n", av_dict_get(stream->metadata, "filename", NULL, 0)->value);
}
```
- 典型内容：MKV中的字体文件（.ttf）、封面图（提取后保存为JPEG/PNG）

---

### **三、代码示例：遍历所有流类型**
```c
for (int i = 0; i < pFormatContext->nb_streams; i++) {
    AVStream *stream = pFormatContext->streams[i];
    AVCodecParameters *codecpar = stream->codecpar;

    switch (codecpar->codec_type) {
        case AVMEDIA_TYPE_VIDEO:
            printf("视频流#%d\n", i);
            break;
        case AVMEDIA_TYPE_AUDIO:
            printf("音频流#%d\n", i);
            break;
        case AVMEDIA_TYPE_SUBTITLE:
            printf("字幕流#%d\n", i);
            break;
        case AVMEDIA_TYPE_DATA:
            printf("数据流#%d\n", i);
            break;
        case AVMEDIA_TYPE_ATTACHMENT:
            printf("附件流#%d\n", i);
            break;
        default:
            printf("未知流类型#%d\n", i);
    }
}
```

---

### **四、特殊场景分析**
#### 场景1：MKV文件中的多字幕
- 可能包含多个 `AVMEDIA_TYPE_SUBTITLE` 流
- 不同字幕流可能对应不同语言（通过 `stream->metadata` 中的 `language` 字段区分）

#### 场景2：MP4中的章节信息
- 可能通过 `AVMEDIA_TYPE_DATA` 流存储章节
- 需要使用特定方式解析（如 `avformat_index_get_entries`）

#### 场景3：直播流中的元数据
- 某些直播协议（如RTMP）可能包含 `AVMEDIA_TYPE_DATA` 流传递实时信息

---

### **五、开发注意事项**
27. **流选择策略**
   - 一个文件可能包含多个同类型流（如多音轨、多字幕）
   - 需通过 `stream->disposition` 或元数据选择默认流

28. **附件流提取**
   ```c
   if (codecpar->codec_type == AVMEDIA_TYPE_ATTACHMENT) {
       const char *filename = av_dict_get(stream->metadata, "filename", NULL, 0)->value;
       FILE *f = fopen(filename, "wb");
       fwrite(codecpar->extradata, 1, codecpar->extradata_size, f);
       fclose(f);
   }
   ```

29. **字幕流处理**
   - 文本字幕：直接解码后得到字符串
   - 图形字幕：需解码为位图（可能使用 `libass` 库渲染）

---

### **总结**
FFmpeg支持的流类型包括但不限于：
- 视频流（核心）
- 音频流（核心）
- 字幕流（常见于影视文件）
- 数据流（携带辅助信息）
- 附件流（嵌入文件）

实际开发中需要根据文件类型和处理需求，对不同类型的流进行针对性处理。