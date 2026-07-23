---
title: Cloudflare Pages 部署教程
date: 2026-07-23 16:00:00
categories: 技术
tags:
  - Cloudflare
  - 部署
  - 静态网站
---

## 一、Cloudflare 简介

[Cloudflare](https://www.cloudflare.com/) 是全球领先的互联网基础设施和安全公司，提供 CDN 加速、DDoS 防护、DNS 解析、SSL 证书等服务。其核心优势在于：

- **全球边缘网络**：覆盖 330+ 城市、120+ 国家，用户请求自动路由到最近节点，延迟极低
- **免费额度慷慨**：大量核心功能提供免费版本，个人开发者和小项目可以零成本使用
- **开发者友好**：提供 Workers（边缘计算）、Pages（静态托管）、R2（对象存储）等一系列产品，生态完善

本教程使用的是其中的 **Cloudflare Pages** 功能——一个免费的静态网站托管服务，支持 GitHub 推送自动构建部署、自带 HTTPS、自定义域名，非常适合部署**静态博客**、**文档站**、**前端项目**、**Landing Page** 等。

### 总体流程

{% asset_img deploy-flow.png Cloudflare Pages 部署流程 %}

整个部署流程分为四步：**本地开发** → **推送代码到 GitHub** → **Cloudflare 自动构建部署** → **全球 CDN 分发上线**。之后每次 `git push` 都会自动触发这一链路，无需手动操作。

---

## 二、前置准备：注册账号

在开始部署之前，你需要准备好 **GitHub** 和 **Cloudflare** 两个账号。如果已经有了，可以跳过对应部分。

### 2.1 注册 GitHub 账号

1. 访问 [github.com/signup](https://github.com/signup)
2. 输入你的**邮箱地址**，点击 `Continue`
3. 设置一个**密码**（至少 8 个字符，包含数字或特殊符号），点击 `Continue`
4. 输入一个**用户名**（将作为你 GitHub 个人主页的地址，如 `github.com/你的用户名`），点击 `Continue`
5. 选择是否接收产品更新邮件（可按需选择），点击 `Continue`
6. 完成**人机验证**（按提示操作即可）
7. 输入邮箱收到的**验证码**，验证通过后即注册成功

> 💡 **提示**
> - 用户名一旦创建后修改会影响仓库 URL，建议一开始就取一个合适的名字
> - 建议使用常用邮箱注册，方便接收后续的通知和验证

### 2.2 注册 Cloudflare 账号

1. 访问 [dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)
2. 填写**邮箱地址**和**密码**（密码至少 8 个字符）
3. 点击 `Create Account`
4. 前往邮箱点击**验证链接**完成邮箱验证
5. 验证成功后自动登录 Cloudflare Dashboard

> 💡 **提示**
> - Cloudflare 免费版已经包含 Pages 的全部功能，无需付费
> - 注册时不需要绑定信用卡

## 三、GitHub 仓库准备

### 3.1 创建仓库

1. 登录 [GitHub](https://github.com)，点击右上角 `+` → `New repository`
2. 填写仓库名称（如 `my-site`），选择 Public 或 Private
3. 点击 `Create repository`

### 3.2 推送项目代码

```bash
# 初始化本地仓库（如果还没有）
git init
git add .
git commit -m "init: 初始化项目"

# 关联远程仓库并推送
git remote add origin https://github.com/你的用户名/my-site.git
git branch -M main
git push -u origin main
```

> 💡 **提示**
> 确保项目根目录包含构建产物所需的配置文件（如 `package.json`、`hugo.toml`、`mkdocs.yml` 等）。

---

## 四、配置 Cloudflare Pages

### 4.1 登录 Cloudflare Dashboard

1. 访问 [dash.cloudflare.com](https://dash.cloudflare.com) 并登录

### 4.2 创建 Pages 项目

1. 左侧菜单选择 **Workers & Pages**
2. 点击 **Create** → 选择 **Pages** 标签页
3. 选择 **Connect to Git**

### 4.3 授权并连接 GitHub

1. 选择 **GitHub** 作为 Git 提供商
2. 首次使用需授权 Cloudflare 访问你的 GitHub 账号
3. 选择要部署的仓库（`my-site`）
4. 选择分支（通常为 `main`）

### 4.4 配置构建设置

根据项目类型填写：

| 项目类型 | 构建命令 | 输出目录 |
|---------|---------|---------|
| Hugo | `hugo --minify` | `public` |
| VitePress | `npm run build` | `.vitepress/dist` |
| MkDocs | `mkdocs build` | `site` |
| Next.js (静态导出) | `npm run build` | `out` |
| 纯 HTML | 留空 | `/`（根目录） |

其他配置：
- **Root directory**：如果项目在子目录中，填写子目录路径
- **Environment variables**：按需添加（如 `NODE_VERSION=18`）

### 4.5 部署

点击 **Save and Deploy**，等待构建完成（通常 1-3 分钟）。

构建成功后会获得一个默认域名：`https://my-site.pages.dev`

---

## 五、自定义域名（可选）

1. 进入项目 → **Custom domains** 标签页
2. 点击 **Set up a custom domain**
3. 输入你的域名（如 `blog.example.com`）
4. 按提示修改 DNS 记录：
   - 如果域名已托管在 Cloudflare：自动配置
   - 如果域名在其他服务商：添加 CNAME 记录指向 `my-site.pages.dev`
5. 等待 DNS 生效（通常几分钟到 48 小时）

---

## 六、验证部署

### 6.1 基础检查

- 访问 `https://my-site.pages.dev`，页面正常加载
- 检查 HTTPS 证书是否有效（浏览器地址栏显示锁图标）
- 移动端访问测试

### 6.2 自动部署验证

```bash
# 本地修改后推送
echo "test" >> README.md
git add .
git commit -m "test: 验证自动部署"
git push
```

回到 Cloudflare Dashboard → **Deployments** 标签页，确认：
- 新的部署已自动触发
- 构建状态为 ✅ Success
- 站点内容已更新

### 6.3 常见问题排查

| 问题 | 解决方案 |
|------|---------|
| 构建失败 | 检查构建日志，确认构建命令和输出目录正确 |
| 页面 404 | 确认输出目录配置正确；SPA 项目需添加 `_redirects` 文件 |
| 自定义域名不生效 | 清除 DNS 缓存，等待 TTL 过期 |
| 构建超时 | 优化构建流程，或检查是否超出免费额度限制 |

> 📝 **注意**
> SPA 项目（如 React/Vue）需要在输出目录添加 `_redirects` 文件：
> ```
> /* /index.html 200
> ```

---

## 七、后续维护

- **更新内容**：直接 `git push` 到 `main` 分支即可自动重新部署
- **预览环境**：推送到非 `main` 分支会生成独立的预览 URL
- **回滚**：在 Deployments 页面可一键回滚到历史版本
- **监控**：Cloudflare 自带 Analytics，可查看访问量和性能数据
