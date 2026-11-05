# ai-skills

面向 AI 编码代理（如 pi）的**技能（Skill）集合**，每个技能一个目录，通过 `SKILL.md` 描述用途与触发方式。

> 部分技能引用自 [zrong/skills](https://github.com/zrong/skills)，并在其目录内标注了来源。

## 目录

| 技能 | 一句话用途 | 触发示例 | 依赖 |
| --- | --- | --- | --- |
| **[os-use](./os-use/)** | 跨平台（macOS/Windows）操作系统自动化与屏幕控制 | 截屏、鼠标键盘操作、窗口管理、浏览器/桌面自动化 | Python 3.11+；`pyautogui`、`opencv-python`、`Pillow`、`pyperclip`；Windows 额外需 `pywinauto` |
| **[git-commit](./git-commit/)** | 在 CalVer 仓库执行明确要求的提交、打标签、推送、历史整理 | "提交这次改动"、"打 tag"、"整理提交历史" | 无（仅 Python 标准库 + `git`） |
| **[dns-manager](./dns-manager/)** | DNS 解析记录管理（查询/增/改/删，腾讯云 DNSPod，幂等） | "把域名指向某 IP"、"查 A 记录"、"新增 CNAME" | 无（仅 Python 标准库） |
| **[filebrowser](./filebrowser/)** | FileBrowser Quantum 一体化 CLI：文件管理与远程传输 | "浏览远程目录"、"下载/上传文件"、"在服务器上创建目录" | `httpx` |
| **[email](./email/)** | 邮件读写（基于 IMAP/SMTP 的邮件处理工作流） | "帮我读一下邮件"、"给某人发邮件" | 无（仅 Python 标准库） |
| **[feishu-image](./feishu-image/)** | 通过飞书（Lark）消息平台发送图片与截图 | "把截图发到飞书"、"发送图片给某人" | Node.js（`feishu-image.js`） |
| **[object-storage](./object-storage/)** | S3 兼容对象存储与 CDN 管理（S3/COS/OSS/TOS/MinIO） | "上传文件到 bucket"、"刷新 CDN"、"返回对象公开 URL" | `boto3` |
| **[media-use](./media-use/)** | 基于 ffmpeg 的媒体处理（转码、裁剪、水印、合并、修复损坏等） | "转码/压缩视频"、"加水印"、"修复下载损坏的 mp4" | `typer`、`rich`；外加 `ffmpeg` |
| **[image-generation](./image-generation/)** | AI 图片生成/编辑与透明背景抠图（多模型、mask、批量） | "生成封面图"、"改图"、"给图片去背景" | `httpx`；外加各模型 API（OpenAI/Gemini/Seedream 等） |
| **[matting](./matting/)** | 图片抠图、去背景，输出透明 PNG（自动选算法） | "抠图"、"去绿幕"、"人物/产品透明底" | `httpx` |
| **[upscale](./upscale/)** | 图片/视频超分、AI 增强、缩放、RIFE 补帧 | "放大图片"、"24fps 补到 60fps"、"高清修复" | `httpx`、`Pillow` |
| **[video-analyzer](./video-analyzer/)** | 用视觉/视频大模型分析视频内容 | "看看这个视频"、"分析视频内容" | `openai`、`httpx` |
| **[video-downloader](./video-downloader/)** | 视频下载（抖音/视频号/weixin SPH、yt-dlp 支持站点） | "下载这个视频"、"下载抖音视频" | `playwright` |
| **[spritesheet](./spritesheet/)** | 从视频提取帧生成 sprite sheet 与独立透明 PNG | "制作 spritesheet"、"提取动画帧"、"循环动画" | `openai` |
| **[ui-extractor](./ui-extractor/)** | 从静态图片提取前景、分离 UI 组件、透视校正 | "提取 UI 元素"、"分离组件"、"棋盘格抠图" | `opencv-python-headless`、`numpy`、`Pillow` |
| **[immich](./immich/)** | 图片/视频上传到 Immich 并管理服务器资产、相簿、标签 | "上传照片到 Immich"、"建相簿"、"给照片打标签" | `httpx` |
| **[jellyfin](./jellyfin/)** | Jellyfin 媒体库命名、刮削校验、查重、元数据维护 | "按规范重命名电影目录"、"校验 IMDB ID"、"刷新媒体库" | `httpx` |
| **[joplin](./joplin/)** | Joplin REST API 客户端：笔记/笔记本/标签/资源 CRUD | "新增笔记"、"给笔记打标签"、"搜索笔记" | `httpx` |
| **[vikunja](./vikunja/)** | Vikunja 任务管理，每周将已完成任务同步到 Joplin | "查看待办"、"把完成的任务同步到 Joplin" | `httpx` |
| **[mcp-deploy](./mcp-deploy/)** | MCP 服务器自动部署与平台自动配置（智谱/Minimax/Gitea 等） | "部署 MCP"、"配置智谱 MCP" | `mcporter`（外加 `uv` / `uvx`） |
| **[prd-analysys](./prd-analysys/)** | 页面路径地图、需求估时、需求拆解、PRD 撰写 | "拆需求"、"估时"、"写 PRD" | 无（仅 Python 标准库） |
| **[sheet-json](./sheet-json/)** | 文本与 Excel（.xlsx）互转 | "读取 .xlsx"、"把表格写成 .xlsx" | 无（仅 Python 标准库） |
| **[seedance-2-video](./seedance-2-video/)** | Seedance 2.0 图生视频方案、运镜/分镜设计、提示词编写 | "Seedance 2.0 图生视频"、"设计运镜分镜" | 无（仅参考文档与提示词，无代码依赖） |

### 来源标注

- 📦 **[image-generation](./image-generation/)** — 核心 CLI 行为与 chroma-key 工具引用自 OpenAI `imagegen` skill（Apache-2.0），原始本地来源见 [`image-generation/NOTICE.md`](image-generation/NOTICE.md)；同时引用自 [zrong/skills](https://github.com/zrong/skills)。
- 📦 **[vikunja](./vikunja/)** — 引用自 [zrong/skills](https://github.com/zrong/skills)（见 [`vikunja/SKILL.md`](vikunja/SKILL.md)）。
- 📦 **[seedance-2-video](./seedance-2-video/)** — 改编自 [MapleShaw/seedance2.0-prompt-skill](https://github.com/MapleShaw/seedance2.0-prompt-skill)（MIT），详见 [`seedance-2-video/NOTICE`](seedance-2-video/NOTICE)。

## 约定

- 每个技能目录下 **`SKILL.md`** 是该技能的定义文件，包含 `name`、`description`、触发场景与执行方式。
- 配套文件通常为：`scripts/`（执行脚本）、`references/`（参考文档）、`references/rules/`（规则）、`agent_config.example.toml`（配置示例）。
- 触发一个技能时，可调用其入口：
  ```bash
  uv run --project {SKILL_DIR}/scripts {skill-name} --non-interactive <command> --json
  ```

## 结构

```
ai-skills/
├── SKILL.md            # 本项目总览
├── LICENSE
├── shared/             # 共享组件（如 agent-config）
│   └── agent-config/
└── <skill>/            # 每个子目录为一个独立技能
```
