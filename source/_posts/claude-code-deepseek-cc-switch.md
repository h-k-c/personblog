---
title: 5分钟搞定 Claude Code 接入 DeepSeek（cc-switch 一键切换）
date: 2026-07-21 21:10:00
categories: 技术
tags:
  - Claude Code
  - DeepSeek
  - AI 编程
---

Claude Code 好用是真的好用，但官方 API 按量计费，跑量大的时候钱包实在扛不住。好消息是 DeepSeek 提供了一个跟 Anthropic Messages API 格式兼容的接口，Claude Code 本身不用改一行代码，只要把请求地址"指"到 DeepSeek，就能用白菜价享受接近的体验。

这篇从零开始，把每一步都写清楚——有手就会，5 分钟上车。

## 前置准备

### 1. 安装 Node.js（推荐 NVM）

Claude Code 依赖 Node.js 环境。建议用 NVM 来管理，版本切换方便：

```bash
# 安装 NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# 重新加载 shell 配置
source ~/.zshrc

# 安装 Node.js LTS 版本
nvm install --lts
nvm use --lts
```
确认装好了：

```bash
node -v   # 应输出 v22.x 或类似
npm -v    # 应输出 10.x 或类似
```

### 2. 安装 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

装完验证：

```bash
claude --version
```

### 3. 获取 DeepSeek API Key

1. 打开 [DeepSeek 开放平台](https://platform.deepseek.com/api_keys)
2. 注册 / 登录
3. 点击「创建 API Key」，复制保存好（`sk-` 开头的那串）

{% asset_img deepseek-api-key.png DeepSeek API Key 申请页面 %}

> 💡 新注册通常有赠送额度，够你跑一阵子了。

## 配置 CC Switch

### 4. 安装 CC Switch

[cc-switch](https://github.com/farion1231/cc-switch) 是一个跨平台的图形化配置管理工具，专门用来切换 Claude Code（以及 Codex、Gemini CLI 等）的供应商配置。不用手写 JSON，点一下就能切。

```bash
# macOS，用 Homebrew 一键装好
brew tap farion1231/ccswitch
brew install --cask cc-switch
```

Windows 用户去 [Releases 页面](https://github.com/farion1231/cc-switch/releases) 下载 MSI 或免安装 ZIP；Linux 提供 `.deb` / `.rpm` / AppImage。

### 5. 配置 CC Switch

1. 打开 cc-switch，默认管理的就是 **Claude Code**
2. 右上角点 **"+"** 新增一个供应商

{% asset_img cc-switch-add-provider.png 添加供应商 %}

3. 预设列表里选 **DeepSeek**，把第 3 步拿到的 API Key 粘进去

{% asset_img cc-switch-deepseek-config.png 配置 DeepSeek %}

4. 高级选项里把模型 ID 填成 `deepseek-v4-pro`
5. 回到主界面选中 DeepSeek，点 **启用**

搞定。cc-switch 底层帮你把环境变量写进了 Claude Code 的配置文件，完全不用自己碰 JSON。

## 启动测试

打开终端，敲：

```bash
claude
```

如果一切正常，Claude Code 的交互界面就会启动，背后跑的是 DeepSeek 的模型。

{% asset_img claude-startup.png 启动 Claude Code %}

想验证是不是真的切过去了？在 Claude Code 里直接问：

> "你是什么模型？"

{% asset_img claude-verify-model.png 验证运行的是 DeepSeek %}

它应该会告诉你运行的是 DeepSeek。

想切回官方 Claude？打开 cc-switch，切回原来的供应商，点一下启用——就这么简单，不用翻配置文件，不用重启终端。
