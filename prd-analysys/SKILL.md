---
name: prd-analysis
description: 用于获取页面路径地图，需求拆解，需求估时。当用户获取页面路径地图，需求估时，需求拆解，估时时调用
license: MIT
metadata:
  author: boxu
  version: "1.0"
---


## 必读顺序

1. 阅读 `./references/self-hosting-routing.yaml` 选择一级路由。
2. 仅命中阅路由的 `required_reads` 指定文档，随后执行该路由对应的工作流。
3. 未命中时使用 `other` 路由。



## 路由原则

- `./references/self-hosting-routing.yaml` 是任务路由的单一事实源。
- 路由匹配依据 `labels`、`trigger_examples` 和 task intent。
- `required_reads` 只放完成该类任务必须读取的文件。
- 不得因为存在规范文件就默认读取全部 `./references` `./rules`。


## 上下文截断处理

- 同一会话内新任务必须重新匹配路由。
- 仅在路由变更、上下文被压缩、或无法判断上下文是否仍完整时，重新读取 `skills/SKILL.md` 和路由文件。
- 未变动的背景信息可以复用上下文缓存，不需要每次任务都全量重读。