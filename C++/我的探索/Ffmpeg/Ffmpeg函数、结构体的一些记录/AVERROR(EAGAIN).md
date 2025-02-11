

`AVERROR(EAGAIN)` 这个语法结构涉及 FFmpeg 的错误处理机制和系统错误码的转换，我们来深入解析其语法本质：

**1. 组成结构解析**
```c
AVERROR(EAGAIN) 
```
这是典型的 C 语言宏嵌套用法，包含两个关键部分：

- **AVERROR**：FFmpeg 定义的错误码转换宏
- **EAGAIN**：系统级错误码（来自 POSIX 标准）

**2. 逐层拆解**

**第一层：AVERROR 宏**
```c
// FFmpeg 源码中的宏定义（libavutil/error.h）
#define AVERROR(e) (-(e)) 
```
作用：将正数错误码转换为负数，保持 FFmpeg 错误码的负数规范

**第二层：EAGAIN 常量**
```c
/* 来自系统头文件（如 errno.h）*/
#define EAGAIN   11  /* Linux 典型值 */
#define EWOULDBLOCK 11 /* 某些系统与 EAGAIN 同值 */
```
注意：具体数值因操作系统而异：
- Linux/macOS：通常为 11
- Windows：通过 WSAEAGAIN 映射

**3. 完整展开示例**
```c
AVERROR(EAGAIN) → -(11) → -11
```

**4. FFmpeg 错误码体系**

FFmpeg 错误码类型对照表：

| 错误类型            | 产生方式                 | 示例                   |
|---------------------|--------------------------|------------------------|
| 系统错误转换        | AVERROR(系统错误码)      | AVERROR(EAGAIN)        |
| FFmpeg 自定义错误   | 直接定义的负数常量        | AVERROR_EOF = -541478725 |
| 标准错误映射        | 特定宏定义                | AVERROR(EINVAL)        |

**5. 对比其他相似结构**

```c
// FFmpeg 自定义错误（通过宏生成唯一值）
#define AVERROR_EOF               FFERRTAG( 'E','O','F',' ')

// 系统错误转换
AVERROR(ENOMEM)  // 转换系统内存错误

// 直接数值定义
#define AVERROR_BUG               -558323010
```

**6. 语法特性分析**

这种设计实现了：
- **类型安全**：通过宏包装明确错误来源
- **平台兼容**：统一不同系统的错误码表现
- **错误分类**：
  - `AVERROR(EXXX)`：系统相关错误
  - `AVERROR_XXX`：多媒体处理特有错误

**7. 实际使用示例**

```c
int ret = avcodec_receive_frame(ctx, frame);

if (ret == AVERROR(EAGAIN)) { 
    // 需要输入新数据包
} else if (ret == AVERROR_EOF) {
    // 流结束处理
} else if (ret < 0) {
    // 其他错误处理
}
```

**8. 调试查看错误值**

可以通过打印验证：
```c
printf("EAGAIN 系统值: %d\n", EAGAIN);
printf("AVERROR(EAGAIN): %d\n", AVERROR(EAGAIN));
printf("AVERROR_EOF: %d\n", AVERROR_EOF);

/* 典型输出（Linux 环境）：
EAGAIN 系统值: 11
AVERROR(EAGAIN): -11 
AVERROR_EOF: -541478725 */
```

**9. 为什么需要这样设计？**

- **错误隔离**：将系统错误与多媒体错误区分开
- **统一接口**：所有函数返回 int 型错误码
- **可扩展性**：可添加自定义错误类型
- **跨平台**：同一代码在不同系统表现一致

**10. 常见陷阱**

1. **头文件缺失**：
   ```c
   // 必须包含
   #include <errno.h>      // 系统错误码
   #include <libavutil/error.h> // FFmpeg 错误宏
   ```

2. **错误类型混淆**：
   ```c
   // 错误！直接比较系统错误码
   if (ret == EAGAIN) // 应为 AVERROR(EAGAIN)
   ```

3. **平台差异**：
   ```c
   // Windows 需要特殊处理
   #ifdef _WIN32
   #define EAGAIN WSAEWOULDBLOCK
   #endif
   ```

**11. 扩展知识：FFERRTAG 宏**

FFmpeg 自定义错误码的生成方式：
```c
#define FFERRTAG(a, b, c, d) (-(int)MKTAG(a, b, c, d))
#define AVERROR_EOF FFERRTAG('E','O','F',' ')
```
这种设计使得每个错误码都有可读的 ASCII 标识。

理解这个语法结构，就能明白 FFmpeg 如何将系统错误与多媒体处理错误统一到同一错误处理体系中。这种设计模式在底层库开发中非常典型，既保持了与系统错误体系的兼容性，又扩展了领域特定的错误类型。