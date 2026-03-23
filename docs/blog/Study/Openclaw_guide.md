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

> 先明确一下：当前这部分记录的是**配置与调试流程**，不是“具体应用案例”。应用侧内容我会放到第三部分后续补充。

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

**openclaw 使用建议：**
其实也和朋友聊了很多，openclaw 这个工具大厂在做出来一键部署的方便上手的产品之前其实对于普通人而言用这个的代价和成本都很高。首先你需要学习服务器命令，学习配置相关的内容，在配置过程中，因为服务器环境等等不少相关限制会出现各种各样的问题，一个没有任何玩电脑经验的小白很容易踩很多很多坑。（啊当然，毕竟我也不是那种传统开发人员，很多东西也还一直在摸索当中）

> 这一部分先留空，后续按真实使用场景逐条补充。

待补充提纲：

- 场景 1：目标、输入、执行流程、输出结果
- 场景 2：目标、输入、执行流程、输出结果
- 场景 3：目标、输入、执行流程、输出结果
- 每个场景对应的风险点与回滚方案
