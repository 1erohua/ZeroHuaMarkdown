#  多版本管理


在 Ubuntu 22.04 上管理多版本 FFmpeg 的常见解决方案如下，我会推荐方法二（静态编译方案）作为最优解：

### 方法一：PPA 共存方案（推荐新手）
```bash
# 添加新版 PPA（非覆盖式）
sudo add-apt-repository ppa:jonathonf/ffmpeg6
sudo apt update

# 安装新版时使用不同二进制名称
sudo apt install ffmpeg-6.0 ffmpeg

# 验证版本
ffmpeg-6.0 -version  # 新版
ffmpeg -version      # 系统原版 4.4.2

# 通过别名快速调用（添加到 ~/.bashrc）
echo 'alias ffmpeg6="/usr/bin/ffmpeg-6.0"' >> ~/.bashrc
source ~/.bashrc
```

### 方法二：静态编译方案（推荐高级用户）
```bash
# 下载最新静态编译版（以 7.0 为例）
mkdir ~/ffmpeg-latest && cd ~/ffmpeg-latest
wget https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz
tar xvf ffmpeg-release-amd64-static.tar.xz
cd ffmpeg-*-static

# 设置专用环境变量（添加到 ~/.bashrc）
echo 'export PATH="$HOME/ffmpeg-latest/ffmpeg-7.0-amd64-static:$PATH"' >> ~/.bashrc
source ~/.bashrc

# 验证（优先使用静态版）
ffmpeg -version  # 显示 7.0
/usr/bin/ffmpeg -version  # 系统原版 4.4.2
```

### 方法三：容器化方案（适合开发环境）
```bash
# 安装 Docker
sudo apt install docker.io

# 运行最新版 FFmpeg 容器
docker run -v $(pwd):/workdir jrottenberg/ffmpeg:latest \
ffmpeg -version

# 创建快捷脚本 ffmpeg-new
echo '#!/bin/sh
docker run -v $(pwd):/workdir jrottenberg/ffmpeg:latest ffmpeg "$@"
' | sudo tee /usr/local/bin/ffmpeg-new
sudo chmod +x /usr/local/bin/ffmpeg-new
```

### 版本切换技巧
创建多版本切换脚本 `/usr/local/bin/ffmpeg-switch`：
```bash
#!/bin/bash
versions=($(ls /usr/bin/ffmpeg* /opt/ffmpeg*/ffmpeg 2>/dev/null))
PS3="Select FFmpeg version: "
select ver in "${versions[@]}"; do
    sudo ln -sf $ver /usr/local/bin/ffmpeg
    break
done
```

### 各方案对比
| 方案        | 隔离性 | 易用性 | 维护性 | 性能 |
|------------|--------|--------|--------|------|
| PPA 共存    | ★★★☆   | ★★★★★ | ★★★★☆ | ★★★★ |
| 静态编译    | ★★★★★ | ★★★☆☆ | ★★★☆☆ | ★★★★★ |
| 容器化      | ★★★★★ | ★★☆☆☆ | ★★★★★ | ★★☆☆ |

建议根据需求选择：
- 日常简单使用 → 方法一（PPA方案）
- 专业视频处理 → 方法二（静态编译）
- 开发测试环境 → 方法三（容器化）

所有方案都不会影响系统自带的 ffmpeg 4.4.2，通过不同调用方式实现版本共存。


# ffmpeg 6.1.2安装、编译选项


以下是手动编译安装 FFmpeg 6.1.2 到 `/opt/ffmpeg` 并实现版本共存的完整流程：

---

### **1. 安装编译依赖**
```bash
sudo apt update
sudo apt install -y \
  build-essential \
  nasm \
  yasm \
  libx264-dev libx265-dev libvpx-dev \
  libfdk-aac-dev libmp3lame-dev libopus-dev \
  libass-dev libfreetype6-dev libsdl2-dev \
  libtool pkg-config texinfo cmake \
  zlib1g-dev libssl-dev
```

---

