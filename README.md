# ai-skills

面向 AI 编码代理（如 pi）的**技能（Skill）集合**，每个技能一个目录，通过 `SKILL.md` 描述用途与触发方式。

> 


## 使用方法

```shell

# 项目级别安装 支持pi
npx skills add OoadaioO/daily-skills --skill <skill-name> --agent <pi|codex|claude-code|···>

# 全局安装 支持 pi
npx skills add -g OoadaioO/daily-skills --skill <skill-name> --agent <pi|codex|claude-code|···>

```



## 技能清单

| 技能 | 说明 |
| --- | --- |
| make-live-photo | 用 make-live-photo 把图片+视频合成 Apple Live Photo，打包成 .pvt，传 iPhone |
| image-generation | AI 图片生成/编辑，调用 OpenAI/Gemini/Seedream 等，含抠图、透明底、参考图编辑 |
| matting | 用独立 matting-api 抠图、去背景，输出透明 PNG |
| upscale | 用独立 upscale-api 对图片/视频超分、AI 增强、补帧 |
| seedance-2-video | Seedance 2.0 图生视频：提示词、运镜、分镜、首尾帧、长视频规划 |
| video-analyzer | 用视觉/视频大模型分析视频内容 |
| spritesheet | 从视频提取帧生成 spritesheet 和独立透明 PNG |
| media-use | 基于 ffmpeg 的媒体处理：转码、裁剪、水印、合并、修复 m3u |
| video-downloader | 下载抖音、视频号、yt-dlp 支持网站的视频 |
| immich | 上传图片/视频到 Immich，管理相簿、标签、扫描图库 |
| object-storage | S3 兼容对象存储与 CDN 管理，上传到 S3/COS/OSS/MinIO 等 |
| filebrowser | FileBrowser CLI：文件浏览/读写/增删目录 |
| git-commit | 在 CalVer 仓库执行提交、打标签、推送、历史整理 |
| os-use | 跨平台系统自动化与屏幕控制：截图、鼠标键盘、浏览器自动化 |
| dns-manager | 管理 DNS 解析记录（A/CNAME/TXT/MX 等），支持 DNSPod，操作幂等 |
| email | 基于 IMAP 的邮件处理：搜索、读取、下载附件、移动邮件 |
| feishu-image | 通过飞书发送图片和截图 |
| joplin | Joplin REST API 客户端：笔记、笔记本、标签、资源 CRUD |
| vikunja | Vikunja 任务管理，每周已完成任务同步到 Joplin |
| sheet-json | 文本与 Excel 转换：读取/写入 .xlsx、内容互转 |
| ui-extractor | 从静态图提取前景、分离 UI 组件，去背景/棋盘格/绿幕抠图 |
| jellyfin | Jellyfin 媒体库命名规范与服务器维护 |
| mcp-deploy | MCP 服务器自动部署（智谱/Minimax/Gitea 等）|
| prd-analysis | 需求拆解、估时、写 PRD 需求文档 |


## References

部分技能引用 [zrong/skills](https://github.com/zrong/skills) 
