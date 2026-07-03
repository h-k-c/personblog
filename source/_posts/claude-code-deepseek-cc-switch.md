---
title: Claude Code 接入 DeepSeek 保姆级教程(用 cc-switch 一键切换)
date: 2026-07-03 10:00:00
categories: 技术
tags:
  - Claude Code
  - DeepSeek
  - AI 编程
---

Claude Code 是 Anthropic 官方的命令行 AI 编程工具,体验是真的好,但官方 API 按量计费,跑量大的时候钱包压力不小。好消息是 DeepSeek 提供了一个跟 Anthropic Messages API 格式兼容的接口,意味着 Claude Code 本身不用改一行代码,只要把它请求的地址"指"到 DeepSeek,就能用 DeepSeek 的模型跑 Claude Code 这套壳子——价格便宜不少。

这篇记录一下具体怎么配,以及踩过的坑。

## 原理:为什么能这么干

Claude Code 判断该往哪发请求,靠的是几个环境变量,核心是:

- `ANTHROPIC_BASE_URL`:请求发到哪个地址
- `ANTHROPIC_AUTH_TOKEN`:鉴权用的密钥

DeepSeek 专门做了一个地址 `https://api.deepseek.com/anthropic`,收发的报文格式跟 Anthropic 官方 API 对得上。所以只要把 `ANTHROPIC_BASE_URL` 指向这个地址,`ANTHROPIC_AUTH_TOKEN` 换成 DeepSeek 的 API Key,Claude Code 就会以为自己还在跟 Anthropic 说话,实际上后面接的是 DeepSeek 的模型。

模型名会做一层映射,不用自己对应:

- 以 `claude-opus` 开头的模型请求 → 转发给 `deepseek-v4-pro`
- 以 `claude-haiku` / `claude-sonnet` 开头的 → 转发给 `deepseek-v4-flash`

## 方式一:环境变量(最直接,但每次开新终端要重设)

先去 [DeepSeek 开放平台](https://platform.deepseek.com/api_keys) 申请一个 API Key,然后:

```bash
# macOS / Linux
export ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
export ANTHROPIC_AUTH_TOKEN=<你的 DeepSeek API Key>
export ANTHROPIC_MODEL=deepseek-v4-pro
export ANTHROPIC_DEFAULT_OPUS_MODEL=deepseek-v4-pro
export ANTHROPIC_DEFAULT_SONNET_MODEL=deepseek-v4-pro
export ANTHROPIC_DEFAULT_HAIKU_MODEL=deepseek-v4-flash
export CLAUDE_CODE_SUBAGENT_MODEL=deepseek-v4-flash
export CLAUDE_CODE_EFFORT_LEVEL=max
```

Windows PowerShell 把 `export` 换成 `$env:`、等号两边写法调整一下就行:

```powershell
$env:ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"
$env:ANTHROPIC_AUTH_TOKEN="<你的 DeepSeek API Key>"
```

这种方式的问题是:只在当前终端会话生效,关掉窗口就没了,得写进 `.zshrc` / `.bashrc` 才能长期保留。

## 方式二:写进 settings.json(推荐,一次配置持久生效)

Claude Code 支持在 `~/.claude/settings.json` 里通过 `env` 字段配置环境变量,不用每次开终端手动 export:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "<你的 DeepSeek API Key>",
    "ANTHROPIC_MODEL": "deepseek-v4-pro",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-v4-pro",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-pro",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash",
    "CLAUDE_CODE_EFFORT_LEVEL": "max"
  }
}
```

## 方式三:用 cc-switch 图形化一键切换(懒人推荐)

手动改 `settings.json` 有个麻烦事:想切回官方 Claude、或者在多个供应商之间来回换,每次都要手动改文件、还容易改错格式。[cc-switch](https://github.com/farion1231/cc-switch) 这个开源工具就是干这个的——一个跨平台桌面 GUI,专门管理 Claude Code(以及 Codex、Gemini CLI 等)的供应商配置,点一下就能切换,底层帮你把配置写进对应工具的配置文件里。

**安装:**

```bash
# macOS,用 Homebrew
brew tap farion1231/ccswitch
brew install --cask cc-switch
```

Windows 用户去 [Releases 页面](https://github.com/farion1231/cc-switch/releases) 下 MSI 或者免安装 ZIP;Linux 有 `.deb` / `.rpm` / AppImage 可选。

**配置 DeepSeek:**

1. 打开 cc-switch,默认管理的是 Claude Code
2. 右上角点"+"新增一个供应商
3. 预设列表里选 DeepSeek,把 API Key(`sk-` 开头)粘进去
4. 高级选项里把模型 ID 填成 `deepseek-v4-pro`
5. 回到主界面选中 DeepSeek,点启用

它的原理其实就是自动帮你改写 `~/.claude/settings.json`,好处是切换、回滚都可视化,不用担心手滑改错 JSON 格式。

## 踩坑记录

- **不要同时设置 `ANTHROPIC_API_KEY`**:只用 `ANTHROPIC_AUTH_TOKEN`,如果 `ANTHROPIC_API_KEY` 也被设置了,Claude Code 会弹出密钥冲突的提示。
- **模型名不要写旧的**:`deepseek-chat`、`deepseek-reasoner` 这两个旧模型名计划在 2026 年 7 月 24 日下线,配置里如果还写着这两个名字,到时候会直接失效,统一换成 `deepseek-v4-pro` / `deepseek-v4-flash`。
- **联网搜索工具会产生额外请求**:Claude Code 的 Web Search 功能背后会多打几次模型调用,用量比想象中大一些,注意别被账单吓到。

配完之后,正常用 `claude` 命令进去就行,该有的功能都在,只是后面跑的模型换成了 DeepSeek。
