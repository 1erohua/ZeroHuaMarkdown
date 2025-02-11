

以下是 FFmpeg 4.x 常用函数及参数的详细参考文档，按模块分类整理：

---

# 一、初始化与创建
## 1. `av_register_all()`
- **作用**：注册所有封装/解封装器和编解码器（FFmpeg 4.0+ 已废弃，自动调用）
- **参数**：无
- **返回值**：无
- **备注**：新版无需显式调用

## 2.`avformat_new_stream()`

```cpp
AVStream* avformat_new_stream(
		AVFormatContext *s,
		const struct AVCodec *c
)
```

`avformat_new_stream` 是 FFmpeg 库中用于创建新流的函数，常用于音视频封装（muxing）场景。以下是其参数、返回值及常用值的详细说明：

---
### **参数说明**
1. **`AVFormatContext *s`**  
   - **含义**：指向当前媒体容器上下文（如 MP4、MKV 文件）的指针，**用于关联新创建的流。**  
   - **常用值**：通过 `avformat_alloc_context()` 初始化的上下文，或已打开的封装器（如 `avformat_write_header` 前的输出上下文）。

2. **`const AVCodec *c`**  
   - **含义**：可选参数，指向编解码器的指针，用于初始化流的默认编解码参数（如编码类型、时间基）。  
   - **常用值**：  
     - **非编码场景（直接复用）**：传 `NULL`，后续手动设置 `AVStream->codecpar`。  
     - **编码场景**：通过 `avcodec_find_encoder()` 获取的编码器（如 `avcodec_find_encoder(AV_CODEC_ID_H264)`）。  
     - **解码场景**：通常不在此函数使用，因为解复用时会自动创建流。

---
### **返回值**
- **成功**：返回新创建的 `AVStream*` 指针，代表容器中的一个音/视频/字幕流。  
- **失败**：返回 `NULL`（如内存不足或参数无效）。
---
### **参数常用值与使用场景**
#### 1. **复用（Muxing）时创建流**
```c
AVFormatContext *fmt_ctx = avformat_alloc_context();
// 假设创建视频流
AVStream *video_stream = avformat_new_stream(fmt_ctx, NULL);
if (!video_stream) { /* 处理错误 */ }

// 手动设置编解码参数
AVCodecParameters *codecpar = video_stream->codecpar;
codecpar->codec_type = AVMEDIA_TYPE_VIDEO;
codecpar->codec_id = AV_CODEC_ID_H264;
codecpar->width = 1280;
codecpar->height = 720;
```

#### 2. **编码并复用（显式指定编码器）**
```c
AVCodec *encoder = avcodec_find_encoder(AV_CODEC_ID_AAC);
AVStream *audio_stream = avformat_new_stream(fmt_ctx, encoder);
audio_stream->time_base = (AVRational){1, encoder->supported_samplerates[0]};
```

#### 3. **关键字段初始化**
- **`codecpar`**：必须设置 `codec_type`、`codec_id` 及分辨率/采样率等。  
- **`time_base`**：流的时间基，影响帧时间戳计算。视频常用 `1/framerate`，音频常用 `1/sample_rate`。

---

### **注意事项**
1. **流索引**：新流的 `index` 自动分配，可通过 `stream->index` 获取。  
2. **默认值**：若传递编码器参数，函数会尝试初始化 `codecpar`，但通常仍需手动完善。  
3. **错误处理**：务必检查返回值，避免后续操作（如写文件头）因无效流崩溃。

---

通过合理使用 `avformat_new_stream`，可以灵活控制音视频流的创建与参数配置，适应封装、转码等多样化需求。









---

# 二、封装格式处理（AVFormatContext）
## 1. `avformat_open_input()`
```c
int avformat_open_input(
    AVFormatContext **ps, 
    const char *url, 
    AVInputFormat *fmt, 
    AVDictionary **options
);
```
- **参数**：
  - `ps`：输出格式上下文指针的地址
  - `url`：输入文件路径或网络流地址（常用值：`"input.mp4"`）
  - `fmt`：强制指定输入格式（通常为 `NULL` 自动检测）
  - `options`：附加参数（如设置超时 `timeout=5000000`）
- **返回值**：`0` 成功，负数错误码（如 `AVERROR(EINVAL)` 无效参数）
- **典型流程**：打开输入文件并读取头部信息

**作用**

- **打开输入源**：根据 `url`（文件路径、网络流地址等）打开媒体输入源。
- **解析格式头**：读取输入源的头部数据，探测其封装格式（如 MP4、FLV、RTMP 等）。
- **填充初步信息**：将格式信息绑定到 `AVFormatContext`，并初始化部分流信息。

