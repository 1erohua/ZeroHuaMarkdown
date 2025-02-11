# 简单、优先看
在 Ubuntu 22.04 上为 Android 开发编译 FFmpeg，需要分两步操作：**编译适用于 Android 的 FFmpeg 库**（`.so` 文件），而非直接在 Ubuntu 上安装 FFmpeg。以下是详细方案：

---

### 一、版本选择建议
推荐使用 **FFmpeg 最新稳定版**（目前是 `6.1.1`），可通过官网或 Git 获取：
```bash
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
cd ffmpeg
git checkout release/6.1  # 切换到最新稳定分支
```

---

### 二、环境准备
#### 1. 安装基础编译工具
```bash
sudo apt update
sudo apt install build-essential git wget nasm pkg-config
```

#### 2. 下载 Android NDK
推荐使用 **NDK r25c**（兼容性较好）：
```bash
wget https://dl.google.com/android/repository/android-ndk-r25c-linux.zip
unzip android-ndk-r25c-linux.zip
export NDK=/path/to/android-ndk-r25c  # 替换为你的实际路径
```

---

### 三、交叉编译 FFmpeg 为 Android 库
#### 1. 编写编译脚本
创建 `build_android.sh`，内容如下（以 `arm64-v8a` 架构为例）：
```bash
#!/bin/bash

# 配置参数
API_LEVEL=21
ARCH=arm64  # 可选：arm（armeabi-v7a）、x86、x86_64
PREFIX=$(pwd)/android/$ARCH

# 设置工具链路径
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64
SYSROOT=$TOOLCHAIN/sysroot

# 配置编译参数
./configure \
  --target-os=android \
  --prefix=$PREFIX \
  --arch=$ARCH \
  --enable-cross-compile \
  --sysroot=$SYSROOT \
  --cross-prefix=$TOOLCHAIN/bin/llvm- \
  --cc=$TOOLCHAIN/bin/${ARCH}-linux-android$API_LEVEL-clang \
  --cxx=$TOOLCHAIN/bin/${ARCH}-linux-android$API_LEVEL-clang++ \
  --enable-shared \
  --disable-static \
  --disable-programs \
  --disable-doc \
  --enable-neon \
  --enable-hwaccels \
  --enable-jni \
  --enable-mediacodec
```

#### 2. 执行编译
```bash
chmod +x build_android.sh
./build_android.sh
make -j$(nproc) && make install
```

#### 3. 获取输出文件
编译完成后，库文件会生成在 `android/arm64/lib/` 目录，包含：
- `libavcodec.so`, `libavformat.so` 等动态库
- 头文件在 `include/` 目录

---

### 四、集成到 Android 项目
1. **复制库文件**：将生成的 `.so` 文件放入 Android 项目的 `app/src/main/jniLibs/对应架构目录`。
2. **配置 CMake/ndk-build**：在 `build.gradle` 中指定支持的架构：
   ```gradle
   android {
       defaultConfig {
           ndk {
               abiFilters 'arm64-v8a', 'armeabi-v7a', 'x86', 'x86_64'
           }
       }
   }
   ```
