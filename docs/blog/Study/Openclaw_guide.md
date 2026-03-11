---
title: openclaw 使用指南
tags:
  - Study
  - Openclaw
createTime: 2026/03/11 22:00:46
permalink: /blog/Study/Openclaw_guide/
---

## 第一部分：openclaw 是什么

openclaw 说白了，就是一个跑在服务器上的 agent 框架。它最让我眼前一亮的一点，是可以接到多个聊天平台里使用（比如 Discord、Telegram 这类），这样你在手机上就能直接“发消息调度它干活”，交互成本非常低。

我自己上手之后，能明显感受到它不是那种只会聊天的壳子，而是一个“能实际执行任务”的工作流入口。包括但不限于：

- 信息抓取（支持浏览器操作，包含登录态场景）
- coding（写代码、改代码、跑命令、排查问题）
- writing（整理文档、写博客、做结构化输出）
- scheduling（任务编排、分步执行、持续迭代）

说个很主观但真实的感受：第一次把 codex 放在 openclaw 这套命令行原生链路里跑起来时，体验和我在 Cursor 里用模型是两种感觉。

官方文档：

- <https://docs.openclaw.ai>
- <https://github.com/openclaw/openclaw>

## 第二部分：怎么安装

折腾这套环境，首先你得有一台海外服务器（VPS）。我最开始的配置方式并没有选择纯手动敲命令，而是**用本地的 Codex 通过 SSH 直连到服务器上进行配置**。这种“用魔法打败魔法”的协作方式非常顺手，本地 AI 帮你敲命令、改配置，能省去很多复制粘贴的体力活。

接下来是具体的安装和配置细节，重点说几个容易卡脖子的地方。

### 2.1 服务器与 SSH 协作方式

强烈建议大家像我一样，把繁琐的初始化工作交给本地的 Codex 代理。你只需要把服务器的 IP、端口和凭证给到本地 AI，让它通过 SSH 连过去，帮你拉取代码、建目录。这种方式不仅效率高，而且在遇到环境依赖报错时，本地 AI 能直接读取终端的错误日志并当场给出修复方案，体验极佳。

### 2.2 代理配置（全局 vs Discord 专用）

这是个大坑！调用 Codex 模型是必须配代理的，但 OpenClaw 的代理逻辑分两层，很多人在这里绕晕了。我的最终解法是明确区分“全局”和“Discord 专用”：

**1) 全局代理（影响 Codex/OpenAI 的底层请求）**
这部分必须写在 systemd 的用户服务里。你需要编辑 `openclaw-gateway.service` 文件，在里面加上这三行环境变量：
*   `Environment=HTTP_PROXY=http://127.0.0.1:7890`
*   `Environment=HTTPS_PROXY=http://127.0.0.1:7890`
*   `Environment=ALL_PROXY=socks5://127.0.0.1:7890`

**2) Discord 专用代理（只影响 Discord 通道的消息收发）**
Discord 的代理不需要写在系统服务里，它在配置文件中有专门的字段。你需要打开 `openclaw.json`，找到 channels 配置，这样写：
`channels.discord.proxy = "http://127.0.0.1:7890"`

**核心结论：** Discord 有自己单独的代理字段，而 Codex 核心模型的请求主要走 `openclaw-gateway.service` 里配的全局代理。**记住，改完 systemd 里的代理环境变量后，一定要重启 `openclaw-gateway` 服务才会生效！**

### 2.3 Discord 接入

把 AI 接入 Discord 频道其实不难。官方虽然提供了文档（[官方文档链接](https://docs.openclaw.ai/zh-CN/channels/discord)），但说实话，我觉得知乎上的这篇教程更直接、更好懂：[知乎教程链接](https://zhuanlan.zhihu.com/p/2003215600378454774)。

核心步骤就两步：去 Discord Developer Portal 建一个 Bot Service，拿到 Token；然后生成 OAuth2 链接，把这个机器人拉进你自己的 Discord 服务器里。照着知乎那篇图文做，基本一遍过。

### 2.4 无头浏览器能力验证

为了让 AI 具备上网查资料的能力，我给它配置了无头浏览器（Headless Browser）。

我在服务器上安装了 Google Chrome（版本号 145.0.7632.159，路径在 `/usr/bin/google-chrome`）。
在配置里设置 `browser.headless = true` 后，我跑了几个命令验证：
*   输入 `openclaw browser status --json`，系统完美识别：`detectedBrowser=chrome`, `detectedExecutablePath=/usr/bin/google-chrome`, `headless=true`。
*   接着输入 `openclaw browser start --json`，返回 `running=true`，启动成功。
*   最后让它去抓取 `https://example.com`，顺利打开并拿到了网页数据。至此，AI 的“眼睛”算是正式睁开了。

---

## 第三部分：具体应用（目前研究出来的）

跑通基础流程只是第一步，真正让它跑起来之后，才是各种“惊喜”和“惊吓”的开始。分享一下我实操中遇到的真实情况。

### 3.1 我踩过的坑

**坑一：千万别让 OpenClaw 频繁重启自己的网关。**
在调试阶段，有时候 AI 会试图通过 `gateway restart` 来解决它遇到的网络或配置问题。千万别让它这么干！频繁重启会导致进程僵死或者状态丢失，最后还得你手动去服务器上擦屁股。

**坑二：一次极其惨痛的“AI 审查 AI”翻车事件。**
有一次我犯了个严重的 coding 错误：我给了它 OpenRouter 的 API 和完整的模型调用权限，然后异想天开地让它“自己 review 自己的配置”。
结果这货直接放飞自我了——它在极短的时间内疯狂循环调用，迅速烧掉了一大波 Token。更离谱的是，它在 review 的过程中，居然把自己的默认底层模型从 Codex 换成了 `glm-5`！这一换直接导致整个系统逻辑不兼容，当场卡死罢工。

### 3.2 安全提醒（最小权限、密钥边界、模型边界）

结合上面的血泪史，我总结了三条绝对不能妥协的安全底线：

1.  **权限隔离（最小权限原则）：** 绝对、绝对不要用 root 账号跑这个服务！一定要建一个普通用户，把它的活动范围严格限制在普通用户目录下。AI 执行命令是不可控的，给 root 权限等于把服务器的自毁按钮交给了它。
2.  **网络安全防护：** 在折腾的中途，我的服务器遭遇了来自神秘地区的 SSH 暴力登录攻击，一度导致我自己的终端都登不上服务器。大家一定要做好基础防护（比如改默认 SSH 端口、用密钥登录禁用密码、装 fail2ban 等）。
3.  **守住密钥与模型边界：** 就像我踩的坑一样，永远不要给 AI 修改自身核心配置（尤其是 API Key 和默认模型）的权限。要明确告诉它：你可以用工具，但你不能换你自己的“脑子”。

### 3.3 出问题时的修复策略

当远端的 OpenClaw 因为配置错误、模型乱改（比如变成 glm-5 卡死）或者代理失效而彻底失联时，不要慌，也不用苦哈哈地纯手动去翻日志。

**我的最佳修复策略是：直接呼叫“场外援助”。**
让**本地的 Codex 代理**通过 SSH 重新登录到服务器上。因为本地的 AI 是清醒的，且不受远端崩溃环境的影响。你可以直接让本地 Codex 去 `tail` 远端的错误日志，定位问题后，让它帮你把改错的 `openclaw.json` 还原，最后重启服务。这种“用本地 AI 抢救远端 AI”的套路，是目前最高效的排障方法。