**对 `AVFormatContext` 的修改**

| 字段/属性                     | 修改内容                                              |
| ------------------------- | ------------------------------------------------- |
| `iformat`                 | 设置为探测到的输入格式（如 `AVInputFormat` 的 `ff_mp4_demuxer`） |
| `url`                     | 赋值为输入的 `url` 参数                                   |
| `streams`                 | 分配流数组（`nb_streams` 可能 >0，但流参数未完全解析）               |
| `start_time` / `duration` | 可能根据文件头信息填充（若格式支持）                                |
| `bit_rate`                | 可能根据文件头或初始数据估算                                    |
| `metadata`                | 填充文件头中的元数据（如 MP4 的 `title`、`author`）              |
| `pb` (AVIOContext)        | 绑定到输入源的 I/O 上下文（如文件句柄、网络连接）                       |
| `flags`                   | 可能更新（如设置 `AVFMT_FLAG_CUSTOM_IO` 若使用自定义 I/O）       |
在`avformat_open_input`完成后：

- **被选中的解复用器**存储在：
```cpp
    AVFormatContext->iformat  // 类型是AVInputFormat指针
```
 



---

## 2. `avformat_find_stream_info()`
```c
int avformat_find_stream_info(
    AVFormatContext *ic, 
    AVDictionary **options
);
```
- **作用**：探测流信息
- **参数**：
  - `ic`：已打开的格式上下文
  - `options`：流探测参数（如限制探测时长 `probesize=4096`）
- **返回值**：`>=0` 成功，负数错误码

> **对 `AVFormatContext` 的修改**
> 
> 1. **填充流信息 (`streams` 字段)**
> 
> - **`ic->nb_streams`**：明确流的数量（例如视频、音频、字幕流的数量）。
> - **`ic->streams[i]`**：为每个流（`AVStream`）填充详细信息：
>     - **`codecpar`**：编解码器参数（如编码类型、分辨率、采样率、比特率等）。
>     - **`time_base`**：时间基（时间戳单位，如 `1/1000`）。
>     - **`duration`**：流的时长（基于时间基）。
>     - **`start_time`**：流的起始时间戳。
>     - **`metadata`**：流的元数据（如语言、标题等）。
> 
> 2. **更新全局信息**
> 
> - **`ic->duration`**：媒体文件的总时长（取所有流的最大值）。
> - **`ic->bit_rate`**：全局比特率（若文件未明确指定，则通过数据估算）。
> - **`ic->iformat`**：若输入格式未明确指定（如通过 `avformat_open_input` 自动检测），此函数会进一步确认格式。
> 
> 3. **解码器参数探测**
> 
> - 对于某些格式（如 MPEG-TS），函数会**尝试解码部分数据包**以获取精确的编解码参数（如 H.264 的 SPS/PPS）。
> 
> 4. **时间戳修正**
> 
> - 若文件时间戳不完整或不规范（如某些 MP4 文件），函数会尝试**重建合理的时序关系**。
> 

## 3.`avformat_alloc_output_context()`

`avformat_alloc_output_context2()` 是 FFmpeg 中用于初始化输出媒体容器格式（如 MP4、FLV、TS 等）的核心函数。它在封装（Muxing）过程中用于创建输出文件的格式上下文（`AVFormatContext`）。

---

### 函数原型
```c
int avformat_alloc_output_context2(
    AVFormatContext **ctx, 
    AVOutputFormat *oformat, 
    const char *format_name, 
    const char *filename
);
```

---

### 参数详解

#### 1. `AVFormatContext **ctx` (输出参数)
- **含义**：用于接收新创建的 `AVFormatContext` 指针的指针。
- **说明**：
  - 调用前需初始化为 `NULL`。
  - 函数成功后会分配内存，并通过此参数返回上下文对象。
  - 使用后需手动释放（通过 `avformat_free_context`）。

#### 2. `AVOutputFormat *oformat` (输入参数)
- **含义**：显式指定输出格式（如 MP4、FLV）。若为 `NULL`，函数会根据其他参数自动推断格式。
- **常用值**：
  - `NULL`：让函数自动选择格式（根据 `format_name` 或 `filename` 后缀）。
  - 手动指定：通过 `av_guess_format()` 获取格式，例如：
    ```c
    AVOutputFormat *fmt = av_guess_format("mp4", NULL, NULL);
    ```

#### 3. `const char *format_name` (输入参数)
- **含义**：通过字符串名称指定输出格式（如 `"mp4"`、`"flv"`）。
- **常用值**：
  - `NULL`：不强制指定格式名称。
  - 明确格式名：如 `"hls"`（HLS 流）、`"mpegts"`（MPEG-TS 流）。