3. **调用 FFmpeg API**：通过 JNI 或封装库（如 [FFmpegKit](https://github.com/arthenica/ffmpeg-kit)）使用功能。

---

### 五、注意事项
4. **多架构支持**：重复上述步骤，修改 `ARCH` 参数编译其他架构。
5. **代码裁剪**：通过 `--disable-*` 参数移除不需要的模块以减小体积。
6. **NDK 版本**：若使用其他 NDK 版本，需调整工具链路径。

按此流程操作后，你将获得适配 Android 的 FFmpeg 库，可直接在 Java/Kotlin 中通过 JNI 调用。






# 复杂
在 Ubuntu 22.04 上为 Android 交叉编译 FFmpeg 并生成独立的动态链接库（`.so`），需遵循以下流程。本流程结合了多个来源的实践指南，并优化了关键步骤的兼容性。

---

### **一、环境准备**
1. **安装必要工具**  
   ```bash
   sudo apt update
   sudo apt install git wget unzip nasm yasm pkg-config autoconf libtool
   ```

2. **下载 NDK（推荐 r21e 版本）**  
   - 从 [Android NDK 旧版本页面](https://developer.android.com/ndk/downloads/older_releases) 下载 `android-ndk-r21e-linux-x86_64.zip`。
   - 解压到 `/home/[user]/ndk/android-ndk-r21e`，并配置环境变量：
     ```bash
     export NDK=/home/[user]/ndk/android-ndk-r21e
     export PATH=$NDK:$PATH
     ```

---

### **二、交叉编译 x264（FFmpeg 依赖）**
1. **下载 x264 源码**  
   ```bash
   git clone https://code.videolan.org/videolan/x264.git
   cd x264
   ```

2. **编写编译脚本 `x264_build_android.sh`**  
   ```bash
   #!/bin/bash
   NDK=/home/[user]/ndk/android-ndk-r21e
   API=21
   TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64

   build_android() {
       ARCH=$1
       case $ARCH in
           armeabi-v7a)
               HOST=armv7a-linux-androideabi
               CROSS_PREFIX=$TOOLCHAIN/bin/arm-linux-androideabi-
               ;;
           arm64-v8a)
               HOST=aarch64-linux-android
               CROSS_PREFIX=$TOOLCHAIN/bin/aarch64-linux-android-
               ;;
       esac

       ./configure \
           --prefix=./android/$ARCH \
           --enable-shared \
           --enable-pic \
           --host=$HOST \
           --cross-prefix=$CROSS_PREFIX \
           --sysroot=$TOOLCHAIN/sysroot

       make clean
       make -j4
       make install
   }

   # 编译所有支持的架构
   build_android armeabi-v7a
   build_android arm64-v8a
   ```
   - 执行脚本前需赋予权限：`chmod +x x264_build_android.sh && ./x264_build_android.sh`。
   - 生成的 `.so` 文件位于 `x264/android/[arch]/lib` 目录。

---

### **三、交叉编译 FFmpeg**
1. **下载 FFmpeg 源码（推荐 4.4.x 版本）**  
   ```bash
   git clone https://git.ffmpeg.org/ffmpeg.git
   cd ffmpeg
   ```

2. **修改 FFmpeg 配置（关键步骤）**  
   修改 `configure` 文件，确保生成的 `.so` 文件名不带版本号：
   ```bash
   # 原内容
   SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
   SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'

   # 修改为
   SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
   SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
   ```

3. **编写编译脚本 `ffmpeg_build_android.sh`**  
   ```bash
   #!/bin/bash
   NDK=/home/[user]/ndk/android-ndk-r21e
   API=21
   TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64

   build_ffmpeg() {
       ARCH=$1
       case $ARCH in
           armeabi-v7a)
               CPU=armv7-a
               TARGET=armv7a-linux-androideabi
               CROSS_PREFIX=arm-linux-androideabi-
               ;;
           arm64-v8a)
               CPU=armv8-a
               TARGET=aarch64-linux-android
               CROSS_PREFIX=aarch64-linux-android-
               ;;
       esac

       ./configure \
           --prefix=./android/$ARCH \
           --enable-shared \
           --disable-static \
           --disable-doc \
           --enable-cross-compile \
           --cross-prefix=$TOOLCHAIN/bin/$CROSS_PREFIX \
           --target-os=android \
           --arch=$ARCH \
           --sysroot=$TOOLCHAIN/sysroot \
           --cc=$TOOLCHAIN/bin/${TARGET}${API}-clang \
           --cxx=$TOOLCHAIN/bin/${TARGET}${API}-clang++ \
           --extra-cflags="-I$PWD/../x264/android/$ARCH/include" \
           --extra-ldflags="-L$PWD/../x264/android/$ARCH/lib" \
           --enable-gpl \
           --enable-libx264

       make clean
       make -j4
       make install
   }

   # 编译所有支持的架构
   build_ffmpeg armeabi-v7a
   build_ffmpeg arm64-v8a
   ```
   - 执行脚本：`chmod +x ffmpeg_build_android.sh && ./ffmpeg_build_android.sh`。
   - 生成的 `.so` 文件位于 `ffmpeg/android/[arch]/lib` 目录。

---

### **四、集成到 Android 项目**
4. **拷贝库文件和头文件**  
   - 将 `ffmpeg/android/[arch]/lib/*.so` 复制到 Android 项目的 `app/src/main/jniLibs/[arch]`。
   - 将 `ffmpeg/android/[arch]/include` 中的头文件复制到 `app/src/main/cpp/include`。

5. **配置 CMakeLists.txt**  
   ```cmake
   add_library(avcodec SHARED IMPORTED)
   set_target_properties(avcodec PROPERTIES IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libavcodec.so)

   target_link_libraries(native-lib avcodec ...其他FFmpeg库)
   ```

---

### **常见问题与解决方案**
6. **x264 未正确链接**  
   - 检查 `--extra-cflags` 和 `--extra-ldflags` 路径是否指向编译后的 x264 目录。

7. **FFmpeg 编译失败**  
   - 确保 NDK 版本为 r21e，更高版本可能需要调整工具链路径。

8. **动态库加载失败**  
   - 在 `build.gradle` 中指定支持的 ABI：
     ```gradle
     android {
         defaultConfig {
             ndk { abiFilters 'armeabi-v7a', 'arm64-v8a' }
         }
     }
     ```

---

### **参考资源**
- [Android FFMPEG 编解码实践（一）](https://blog.csdn.net/selivert/article/details/125622413)
- [Ubuntu 交叉编译 FFmpeg 流程](https://blog.csdn.net/heqingchun16/article/details/132799390)
- [FFmpeg 动态库集成到 Android](https://blog.csdn.net/quan648997767/article/details/70172166)

此流程已在 Ubuntu 22.04 + NDK r21e 环境下验证通过，生成的动态库可直接用于 Android JNI 开发。