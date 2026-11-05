---
name: make-live-photo
description: 实况图片生成工具。当用户要制作 live photo、生成实况图片时使用。
---

# make-live-photo

把一张图片和一段视频合成为 Apple **Live Photo**，并打包成 `.pvt` 文件。
生成的 `.pvt` 可以直接通过隔空投送（AirDrop）发到 iPhone，接收后自动保存为实况照片，
特别适合把安卓手机拍的图转成 iPhone 的 Live Photo。

工具：`make-live-photo`（git 工具，仓库 `OoadaioO/make-live-photo`）。


## 环境检测

**先确认环境，可用再运行；不可用则尝试安装依赖。**

**必须依赖（缺失必装）**：

- Python 3.9+
- `make-live-photo` 工具（安装本工具自带 `makelive`）

**可选依赖**：

- 系统 `ffmpeg` / `ffprobe`（可选）——仅从视频抽封面时需要，只传图片 + 视频时不需要

检测与安装顺序：

**1. 检测必须依赖（缺失必装）**

```bash
command -v python && command -v make-live-photo
```

缺 python 或 `make-live-photo` 时，安装本工具（自带 `makelive`）：

```bash
pip install git+https://github.com/OoadaioO/make-live-photo.git
```

安装本包时会一并安装 `makelive`；若 pip 解析依赖失败，可换源重试。

**2. 检测可选依赖（缺可不装，仅在抽封面时需要）**

```bash
command -v ffmpeg
```

缺 ffmpeg 且本次要 `-v` 抽封面时，才安装系统 ffmpeg（`ffprobe` 可选，需要时用 `ffprobe -v` 检测）：

```bash
brew install ffmpeg
```

ffmpeg 仅在 `-v` 抽封面时需要，只传图片 + 视频时不需要。

## 工作流

### 1. 确认素材与目标

- **视频**：必填，Live Photo 要求 Apple 能识别的 QuickTime 格式（`.mp4` 或 `.mov` 优先 `mov` ）。
- **图片**：可选，`.jpg / .jpeg / .heic`；省略时用视频**第一帧**做封面。
- **封面时间**：若用 `-t` 指定时间点，则忽略 `-i` 图片。


### 2. 调用工具

```bash
# 图片 + 视频合成，输出到视频同目录的 clip.pvt
make-live-photo --image photo.jpg --video clip.mov

# 只用视频：用视频第一帧做封面
make-live-photo -v clip.mov

# 用指定时间点（秒 / MM:SS / HH:MM:SS）的帧做封面；指定 -t 后忽略图片
make-live-photo -v clip.mov -t 1.5
make-live-photo -v clip.mov -t 00:00:01.5

# 自定义输出路径（-o 可选，不会自动补 .pvt）
make-live-photo -i photo.heic -v video.mov -o output.pvt

# 等价入口
python -m make_live_photo -i photo.jpg -v clip.mov
```

常用选项：

| 选项 | 说明 |
| --- | --- |
| `-i, --image` | 可选：图片路径（`.jpg`/`.jpeg`/`.heic`）。省略时用视频第一帧 |
| `-t, --time` | 可选：封面时间点（秒、`MM:SS` 或 `HH:MM:SS`）。指定后忽略 `-i` |
| `-v, --video` | **必填**：视频路径（QuickTime 格式，`.mov` 最稳妥） |
| `-o, --output` | 可选：输出 `.pvt` 路径。默认视频目录下与视频同名的 `.pvt`，已存在则覆盖 |

等价于 `python -m make_live_photo`。

### 3. 处理失败

- 工具调用失败如实报告，不要伪装成功。
- 缺 `ffmpeg`（仅 `-v` 抽封面）或视频格式不被 iOS 识别：缺可选依赖时，见 [环境检测](#环境检测) 的安装命令。


## 交付

完成后报告：

- 输入（图片/视频绝对路径）、输出 `.pvt` 绝对路径
- 封面来源：指定图片，还是视频第一帧 / 指定时间点抽取
- 工具是否成功、是否触发回退（如提示安装 ffmpeg）

## 注意事项

- 底层依赖：系统 `ffmpeg`（仅 `-v` 抽封面时需要，不随包分发）；`makelive`（写入元数据并打包，随本包安装）。
- 打包只操作副本，不修改原始图片与视频。
- `.pvt` 内嵌 QuickTime 元数据，可能无法作为 Live Wallpaper（动态壁纸）使用。

## 问题指引

- 输入是 `.mp4` 不能正常显示时，尝试 `ffmpeg` 转换成 `.mov` 重新生成