### **2. 下载源码**
```bash
# 创建专用目录
mkdir -p ~/ffmpeg_src && cd ~/ffmpeg_src

# 下载指定版本
wget https://ffmpeg.org/releases/ffmpeg-6.1.2.tar.xz
tar xvf ffmpeg-6.1.2.tar.xz
cd ffmpeg-6.1.2
```

---

### **3. 配置编译选项**
```bash
./configure \
  --prefix=/opt/ffmpeg \
  --enable-gpl \
  --enable-version3 \
  --enable-nonfree \
  --enable-libx264 \
  --enable-libx265 \
  --enable-libvpx \
  --enable-libfdk-aac \
  --enable-libmp3lame \
  --enable-libopus \
  --enable-libass \
  --enable-openssl \
  # NVIDIA 加速部分
  --enable-cuda-nvcc \
  --enable-nvdec \
  --enable-nvenc \
  --enable-cuvid \
  --enable-ffnvcodec \
  # AMD/通用加速部分
  --enable-opencl \
  --enable-vaapi \
  --enable-amf \
  # 路径配置
  --extra-cflags="-I/opt/ffmpeg/include -I/usr/local/cuda/include" \
  --extra-ldflags="-L/opt/ffmpeg/lib -L/usr/local/cuda/lib64"

```

---

### **4. 编译安装**
```bash
# 多线程编译（根据CPU核数调整）
make -j$(nproc)

# 安装到系统目录
sudo make install
```

---

### **5. 环境变量配置**
```bash
# 编辑bashrc
echo 'export FFMPEG_NEW="/opt/ffmpeg/bin"' | sudo tee -a /etc/profile.d/ffmpeg.sh
echo 'export PATH="$FFMPEG_NEW:$PATH"' | sudo tee -a /etc/profile.d/ffmpeg.sh
echo 'export LD_LIBRARY_PATH="/opt/ffmpeg/lib:$LD_LIBRARY_PATH"' | sudo tee -a /etc/profile.d/ffmpeg.sh

# 立即生效
source /etc/profile.d/ffmpeg.sh
```

---

### **6. 版本验证**
```bash
# 新版本验证
ffmpeg -version | grep 'version'  # 应显示6.1.2
which ffmpeg                     # 应显示/opt/ffmpeg/bin/ffmpeg

# 系统原版验证
/usr/bin/ffmpeg -version | grep 'version'  # 应保持4.4.2
```

---

### **7. 使用技巧**
#### 调用新版：
```bash
ffmpeg -i input.mp4 ...  # 默认使用6.1.2
```

#### 调用旧版：
```bash
/usr/bin/ffmpeg -i input.mp4 ...  # 明确使用系统4.4.2
```

#### 创建快捷别名：
```bash
echo 'alias ffmpeg-old="/usr/bin/ffmpeg"' >> ~/.bashrc
source ~/.bashrc
```

---

### **8. 卸载方法**
```bash
# 进入源码目录
cd ~/ffmpeg_src/ffmpeg-6.1.2

# 执行卸载
sudo make uninstall

# 删除环境变量
sudo rm /etc/profile.d/ffmpeg.sh
```

---

### **常见问题处理**
1. **依赖库冲突**：
   - 若出现 `ERROR: libx264 not found` 等提示：
   ```bash
   sudo apt install -y libx264-dev libx265-dev
   ```

2. **更新版本**：
   ```bash
   # 重新下载新源码包，重复编译流程即可
   # 旧版本会自动被覆盖
   ```

3. **多版本共存验证**：
   ```bash
   # 对比输出差异
   ffmpeg -version | grep configuration
   /usr/bin/ffmpeg -version | grep configuration
   ```

---

通过这种方式，您可以：
- 保留系统原生 4.4.2 版本（位于 `/usr/bin`）
- 使用自定义编译的 6.1.2 版本（位于 `/opt/ffmpeg`）
- 通过环境变量自动优先调用新版
- 明确路径调用旧版实现精准控制