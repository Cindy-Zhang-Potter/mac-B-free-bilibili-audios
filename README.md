# 🎵 Mac-B-Free-Bilibili-Audios

> mac电脑快速/批量提取并下载B站视频的音频，可用于VIP音乐下载，仅需提供B站链接和下载目录

> **只需粘贴链接，无需任何配置，像 VIP 一样听歌**

---

## ✨ 特性

- ✅ **一键运行** – 粘贴即用，无需任何环境配置
- ✅ **自动安装** – 自动检查并安装 Homebrew、yt-dlp、ffmpeg
- ✅ **批量下载** – 支持多个链接，自动命名、自动整理
- ✅ **最高音质** – 默认 bestaudio + M4A 格式（可改 MP3）
- ✅ **纯净免费** – 开源脚本，无广告，无收费

---

## 🚀 3 分钟快速上手

### 1️⃣ 创建脚本文件

打开终端，逐行输入：

```bash
touch down.sh
```

```bash
open down.sh
```

系统会自动打开一个空白文本。

---

### 2️⃣ 粘贴脚本代码并修改配置

**将下方完整脚本复制粘贴到空白文件中**，然后**立即修改两处配置**：

```bash
#!/bin/bash

# ===== 必改配置 =====
# 修改1：设置你的音频保存目录，替换在引号里面，引号里面只是示例地址
SAVE_DIR="/Users/你的用户名/Downloads/B站音乐"
# ===================

mkdir -p "$SAVE_DIR"

# 检查并安装依赖
install_dependencies() {
    # 检查 Homebrew
    if ! command -v brew &> /dev/null; then
        echo "正在安装 Homebrew..."
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
        if [[ -f /opt/homebrew/bin/brew ]]; then
            eval "$(/opt/homebrew/bin/brew shellenv)"
        elif [[ -f /usr/local/bin/brew ]]; then
            eval "$(/usr/local/bin/brew shellenv)"
        fi
    fi

    # 检查 yt-dlp
    if ! command -v yt-dlp &> /dev/null; then
        echo "正在安装 yt-dlp..."
        brew install yt-dlp
    fi

    # 检查 ffmpeg
    if ! command -v ffmpeg &> /dev/null; then
        echo "正在安装 ffmpeg..."
        brew install ffmpeg
    fi

    echo "依赖检查完成！"
}

# 批量下载音频的核心函数
download_audio() {
    local url="$1"

    # 提取视频标题
    local title=$(yt-dlp --get-title "$url" 2>/dev/null)

    # 如果获取标题失败，使用默认名称
    if [ -z "$title" ] || [ "$title" = "NA" ]; then
        title="未知标题_$(date +%Y%m%d_%H%M%S)"
        echo "警告：无法获取视频标题，使用默认名称: $title"
    fi

    # 清理标题中的非法字符
    title=$(echo "$title" | tr '/:*?"<>|' '_' | sed 's/\\//g' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')

    echo "====================================="
    echo "开始下载：$title"
    echo "原始链接：$url"
    echo "====================================="

    # 下载最高音质音频
    if yt-dlp -f bestaudio --extract-audio --audio-format m4a \
       --audio-quality 0 \
       -o "$SAVE_DIR/%(title)s.%(ext)s" \
       --no-playlist \
       "$url" 2>&1; then
        echo "✓ 下载完成: $title"
        return 0
    else
        echo "✗ 下载失败: $title"
        return 1
    fi
}

# 安装依赖
install_dependencies

# 检查 yt-dlp 是否安装成功
if ! command -v yt-dlp &> /dev/null; then
    echo "错误：yt-dlp 安装失败，请手动安装"
    echo "可以尝试: pip3 install yt-dlp"
    exit 1
fi

# 检查 ffmpeg 是否安装成功
if ! command -v ffmpeg &> /dev/null; then
    echo "警告：ffmpeg 未安装，音频转换可能受限"
    echo "可以尝试: brew install ffmpeg"
fi

# ===== 必改配置 =====
# 修改2：在这里粘贴你要下载的B站链接，替换引号里的链接，引号里面的只是示例链接
VIDEO_URLS=(
    "https://www.bilibili.com/video/BV1EH4y1S7tn"
    "https://www.bilibili.com/video/BV14y4y1h7wn"
    "https://www.bilibili.com/video/BV1TV4y1J7ix"
    # 在这里继续添加更多链接，每行一个
)
# ===================

# 统计变量
total=${#VIDEO_URLS[@]}
success=0
failed=0

echo "====================================="
echo "开始批量下载音频"
echo "总计: $total 个视频"
echo "保存目录: $SAVE_DIR"
echo "====================================="

# 遍历链接并下载
for i in "${!VIDEO_URLS[@]}"; do
    url="${VIDEO_URLS[$i]}"
    current=$((i+1))
    
    echo ""
    echo "[$current/$total] 处理中..."
    
    if download_audio "$url"; then
        ((success++))
    else
        ((failed++))
        echo "$url" >> "$SAVE_DIR/failed_downloads.txt"
    fi
    
    sleep 2
done

echo ""
echo "====================================="
echo "批量下载完成！"
echo "成功: $success / 失败: $failed"
echo "文件保存在: $SAVE_DIR"
if [ $failed -gt 0 ]; then
    echo "失败的链接已保存到: $SAVE_DIR/failed_downloads.txt"
fi
echo "====================================="
```

**✅ 粘贴后立即修改两处：（在脚本里已以“必改配置”标注）**
1. **第5行**：`SAVE_DIR` 改成你想要的下载目录
2. **第90行起**：`VIDEO_URLS` 里面粘贴你要下载的B站链接

---

### 3️⃣ 保存文件

按 `Command + S` 保存，然后关闭文本编辑器。

---

### 4️⃣ 赋予执行权限

回到终端，输入：

```bash
chmod +x down.sh
```

---

### 5️⃣ 开始下载！

```bash
./down.sh
```

静待片刻，音频文件就会自动保存到你设置的目录中 🎶

---

## ⚙️ 自定义配置

| 配置项 | 说明 | 默认值（示例） |
|--------|------|----------------|
| `SAVE_DIR` | 音频保存路径 | `/Users/你的用户名/Downloads/B站音乐` |
| `--audio-format` | 音频格式 | `m4a`（可改为 `mp3`） |
| `--audio-quality` | 音质 | `0`（0=最佳，5=好，9=普通） |

---

## 📱 配合手机使用

下载完成后，**AirDrop 到 iPhone** → **用 VLC 播放**：

1. Mac 右键音频文件 → 共享 → AirDrop → 选择你的 iPhone
2. iPhone 点接收 → 保存到「文件」APP
3. 打开 VLC → 点「浏览」→ 选择「本地存储」→ 找到文件播放

---

## ❓ 常见问题

**Q：提示 “Permission denied”？**

```bash
chmod +x down.sh
```

**Q：下载失败/卡住？**

- 检查网络连接
- 尝试单个链接下载
- 部分 VIP 视频需要登录

**Q：如何下载 MP3 格式？**

将脚本第 54 行的 `--audio-format m4a` 改为 `--audio-format mp3`。

**Q：脚本可以重复使用吗？**

可以。每次运行前修改 `VIDEO_URLS` 里的链接即可，保存目录不用重复设置。

---

## 📄 开源协议

MIT License · 自由使用 · 禁止倒卖

---

**Happy downloading! 🎧**
