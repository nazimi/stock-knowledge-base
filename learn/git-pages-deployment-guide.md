# Git 授权与 GitHub Pages 部署流程详解

## 概述

本文档记录了将个人股市知识库网页通过 GitHub Pages 发布到互联网的完整流程，涵盖 Git 授权、代码推送、网页部署三个核心环节。

---

## 一、Git 授权流程

### 为什么需要授权？

Git 推送代码到 GitHub 时，GitHub 需要确认"你有权写入这个仓库"。所以本地 Git 必须携带你的 GitHub 身份凭证。

### 授权流程图

```
你（本地电脑）  ──推送代码──>  GitHub（远程仓库）
     ↑                              |
     |  需要证明"我是 nazimi"         |
     └──────────────────────────────┘
```

### 具体步骤拆解

#### 1. 安装 GitHub CLI (`gh`)

`gh` 是 GitHub 官方命令行工具，它帮你管理 GitHub 认证，免去手动处理 token 的麻烦。

```bash
brew install gh        # 安装工具（macOS）
gh auth login          # 登录认证
```

#### 2. `gh auth login` 做了什么？

```
终端                          GitHub 服务器
  |                               |
  |--- "我要登录" ──────────────>|
  |<-- "这是8位代码 XXXX-XXXX" ──|
  |                               |
  |  打开浏览器                   |
  |  你粘贴代码 ──> 浏览器 ──────>|
  |                               |
  |  你点 Authorize ────────────>|  ← 这一步授权 gh 访问你的 GitHub
  |                               |
  |<-- "登录成功，给你一个令牌" ──|
  |                               |
  |  令牌保存在本地               |
  |  路径：~/.config/gh/hosts.yml |
```

核心是：GitHub 返回一个 **Personal Access Token（令牌）**，`gh` 把它保存在本地。这个令牌就是你的"身份证"。

#### 3. `gh auth setup-git` 做了什么？

这一步把 `gh` 的令牌**桥接给 Git**，让 Git 推送代码时自动使用这个令牌。它实际上是配置了 Git 的凭证助手：

```
Git → 调用 gh 作为凭证助手 → gh 提供保存的令牌 → GitHub 验证通过 → 允许推送
```

所以之后 `git push` 时你不需要再输入密码，Git 会自动通过 `gh` 获取凭证。

---

## 二、Git 推送流程

### 本地仓库 → 远程仓库

```
本地仓库                        远程仓库
(knowledge-base/)               (github.com/nazimi/stock-knowledge-base)

git init          →  创建 .git 目录，初始化本地版本库
git add .         →  把所有文件标记为"待提交"
git commit -m ""  →  生成一个提交快照（commit）
git remote add    →  告诉 Git 远程仓库地址
git push          →  把本地 commit 推送到远程
```

### 每次推送的本质

```
commit f82bc67  "初始化个人股市知识库"      ← 第一次推送
commit 1a64d71  "调整目录结构适配 Pages"   ← 第二次推送
```

GitHub 收到后，仓库就更新为最新 commit 的状态。

---

## 三、GitHub Pages 发布流程

### GitHub Pages 是什么？

GitHub Pages 是 GitHub 内置的**静态网站托管服务**。它的工作原理：

```
你的仓库文件          GitHub Pages 服务              用户浏览器
                                       
index.html  ─┐                        ┌─> https://nazimi.github.io/...
.nojekyll   ─┤  ──GitHub 服务器读取──> │
CSS (内联)  ─┤    部署为静态网站        │
JS  (内联)  ─┘                        └─> 用户访问时直接返回 HTML
```

### 关键配置项

| 配置 | 作用 |
|------|------|
| **Source: Deploy from a branch** | 告诉 Pages 从哪个分支读取文件 |
| **Branch: main** | 从 `main` 分支读取 |
| **Folder: /root** | 从仓库根目录读取 `index.html` |

### `.nojekyll` 的作用

默认情况下 GitHub Pages 用 **Jekyll**（一个静态网站生成器）处理文件。Jekyll 会忽略下划线开头的文件/目录（如 `_data`、`_includes`）。`.nojekyll` 是一个空文件，它的存在告诉 GitHub：

> **跳过 Jekyll 处理，原样发布所有文件。**

我们的网页是纯 HTML，不需要 Jekyll 处理，所以放了这个文件。

### 发布后的 URL 规则

```
https://<用户名>.github.io/<仓库名>/
    ↓              ↓
  nazimi    stock-knowledge-base

→ https://nazimi.github.io/stock-knowledge-base/
```

> **特殊规则**：如果你用 `<用户名>.github.io` 作为仓库名（即 `nazimi.github.io`），则直接访问 `https://nazimi.github.io/`，不需要仓库名后缀。

### GitHub Pages 目录选择限制

GitHub Pages 的 Branch 源只支持两种文件夹选择：

| 选项 | 说明 |
|------|------|
| **/root** | 从仓库根目录读取 `index.html` |
| **/docs** | 从仓库 `/docs` 目录读取 `index.html` |

**不支持** `/src` 或其他任意子目录。因此我们将 `index.html` 从 `src/` 移到了仓库根目录，选择 `/root` 部署。

---

## 四、后续更新流程

以后每日更新内容时，流程非常简单：

```
Agent 修改 index.html（更新研报/推荐数据的 JSON）
    ↓
git add index.html
    ↓
git commit -m "每日更新 2026-08-31"
    ↓
git push
    ↓
GitHub Pages 自动重新部署（1-2分钟）
    ↓
网页自动更新 ✅
```

**不需要手动触发部署**，GitHub Pages 检测到 `main` 分支有新 commit 就会自动重建。

---

## 五、自定义域名（可选）

如果你想用 `www.yourdomain.com` 而不是 `nazimi.github.io/stock-knowledge-base/`：

```
你的域名 DNS                          GitHub Pages
                                       
添加 CNAME 记录                       仓库设置 → Custom domain
www → nazimi.github.io     <────────  填入 www.yourdomain.com
                                        ↓
                                  GitHub 自动关联
                                  HTTPS 证书自动签发（Let's Encrypt）
```

---

## 六、整体流程图

```
本地编辑                          GitHub 云端                        访客
                                                          
 index.html  ──git push──>  仓库 main 分支  ──Pages读取──>  网页
 (含内联CSS/JS)             + .nojekyll                    
 (含内嵌JSON数据)           + index.html 在根目录           
                                                          
 gh 认证令牌  <──gh auth──  验证身份通过                    
                                                          
后续更新：只改 index.html → git push → 自动发布            
```

---

## 附：常用 Git 命令速查

| 命令 | 作用 |
|------|------|
| `git status` | 查看当前工作区状态 |
| `git add .` | 添加所有修改到暂存区 |
| `git commit -m "说明"` | 提交到本地仓库 |
| `git push` | 推送到远程仓库 |
| `git pull` | 拉取远程最新代码到本地 |
| `git log --oneline` | 查看提交历史 |
| `git diff` | 查看未暂存的修改内容 |
| `gh auth status` | 检查 GitHub 认证状态 |
