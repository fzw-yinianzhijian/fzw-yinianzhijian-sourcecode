---
title: openclaw 使用指南
tags:
  - Study
  - Openclaw
createTime: 2026/03/11 22:00:46
permalink: /blog/Study/Openclaw_guide/
---

## 第一部分：openclaw 是什么

openclaw 本质上是一个跑在服务器上的 agent 框架，重点不是“聊天”，而是“执行任务”。

它最实用的点是可以接入聊天平台（如 Discord / Telegram），你可以直接在手机上发消息，让它在服务器上执行动作。

我现在主要把它用在这几类工作：

- 信息抓取（含浏览器操作与登录态场景）
- coding（改代码、跑命令、排错）
- writing（整理文档、生成草稿）
- scheduling（任务拆分与持续执行）

官方文档：

- <https://docs.openclaw.ai>
- <https://github.com/openclaw/openclaw>

## 第二部分：怎么安装

### 2.1 基础环境与协作方式

先准备一台服务器（VPS）。我的配置方式是：本地 Mac 上用 Codex，通过 SSH 连到服务器做安装和调试。

这样做的好处是：环境报错可以直接在终端读日志并立即修，效率比纯手动高很多。

### 2.2 代理配置（这是关键）

OpenClaw 里代理要分两层，不要混：

**A. 全局代理（影响 Codex/OpenAI 请求）**
在 `openclaw-gateway.service` 里配置：

- `Environment=HTTP_PROXY=http://127.0.0.1:7890`
- `Environment=HTTPS_PROXY=http://127.0.0.1:7890`
- `Environment=ALL_PROXY=socks5://127.0.0.1:7890`

**B. Discord 专用代理（只影响 Discord 通道）**
在 `openclaw.json` 中配置：

- `channels.discord.proxy = "http://127.0.0.1:7890"`

结论：Discord 有单独代理字段；Codex 主要走 gateway service 的全局代理。改完 service 记得重启 `openclaw-gateway`。

### 2.3 Discord 接入

官方文档：
<https://docs.openclaw.ai/zh-CN/channels/discord>

更直观的教程：
<https://zhuanlan.zhihu.com/p/2003215600378454774>

流程很直接：创建 Bot Service -> 获取 Token -> 生成 OAuth2 邀请链接 -> 拉进你自己的服务器。

### 2.4 无头浏览器能力验证

我这边验证通过的环境是：

- Chrome: `145.0.7632.159`
- 路径：`/usr/bin/google-chrome`
- 配置：`browser.headless = true`

验证命令与结果：

- `openclaw browser status --json`
  - `detectedBrowser = chrome`
  - `detectedExecutablePath = /usr/bin/google-chrome`
  - `headless = true`
- `openclaw browser start --json`
  - `running = true`
- 实测可抓取：`https://example.com`

## 第三部分：具体应用（目前研究出来的）

### 3.1 我当前稳定在用的场景

- 远程运维：通过聊天消息触发服务器任务（查询、修改、排查）
- 浏览器抓取：让 agent 打开页面并提取信息
- 技术写作：按结构生成文档初稿，再人工修订
- coding 协作：让它执行重复性改动，我负责审核关键决策

### 3.2 踩坑记录（重要）

1. 不要让 openclaw 频繁 `gateway restart`，容易把状态搞乱，后面更难排查。  
2. 不要给它“自改核心配置”的权限。一次错误尝试里，我给了 openrouter API 和模型配置权限，它很快消耗了大量 token，还把默认模型从 codex 改到 `glm-5`，直接导致流程卡死。  
3. 不要用 root 常驻运行。请做权限隔离，尽量限制在普通用户目录。  
4. 服务器会遇到 SSH 攻击，基础防护要先做（密钥登录、限制来源、日志监控）。

### 3.3 出问题时的修复策略

我的做法是：让本地 Codex 通过 SSH 接管排障，而不是让故障中的远端 agent 自救。

最小修复流程：

1. 本地 SSH 登录服务器  
2. 看 gateway / agent 日志定位错误  
3. 回滚错误配置（尤其 `openclaw.json` 里的模型与代理）  
4. 重启服务并做最小验证（模型请求、通道连通、工具可用）

这套方式的核心是：把“执行权限”和“审计判断”分开，先恢复稳定，再继续迭代。
