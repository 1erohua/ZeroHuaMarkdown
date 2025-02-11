

`av_frame_unref`和`av_packet_unref`是FFmpeg中用于管理引用计数和资源释放的关键函数。它们的核心区别和使用场景如下：

#### 一、核心作用对比
| 函数                | 作用                                                                 |
|---------------------|--------------------------------------------------------------------|
| `av_packet_unref`   | 释放数据包内部缓冲区的引用计数，当引用归零时自动释放内存。用于管理压缩数据包的资源。 |
| `av_frame_unref`    | 释放帧内部缓冲区的引用计数，当引用归零时自动释放内存。用于管理解码后的原始帧资源。 |

#### 二、使用场景详解
1. **`av_packet_unref`的典型场景**：
   - 每次调用`av_read_frame`后必须调用，否则会导致内存泄漏。
   - 在数据包处理完成后立即调用（如示例中的`main`循环）。
   - 示例代码中正确使用：
     ```cpp
     while (av_read_frame(...)) {
         // 处理数据包...
         av_packet_unref(pPacket); // ✔️ 必须手动释放
     }
     ```

2. **`av_frame_unref`的特殊性**：
   - **不需要手动调用的情况**：
     - **当通过`avcodec_receive_frame`获取帧时，FFmpeg内部会自动调用`av_frame_unref`**。
     - 示例代码中`decode_packet`函数未手动调用，但无内存泄漏：
       ```cpp
       while (avcodec_receive_frame(...) >= 0) {
           // FFmpeg内部自动unref之前的帧 ✔️
           // 直接使用pFrame数据...
       }
       ```
   - **需要手动调用的情况**：
     - 如果通过其他方式（如`av_frame_clone`）创建了帧的副本。
     - 需要主动放弃帧所有权时（如提前结束处理）。

#### 三、示例代码为何没有`av_frame_unref`
在提供的代码中：
1. `avcodec_receive_frame`内部机制：
   - 每次调用时会自动释放之前关联的帧数据。
   - 开发者只需复用同一个`AVFrame`结构体，无需手动清理。

2. 资源管理示意图：
   ```cpp
   AVFrame *pFrame = av_frame_alloc(); // 初始化
   while (avcodec_receive_frame(ctx, pFrame) == 0) {
       // 自动释放旧数据，填充新数据 ✔️
       save_frame(pFrame);
   }
   av_frame_free(&pFrame); // 最终释放结构体
   ```

#### 四、最佳实践总结
| 操作                     | 是否需要手动unref | 原因说明                                                                 |
|--------------------------|------------------|------------------------------------------------------------------------|
| 从`av_read_frame`获取包   | 是 ✅             | 必须手动释放，否则每个未释放的包会累计内存泄漏。                                |
| 从`avcodec_receive_frame`获取帧 | 否 ❌             | FFmpeg内部自动管理，手动调用会导致数据提前释放。                               |
| 克隆帧或包（如`av_packet_clone`） | 是 ✅             | 克隆操作会增加引用计数，必须通过unref释放所有权。                               |

#### 五、底层机制解析
FFmpeg使用引用计数（`refcount`）管理数据：
- **`av_packet_unref`**：将`AVPacket.buf`的引用减1，若归零则释放实际压缩数据。
- **`av_frame_unref`**：释放`AVFrame.data`中所有缓冲区的引用，重置帧元数据。

通过这种设计，开发者可以安全地在多个上下文（如多线程）中共享数据包或帧，而无需深拷贝。