#### 4. `const char *filename` (输入参数)
- **含义**：输出文件名或 URL（如 `"output.mp4"`、`"rtmp://example.com/live/stream"`）。
- **常用值**：
  - 本地文件路径：如 `"video.mp4"`。
  - 流媒体地址：如 `"udp://192.168.1.1:1234"`。
  - `NULL`：若不需要通过文件名推断格式（需显式指定 `format_name` 或 `oformat`）。

---

### 返回值
- **成功**：返回 `0`，并通过 `ctx` 返回有效的 `AVFormatContext`。
- **失败**：返回负值错误码（如 `AVERROR(ENOMEM)` 内存不足）。

---

### 参数优先级
4. 若 `oformat` 非空：直接使用它指定的格式。
5. 若 `format_name` 非空：根据名称查找匹配的格式。
6. 若 `filename` 非空：根据文件扩展名推断格式。

---

### 典型用法示例

#### 示例 1：通过文件名自动推断格式
```c
AVFormatContext *fmt_ctx = NULL;
int ret = avformat_alloc_output_context2(&fmt_ctx, NULL, NULL, "output.mp4");
if (ret < 0) {
    // 错误处理
}
```

#### 示例 2：显式指定格式名称
```c
AVFormatContext *fmt_ctx = NULL;
int ret = avformat_alloc_output_context2(&fmt_ctx, NULL, "flv", NULL);
if (ret < 0) {
    // 错误处理
}
```

#### 示例 3：使用已有的 AVOutputFormat
```c
AVOutputFormat *fmt = av_guess_format("mpegts", NULL, NULL);
AVFormatContext *fmt_ctx = NULL;
int ret = avformat_alloc_output_context2(&fmt_ctx, fmt, NULL, NULL);
```

---

### 注意事项
7. **内存管理**：成功调用后，必须通过 `avformat_free_context(fmt_ctx)` 释放上下文。
8. **错误处理**：始终检查返回值，失败时可通过 `av_err2str(ret)` 获取错误描述。
9. **流创建**：创建上下文后，需手动添加流（`avformat_new_stream`）并设置编码参数。

---
### 常见使用场景
- 封装本地文件（MP4/MKV/FLV）。
- 推流到 RTMP/RTSP 服务器。
- 生成 HLS 分片或 DASH 格式。

通过合理使用此函数，可以灵活控制输出格式，适应不同的封装需求。









---

# 三、编解码器处理（AVCodecContext）
## 1. `avcodec_find_decoder()`
```c
AVCodec *avcodec_find_decoder(enum AVCodecID id);
```
- **参数**：
  - `id`：编解码器ID（常用值：`AV_CODEC_ID_H264`, `AV_CODEC_ID_AAC`）
- **返回值**：编解码器指针，`NULL` 表示未找到

---

## 2. `avcodec_open2()`
```c
int avcodec_open2(
    AVCodecContext *avctx, 
    const AVCodec *codec, 
    AVDictionary **options
);
```
- **参数**：
  - `avctx`：已初始化的编解码器上下文
  - `codec`：通过 `avcodec_find_decoder` 获取的编解码器
  - `options`：编码/解码参数（如设置 `preset`）
- **返回值**：`0` 成功，负数错误码

---






以下是关于三个FFmpeg函数的详细说明：

---

## 3. `avcodec_parameters_copy`
**函数原型**  
`int avcodec_parameters_copy(AVCodecParameters *dst, const AVCodecParameters *src);`

**作用**  
将源 `AVCodecParameters` 的内容深拷贝到目标 `AVCodecParameters` 中，用于复制编解码器参数（如视频分辨率、音频采样率等）。

**参数**  
- `dst`：目标参数结构体，需已通过 `avcodec_parameters_alloc()` 分配内存或来自 `AVStream->codecpar`。
- `src`：源参数结构体，必须非空。

**返回值**  
- 成功返回 `0`。
- 失败返回负错误码（如 `AVERROR(ENOMEM)` 表示内存不足）。

**使用场景**  
- 复制流的编解码参数（如从输入流拷贝到输出流）。
- 示例：  
  ```c
  AVStream *in_stream = ...;
  AVStream *out_stream = avformat_new_stream(output_ctx, NULL);
  avcodec_parameters_copy(out_stream->codecpar, in_stream->codecpar);
  ```

---

## 4.`avcodec_parameters_from_context`
**函数原型**  
`int avcodec_parameters_from_context(AVCodecParameters *par, const AVCodecContext *codec);`

**作用**  
从 `AVCodecContext` 中提取编解码参数并填充到 `AVCodecParameters`，用于将编码器/解码器上下文的参数传递给流。

**参数**  
- `par`：目标参数结构体，需已分配内存。
- `codec`：源编解码器上下文，必须非空且已初始化。

**返回值**  
- 成功返回 `0`。
- 失败返回负错误码（如参数不兼容或内存错误）。

**使用场景**  
- 初始化输出流的 `codecpar`（如复用视频时）：  
  ```c
  AVCodecContext *encoder_ctx = ...;  // 已配置的编码器上下文
  AVStream *stream = avformat_new_stream(muxer_ctx, NULL);
  avcodec_parameters_from_context(stream->codecpar, encoder_ctx);
  ```

---

## 5.`avcodec_parameters_to_context`
**函数原型**  
`int avcodec_parameters_to_context(AVCodecContext *codec, const AVCodecParameters *par);`

**作用**  
将 `AVCodecParameters` 中的参数复制到 `AVCodecContext`，用于根据流参数初始化解码器/编码器上下文。

**参数**  
- `codec`：目标编解码器上下文，需已通过 `avcodec_alloc_context3()` 分配。
- `par`：源参数结构体，必须非空。

**返回值**  
- 成功返回 `0`。
- 失败返回负错误码（如参数不支持）。

**使用场景**  
- 初始化解码器上下文：  
  ```c
  AVStream *stream = ...;       // 输入流的视频/音频流
  AVCodecContext *decoder_ctx = avcodec_alloc_context3(codec);
  avcodec_parameters_to_context(decoder_ctx, stream->codecpar);
  avcodec_open2(decoder_ctx, codec, NULL);
  ```

---

### 关键区别与注意事项
1. **参数方向**：
   - `avcodec_parameters_copy`：`AVCodecParameters` → `AVCodecParameters`。
   - `avcodec_parameters_from_context`：`AVCodecContext` → `AVCodecParameters`。
   - `avcodec_parameters_to_context`：`AVCodecParameters` → `AVCodecContext`。

2. **内存管理**：
   - `AVCodecParameters` 需手动分配/释放（`avcodec_parameters_alloc()`/`avcodec_parameters_free()`）。
   - `AVCodecContext` 需通过 `avcodec_alloc_context3()` 创建，`avcodec_free_context()` 释放。

3. **不完整复制**：
   - `AVCodecParameters` 仅包含基础参数（如宽高、码率），不包含动态信息（如帧数据）。
   - `AVCodecContext` 包含更多运行时配置（如 `time_base`、`pix_fmt`），需额外设置。

---

### 常见错误码
| 错误码              | 描述                  |
|---------------------|----------------------|
| `AVERROR(EINVAL)`   | 无效参数（如 NULL 输入） |
| `AVERROR(ENOMEM)`   | 内存不足              |
| `AVERROR(ENOSYS)`   | 功能未实现（如不支持的编解码器） |

---

### 总结
- **复用流时**：用 `avcodec_parameters_copy` 或 `avcodec_parameters_from_context` 设置输出流的 `codecpar`。
- **初始化解码器时**：用 `avcodec_parameters_to_context` 从流参数初始化上下文。
- 始终检查返回值，确保参数传递成功。












# 四、数据包与帧处理
## 1. `av_read_frame()` 
```c
int av_read_frame(
    AVFormatContext *s, 
    AVPacket *pkt
);
```
- **作用**：读取原始数据包
- **参数**：
  - `s`：格式上下文
  - `pkt`：输出的数据包指针
- **返回值**：`0` 成功，`AVERROR_EOF` 文件结束，其他负数错误码

---

## 2. `avcodec_send_packet()`
```c
int avcodec_send_packet(
    AVCodecContext *avctx, 
    const AVPacket *avpkt
);
```
- **参数**：
  - `avctx`：编解码器上下文
  - `avpkt`：输入数据包（设为 `NULL` 会触发刷新）
- **返回值**：`0` 成功，`AVERROR(EAGAIN)` 需要先接收帧

---

## 3. `avcodec_receive_frame()`
```c
int avcodec_receive_frame(
    AVCodecContext *avctx, 
    AVFrame *frame
);
```
- **参数**：
  - `avctx`：编解码器上下文
  - `frame`：输出的解码后帧
- **返回值**：`0` 成功，`AVERROR(EAGAIN)` 需要发送更多包，`AVERROR_EOF` 结束

---

## 4. `avcodec_send_frame()`
### **函数原型**
```C
int avcodec_send_frame(AVCodecContext *avctx, const AVFrame *frame);
```
---
### **参数说明**

4. **`AVCodecContext *avctx`**
    
    - **含义**：编码器上下文，包含编码器的配置信息（如码率、分辨率、编码格式等）。
    - **常用值**：通过 `avcodec_alloc_context3` 分配，并通过 `avcodec_open2` 初始化的编码器上下文。
5. **`const AVFrame *frame`**
    
    - **含义**：待编码的原始帧数据。
    - **常用值**：
        - **正常帧**：填充了视频像素数据（如YUV）或音频样本（如PCM）的 `AVFrame`。
        - **结束标记**：传入 `NULL`，表示刷新编码器（处理剩余数据并结束编码）。

### **关键区别总结**

| 函数                      | 所属模块        | 用途                | 数据类型        |
| ----------------------- | ----------- | ----------------- | ----------- |
| `avcodec_send_packet`   | libavcodec  | 向解码器发送压缩数据包       | `AVPacket*` |
| `avcodec_send_frame`    | libavcodec  | 向编码器发送原始帧         | `AVFrame*`  |
| `avcodec_receive_frame` | libavcodec  | 从解码器获取解码后的帧       | `AVFrame*`  |
| `av_read_frame`         | libavformat | 从媒体文件读取压缩数据包（不解码） | `AVPacket*` |
### **解码流程**

6. `av_read_frame()` → 从文件读取`AVPacket`。
7. `avcodec_send_packet()` → 发送`AVPacket`到解码器。
8. `avcodec_receive_frame()` → 从解码器获取`AVFrame`。

### **编码流程**

9. 构造`AVFrame`（如从摄像头或图像生成）。
10. `avcodec_send_frame()` → 发送`AVFrame`到编码器。
11. `avcodec_receive_packet()` → 从编码器获取`AVPacket`。
12. 写入输出文件（如MP4）。

# 五、工具函数
## 1. `av_log_set_level()`
```c
void av_log_set_level(int level);
```
- **参数**：
  - `level`：日志级别（常用值：`AV_LOG_DEBUG`, `AV_LOG_WARNING`, `AV_LOG_ERROR`）

---

## 2. `av_strerror()`
```c
int av_strerror(int errnum, char *errbuf, size_t errbuf_size);
```
- **作用**：将错误码转为可读信息
- **参数**：
  - `errnum`：错误码（如函数返回的负值）
  - `errbuf`：输出缓冲区
  - `errbuf_size`：缓冲区大小
- **返回值**：`0` 成功，负数错误码



## 3.`av_log_set_callback` 函数详解

### 1. 函数原型
```c
void av_log_set_callback(void (*callback)(void*, int, const char*, va_list));
```

### 2. 核心作用
设置自定义日志回调函数，接管 FFmpeg 的日志输出（默认输出到 `stderr`）
### ▶︎ 参数解析
| 参数        | 类型               | 说明                                                                 |
|-------------|--------------------|----------------------------------------------------------------------|
| `callback`  | `void (*)(...)`    | 自定义回调函数指针，原型需严格匹配                                   |
| 回调参数1   | `void*`            | 用户自定义指针（需配合 `av_log_set_context` 使用，通常设为 `NULL`） |
| 回调参数2   | `int`              | 日志级别（如 `AV_LOG_INFO`）                                         |
| 回调参数3   | `const char*`      | 格式化字符串（类似 `printf`）                                        |
| 回调参数4   | `va_list`          | 可变参数列表                                                         |

### ▶︎ 常用日志级别
| 宏定义              | 值   | 说明                     |
|---------------------|------|--------------------------|
| `AV_LOG_PANIC`      | 0    | 致命错误（程序可能崩溃） |
| `AV_LOG_FATAL`      | 8    | 严重错误                 |
| `AV_LOG_ERROR`      | 16   | 普通错误                 |
| `AV_LOG_WARNING`    | 24   | 警告信息                 |
| `AV_LOG_INFO`       | 32   | 状态信息（默认级别）     |
| `AV_LOG


---

# 六、资源释放
## 1. `avformat_close_input()`
```c
void avformat_close_input(AVFormatContext **s);
```
- **作用**：关闭输入流并释放资源

---

## 2. `av_packet_unref()`
```c
void av_packet_unref(AVPacket *pkt);
```
- **作用**：释放数据包引用

---

### 错误码速查表
| 错误码              | 值          | 含义                     |
|---------------------|-------------|--------------------------|
| `AVERROR(EAGAIN)`   | -11         | 需要重新尝试操作          |
| `AVERROR_EOF`       | -541478725  | 文件/流结束              |
| `AVERROR(EINVAL)`   | -22         | 无效参数                 |
| `AVERROR(ENOMEM)`   | -12         | 内存不足                 |

---

### 使用示例片段
```c
AVFormatContext *fmt_ctx = NULL;
if (avformat_open_input(&fmt_ctx, "input.mp4", NULL, NULL) < 0) {
    char errbuf[128];
    av_strerror(ret, errbuf, sizeof(errbuf));
    fprintf(stderr, "Error: %s\n", errbuf);
    exit(1);
}
```

---

此文档可作为快速参考，实际使用时需结合官方文档和具体场景调整参数。建议通过 `av_err2str()` 宏（需要C99）快速获取错误描述。


## avformat_free_context与avformat_close_input的区别


`avformat_free_context(input_fmt_ctx)` 和 `avformat_close_input(&input_fmt_ctx)` 在 FFmpeg 中有不同的职责和使用场景，具体区别如下：

---

### **1. `avformat_close_input(&input_fmt_ctx)`**
- **作用**  
  关闭由 `avformat_open_input` 打开的输入流，并释放所有相关资源，包括：
  - 内部缓冲区（如流信息、编解码器上下文等）。
  - 文件句柄（如已打开的文件或网络连接）。
  - `AVFormatContext` 结构体本身。

- **使用场景**  
  当通过 `avformat_open_input` 成功打开输入流后，清理时应调用此函数。它会自动处理所有资源的释放，并将指针置为 `NULL`，避免悬空指针。

- **示例**  
  ```c
  AVFormatContext *fmt_ctx = NULL;
  avformat_open_input(&fmt_ctx, "input.mp4", NULL, NULL); // 打开输入
  // ... 处理数据 ...
  avformat_close_input(&fmt_ctx); // 关闭输入并释放所有资源，fmt_ctx 变为 NULL
  ```

---

### **2. `avformat_free_context(input_fmt_ctx)`**
- **作用**  
  直接释放手动分配的 `AVFormatContext` 结构体内存，但**不会关闭输入流或释放内部资源**（如流信息、编解码器上下文等）。

- **使用场景**  
  当手动分配了 `AVFormatContext`（例如通过 `avformat_alloc_context`），但未成功调用 `avformat_open_input`，或需要提前释放结构体时使用。

- **示例**  
  ```c
  AVFormatContext *fmt_ctx = avformat_alloc_context(); // 手动分配
  // ... 未调用 avformat_open_input 或调用失败 ...
  avformat_free_context(fmt_ctx); // 直接释放结构体
  ```

---

### **关键区别**
| 函数                        | 释放内容                     | 自动置空指针 | 适用场景                          |
|----------------------------|----------------------------|-------------|----------------------------------|
| `avformat_close_input`     | 输入流、内部资源、结构体本身 | 是          | 成功调用 `avformat_open_input` 后 |
| `avformat_free_context`    | 仅结构体内存                | 否          | 手动分配但未成功打开输入流时       |

---

### **注意事项**
- **不要混合使用**  
  若已调用 `avformat_close_input`，**无需再调用** `avformat_free_context`，否则会导致双重释放（Double Free）引发崩溃。
  
- **错误处理**  
  如果 `avformat_open_input` 失败，需手动调用 `avformat_free_context` 释放已分配的结构体：
  ```c
  AVFormatContext *fmt_ctx = NULL;
  if (avformat_open_input(&fmt_ctx, "input.mp4", NULL, NULL) < 0) {
      avformat_free_context(fmt_ctx); // 打开失败时手动释放
  }
  ```

---

### **总结**
- 使用 `avformat_close_input` 关闭由 `avformat_open_input` 打开的输入流。
- 使用 `avformat_free_context` 释放手动分配但未关联实际输入流的 `AVFormatContext`。




# 七、内存管理


`av_calloc` 和 `av_free` 是 FFmpeg 的 `libavutil` 库中用于内存管理的函数，专为音视频数据处理设计。以下是它们的详细说明：

---

### **1. `av_calloc` 函数**
#### **函数原型**：
```c
void *av_calloc(size_t nmemb, size_t size);
```

#### **参数**：
- **`nmemb`**：要分配的元素数量。
- **`size`**：每个元素的大小（字节）。

#### **返回值**：
- **成功**：返回指向分配内存的指针，内存内容初始化为零。
- **失败**：返回 `NULL`（如内存不足）。

#### **参数常用值**：
- **分配缓冲区数组**：
  - 音频处理：`nmemb` 可能是音频样本数，`size` 为每个样本的大小（例如 2 字节的 16 位 PCM）。
  - 视频处理：`nmemb` 可能是帧的宽 × 高，`size` 为像素格式大小（如 YUV420P 每个像素占 1 字节）。
- **分配结构体数组**：
  - 例如分配 `AVPacket` 或 `AVFrame` 结构体数组，`nmemb` 为数量，`size` 为 `sizeof(AVFrame)`。

#### **特点**：
- 内存对齐：分配的内存可能按 FFmpeg 的默认对齐（如 16/32 字节），提升 SIMD 指令效率。
- 安全性：自动置零内存，避免未初始化数据的问题。

---

### **2. `av_free` 函数**
#### **函数原型**：
```c
void av_free(void *ptr);
```

#### **参数**：
- **`ptr`**：指向需释放内存的指针（必须由 `av_malloc`/`av_calloc` 分配）。

#### **返回值**：
- 无（`void` 类型）。

#### **参数常用值**：
- 释放音视频数据结构：
  - 如 `av_free(frame->data[0])` 释放 `AVFrame` 的视频数据。
  - 释放 `AVPacket` 后调用 `av_free_packet`（内部可能调用 `av_free`）。

#### **注意事项**：
- **不可混用**：只能用 `av_free` 释放 FFmpeg 分配的内存，与标准 `free()` 混用会导致崩溃。
- **安全释放**：若 `ptr` 为 `NULL`，函数无操作（类似标准 `free`）。

---

### **使用示例**
```c
// 分配 100 个 AVFrame 结构体的内存
size_t count = 100;
AVFrame **frames = av_calloc(count, sizeof(AVFrame *));
if (!frames) {
    // 处理内存不足错误
}

// 使用后释放
for (size_t i = 0; i < count; i++) {
    av_free(frames[i]);
}
av_free(frames);
```

---

### **与标准函数的区别**
| 特性                | FFmpeg 函数          | 标准 C 函数       |
|---------------------|---------------------|------------------|
| **内存对齐**         | 按平台优化对齐       | 无特殊对齐       |
| **错误处理**         | 返回 `NULL` 明确失败 | 同左，但无额外处理 |
| **多线程安全**       | 是                  | 依赖实现         |
| **释放安全性**       | 必须配对使用         | 需与 `malloc`/`calloc` 配对 |

---

### **总结**
- **`av_calloc`**：用于分配对齐且初始化的内存，适合音视频大数据块。
- **`av_free`**：专用于释放 FFmpeg 分配的内存，确保内存管理一致性。
- **典型场景**：处理 `AVFrame`、`AVPacket`、音视频缓冲区时高频使用。











# 遗漏的
## 1. `avformat_alloc_context()`

**所属库**: `libavformat`  
**作用**: 分配并初始化一个 `AVFormatContext` 结构体，用于管理媒体文件的格式上下文（如输入/输出容器格式、流信息等）。


### 参数

- **无参数**：直接调用，无需传入参数。

### 返回值

- **成功**: 返回指向 `AVFormatContext` 的指针。
- **失败**: 返回 `NULL`（内存不足时）。

### 常用场景

- 在解复用（读取媒体文件）或复用（写入媒体文件）前初始化上下文。
- 示例：
```cpp
	AVFormatContext *fmt_ctx = avformat_alloc_context(); 
	if (!fmt_ctx) {
	     // 处理分配失败 
    }
```

    

### 注意事项


- **输入文件无需显示调用**：`avformat_open_input()` 函数在打开输入文件时，**会自动分配输入格式上下文**（`AVFormatContext`）输入文件的格式（如 MP4、MKV）是已知的，`avformat_open_input()` 会通过文件内容自动探测格式并填充 `AVFormatContext`。显式分配反而可能冗余。

- **输出文件必须使用**：输出文件的格式需要显式指定（如 MP4、FLV），因此必须通过 `avformat_alloc_output_context2()` 手动分配上下文，并明确设置输出格式参数。

- **释放资源**: 必须使用 `avformat_free_context(fmt_ctx)` 释放，不可用 `av_free`。
- **后续初始化**: 通常需要调用 `avformat_open_input()` 打开具体文件并填充上下文信息。

**作用**

- **分配内存**：创建一个新的 `AVFormatContext` 结构体实例，并初始化其默认字段。
- **设置默认值**：为结构体中的部分字段赋予初始值（例如 `flags`、`io_open` 回调等）。

**它对 `AVFormatContext` 的修改**

| 字段/属性                   | 初始值/行为                                 |
| ----------------------- | -------------------------------------- |
| `iformat` / `oformat`   | 初始化为 `NULL`（未指定输入/输出格式）                |
| `nb_streams`            | 0（无流信息）                                |
| `streams`               | `NULL`（未分配流数组）                         |
| `url`                   | `NULL`（未设置输入/输出 URL）                   |
| `duration` / `bit_rate` | 0（未探测时长或比特率）                           |
| `metadata`              | 空字典（无元数据）                              |
| `flags`                 | 初始化为 `AVFMT_FLAG_*` 的默认组合（如自动生成 PTS 等） |
| `io_open` / `io_close`  | 设置默认的 I/O 回调函数（若未自定义）                  |
| `pb` (AVIOContext)      | `NULL`（未绑定自定义 I/O 上下文）                 |




## 2. `av_malloc()`

**所属库**: `libavutil`  
**作用**: 动态分配内存块，功能类似标准库的 `malloc`，但提供内存对齐优化（如32字节对齐），适合FFmpeg内部数据处理。

### 参数

- **`size`**（`size_t` 类型）: 要分配的字节数。

### 返回值

- **成功**: 返回指向分配内存的 `void*` 指针。
- **失败**: 返回 `NULL`（内存不足时）。

### 常用参数值

- 分配结构体实例：`av_malloc(sizeof(AVPacket))`。
- 分配缓冲区：根据数据需求计算大小，例如：
    - 视频帧缓冲区：`width * height * 3/2`（YUV420P格式）。
    - 音频缓冲区：`nb_samples * channels * av_get_bytes_per_sample(sample_fmt)`。

### 示例

C

`// 分配一个AVPacket结构体 AVPacket *pkt = av_malloc(sizeof(AVPacket)); if (!pkt) {     // 处理分配失败 } // 分配一个音频缓冲区（假设参数已定义） int buffer_size = nb_samples * channels * av_get_bytes_per_sample(AV_SAMPLE_FMT_FLT); uint8_t *audio_buffer = av_malloc(buffer_size);`

### 注意事项

- **释放资源**: 必须使用 `av_free(ptr)`，不可混用标准库的 `free()`。
- **内存对齐**: 分配的内存已对齐，适合SIMD指令优化。
- **清零内存**: 若需要初始化为0，使用 `av_mallocz()`。



## 3.av_strerror()


在 FFmpeg 4 中，`av_strerror()` 是一个用于将错误码转换为可读字符串的关键函数，常用于调试和错误处理。

---

### **函数原型**
```c
int av_strerror(int errnum, char *errbuf, size_t errbuf_size);
```

---

### **参数详解**
13. **`errnum`**  
   - **含义**：FFmpeg 返回的错误码（通常为负数）。  
   - **常见值**：  
     - `AVERROR(EAGAIN)`：资源暂时不可用（需重试）。  
     - `AVERROR(ENOMEM)`：内存不足。  
     - `AVERROR(EINVAL)`：无效参数或格式。  
     - `AVERROR_EOF`：文件结束（End Of File）。  
     - `AVERROR_BUG`：内部逻辑错误。  
     - `AVERROR_UNKNOWN`：未知错误。  
     - 其他系统错误码（如 `ENOENT` 文件不存在）。  

14. **`errbuf`**  
   - **含义**：存储错误信息的字符缓冲区。  
   - **常用值**：用户定义的 `char` 数组，例如 `char errbuf[256];`。

15. **`errbuf_size`**  
   - **含义**：缓冲区的最大容量（需足够大以容纳错误信息）。  
   - **推荐值**：使用宏 `AV_ERROR_MAX_STRING_SIZE`（通常定义为 64）。

---

### **返回值**
- **成功**：返回 `0`，错误信息写入 `errbuf`。  
- **失败**：返回负数，表示无法转换错误码（如缓冲区过小或无效错误码）。

---

### **使用示例**
```c
int ret = avformat_open_input(&format_ctx, filename, NULL, NULL);
if (ret < 0) {
    char errbuf[AV_ERROR_MAX_STRING_SIZE];
    av_strerror(ret, errbuf, sizeof(errbuf));
    fprintf(stderr, "打开输入失败: %s\n", errbuf);
    return;
}
```

---

### **注意事项**
16. **错误码处理**：  
   FFmpeg 错误码通常为负数。若错误来自系统调用（如 `open()`），需用 `AVUNERROR(errnum)` 转换正数错误码后再调用 `av_strerror()`。

17. **缓冲区安全**：  
   确保 `errbuf_size` ≥ `AV_ERROR_MAX_STRING_SIZE`，避免截断。

---

### **典型场景**
- 解码/编码失败时获取具体原因。  
- 文件打开/读取错误时输出友好提示。  
- 网络流超时或中断的日志记录。

通过 `av_strerror()`，开发者可以快速定位问题，提升调试效率。