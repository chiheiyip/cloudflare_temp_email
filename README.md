<!-- markdownlint-disable-file MD033 MD045 -->
# Cloudflare 临时邮箱 - 免费搭建临时邮件服务

<p align="center">
  <a href="https://temp-mail-docs.awsl.uk" target="_blank">
    <img alt="docs" src="https://img.shields.io/badge/docs-grey?logo=vitepress">
  </a>
  <a href="https://github.com/dreamhunter2333/cloudflare_temp_email/releases/latest" target="_blank">
    <img src="https://img.shields.io/github/v/release/dreamhunter2333/cloudflare_temp_email">
  </a>
  <a href="https://github.com/dreamhunter2333/cloudflare_temp_email/blob/main/LICENSE" target="_blank">
    <img alt="MIT License" src="https://img.shields.io/github/license/dreamhunter2333/cloudflare_temp_email">
  </a>
  <a href="https://github.com/dreamhunter2333/cloudflare_temp_email/graphs/contributors" target="_blank">
   <img alt="GitHub contributors" src="https://img.shields.io/github/contributors/dreamhunter2333/cloudflare_temp_email">
  </a>
  <a href="">
    <img alt="GitHub top language" src="https://img.shields.io/github/languages/top/dreamhunter2333/cloudflare_temp_email">
  </a>
  <a href="">
    <img src="https://img.shields.io/github/last-commit/dreamhunter2333/cloudflare_temp_email">
  </a>
</p>

<p align="center">
  <a href="https://hellogithub.com/repository/2ccc64bb1ba346b480625f584aa19eb1" target="_blank">
    <img src="https://abroad.hellogithub.com/v1/widgets/recommend.svg?rid=2ccc64bb1ba346b480625f584aa19eb1&claim_uid=FxNypXK7UQ9OECT" alt="Featured｜HelloGitHub" height="30"/>
  </a>
</p>

<p align="center">
  <a href="README.md">中文文档</a> |
  <a href="README_EN.md">English Document</a> |
  <a href="README_JA.md">日本語ドキュメント</a>
</p>

> 本项目仅供学习和个人用途，请勿将其用于任何违法行为，否则后果自负。

**一个功能完整的临时邮箱服务！**

- **完全免费** - 基于 Cloudflare 免费服务构建，零成本运行
- **高性能** - Rust WASM 邮件解析，响应速度极快
- **现代化界面** - 响应式设计，支持多语言，操作简便
- **地址密码** - 支持为邮箱地址设置独立密码，增强安全性
- **Agent 友好** - 内置邮箱 [`skill`](skills/cf-temp-mail-agent-mail/SKILL.md)，方便 AI agent 使用邮箱
- **移动端管理** - 社区客户端 [CloudMail](https://github.com/Lur1N77777/CloudMail)，支持 Android 管理后台和邮箱管理

## 部署文档 — 从零到上线（一步不跳）

> 💡 也可以看详细文档：[temp-mail-docs.awsl.uk](https://temp-mail-docs.awsl.uk) | [Github Action 部署文档](https://temp-mail-docs.awsl.uk/zh/guide/actions/github-action.html)

<a href="https://temp-mail-docs.awsl.uk/zh/guide/actions/github-action.html">
  <img src="https://deploy.workers.cloudflare.com/button" alt="Deploy to Cloudflare Workers" height="32">
</a>

### 第 0 步：你需要什么

- 一个 [Cloudflare 账号](https://dash.cloudflare.com/)（免费）
- 一个你自己的域名，DNS 托管在 Cloudflare（如 `mail.mydomain.com`）
- 一个 [GitHub 账号](https://github.com/)（免费）

---

### 第 1 步：Fork 仓库

点击右上角 **Fork**，把这个仓库 fork 到你自己的 GitHub 账号下。

---

### 第 2 步：在 Cloudflare 创建资源

打开 [Cloudflare Dashboard](https://dash.cloudflare.com/)，创建以下三个资源：

#### 2a. 创建 D1 数据库

左侧菜单 → **Workers & Pages** → **D1** → **创建数据库**

- 数据库名称：`temp-email-db`（可以随意取名）
- 创建后记下 **数据库 ID**（一串 UUID）

#### 2b. 创建 KV 命名空间

左侧菜单 → **Workers & Pages** → **KV** → **创建命名空间**

- 命名空间名称：`temp-email-kv`（可以随意取名）
- 创建后记下 **命名空间 ID**（一串 UUID）

#### 2c. 创建 API Token

右上角我的头像 → **我的个人资料** → **API 令牌** → **创建令牌**

选择「创建自定义令牌」，权限设置：
- **Account / Workers KV Storage**：编辑
- **Account / D1**：编辑
- **Account / Workers Scripts**：编辑

记下生成的 **API Token**。

#### 2d. 找到账户 ID

Cloudflare Dashboard 首页右下角，或者 URL 里 `dash.cloudflare.com/` 后面那串就是 **账户 ID**。

---

### 第 3 步：配置 GitHub Secrets

打开你 fork 后的仓库 → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**，逐一添加以下 4 个 Secrets：

| Secret 名称 | 如何获取 |
|---|---|
| `CLOUDFLARE_ACCOUNT_ID` | 第 2d 步记下的账户 ID 填入 |
| `CLOUDFLARE_API_TOKEN` | 第 2c 步记下的 API Token 填入 |
| `USE_WORKER_ASSETS` | 填入 `true` |
| `BACKEND_TOML` | 见下方模板 ⬇️ |

#### `BACKEND_TOML` 模板

把下面内容中所有 `***` 替换成你的实际值，然后整段复制粘贴到 Secret 里：

```toml
name = "temp-email"
main = "src/worker.ts"
compatibility_date = "2025-07-01"
compatibility_flags = ["nodejs_compat"]

[assets]
directory = "../frontend/dist/"
binding = "ASSETS"
run_worker_first = true

[[d1_databases]]
binding = "DB"
database_name = "***"
database_id = "***"

[[kv_namespaces]]
binding = "KV"
id = "***"

[vars]
DOMAIN = "***"
DOMAINS = ["***"]
DEFAULT_DOMAINS = ["***"]
JWT_SECRET = "***"
DEFAULT_LANG = "zh-CN"

[[routes]]
pattern = "***/*"
zone_name = "***"
```

**字段说明**：

| 字段 | 说明 | 示例 |
|---|---|---|
| `database_name` | D1 数据库名称 | `temp-email-db` |
| `database_id` | D1 数据库 UUID | `85872f53-...` |
| `KV id` | KV 命名空间 UUID | `158b8e4e...` |
| `DOMAIN` | 你的主域名 | `mail.mydomain.com` |
| `DOMAINS` | 域名列表（用于邮箱地址生成） | `["mail.mydomain.com"]` |
| `DEFAULT_DOMAINS` | 默认域名列表 | `["mail.mydomain.com"]` |
| `JWT_SECRET` | 随机密钥（建议 32 位以上） | 随机生成 |
| `pattern` | Worker 路由匹配 | `mail.mydomain.com/*` |
| `zone_name` | 你的 Cloudflare 区域名 | `mydomain.com` |

> ⚠️ **务必不要漏掉 `[assets]` 配置块**（第 6-9 行）。没有它，部署后访问域名只显示 `ok` 两个字，看不到前端页面。

---

### 第 4 步：触发部署

打开仓库 → **Actions** → 左侧选择 **Deploy Backend** → 右侧 **Run workflow** → Branch 选 `main` → **Run workflow**。

然后等它跑完（通常 2-3 分钟）。

---

### 第 5 步：验证

浏览器访问你的域名（如 `https://mail.mydomain.com`）。

✅ **成功**：看到完整的前端登录页面
❌ **失败**：只显示 `ok` 两个字 → **回到第 3 步，检查 `BACKEND_TOML` 有没有 `[assets]` 配置块，检查 `USE_WORKER_ASSETS` 是否是 `true`**

---

### ⚠️ 常见踩坑

<details>
<summary>展开查看常见问题</summary>

**部署后只显示 `ok`？**
> `BACKEND_TOML` 里没有 `[assets]` 配置块，或者 `USE_WORKER_ASSETS` 不是 `true`。

**应该跑哪个 Workflow？**
> 新手只跑 `Deploy Backend`。`Deploy Frontend` 和 `Deploy Frontend with page function` 都需要额外配置 Cloudflare Pages，暂不需要。

**Workflow 报 `fatal: couldn't find remote ref`？**
> 如果你通过 push tag 触发 workflow 后立即删了 tag，Actions 可能还没 fetch 到。等跑完再删。

**后续前端更新怎么部署？**
> 改完前端代码后，重新跑 `Deploy Backend` workflow 即可——它会自动构建前端并跟 Worker 一起部署。

</details>

---

## 更新日志

查看 [CHANGELOG](CHANGELOG.md) 了解最新更新内容。

## 在线体验

立即体验 → [https://mail.awsl.uk/](https://mail.awsl.uk/)

<details>
<summary>服务状态监控（点击收缩/展开）</summary>

|                                            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [Backend](https://temp-email-api.awsl.uk/) | [![Deploy Backend Production](https://github.com/dreamhunter2333/cloudflare_temp_email/actions/workflows/backend_deploy.yaml/badge.svg)](https://github.com/dreamhunter2333/cloudflare_temp_email/actions/workflows/backend_deploy.yaml) ![](https://uptime.aks.awsl.icu/api/badge/10/status) ![](https://uptime.aks.awsl.icu/api/badge/10/uptime) ![](https://uptime.aks.awsl.icu/api/badge/10/ping) ![](https://uptime.aks.awsl.icu/api/badge/10/avg-response) ![](https://uptime.aks.awsl.icu/api/badge/10/cert-exp) ![](https://uptime.aks.awsl.icu/api/badge/10/response) |
| [Frontend](https://mail.awsl.uk/)          | [![Deploy Frontend](https://github.com/dreamhunter2333/cloudflare_temp_email/actions/workflows/frontend_deploy.yaml/badge.svg)](https://github.com/dreamhunter2333/cloudflare_temp_email/actions/workflows/frontend_deploy.yaml) ![](https://uptime.aks.awsl.icu/api/badge/12/status) ![](https://uptime.aks.awsl.icu/api/badge/12/uptime) ![](https://uptime.aks.awsl.icu/api/badge/12/ping) ![](https://uptime.aks.awsl.icu/api/badge/12/avg-response) ![](https://uptime.aks.awsl.icu/api/badge/12/cert-exp) ![](https://uptime.aks.awsl.icu/api/badge/12/response)         |

</details>

<details>
<summary>Star History（点击收缩/展开）</summary>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=dreamhunter2333/cloudflare_temp_email&type=Date&theme=dark" />
  <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=dreamhunter2333/cloudflare_temp_email&type=Date" />
  <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=dreamhunter2333/cloudflare_temp_email&type=Date" />
</picture>

</details>

<details open>
<summary>目录（点击收缩/展开）</summary>

- [Cloudflare 临时邮箱 - 免费搭建临时邮件服务](#cloudflare-临时邮箱---免费搭建临时邮件服务)
  - [部署文档 — 从零到上线（一步不跳）](#部署文档--从零到上线一步不跳)
  - [更新日志](#更新日志)
  - [在线体验](#在线体验)
  - [核心功能](#核心功能)
    - [邮件处理](#邮件处理)
    - [用户管理](#用户管理)
    - [管理功能](#管理功能)
    - [多语言与界面](#多语言与界面)
    - [集成与扩展](#集成与扩展)
  - [技术架构](#技术架构)
    - [系统架构](#系统架构)
    - [技术栈](#技术栈)
    - [主要组件](#主要组件)
  - [常见踩坑](#常见踩坑)
  - [加入社区](#加入社区)

</details>

## 核心功能

<details open>
<summary>核心功能详情（点击收缩/展开）</summary>

### 邮件处理

- [x] 使用 `rust wasm` 解析邮件，解析速度快，几乎所有邮件都能解析，node 的解析模块解析邮件失败的邮件，rust wasm 也能解析成功
- [x] **AI 邮件识别** - 使用 Cloudflare Workers AI 自动提取邮件中的验证码、认证链接、服务链接等重要信息
- [x] 支持为指定基础域名创建随机二级域名邮箱地址，更适合收件隔离场景
- [x] 支持发送邮件，支持 `DKIM` 验证
- [x] 支持 `SMTP` 和 `Resend` 等多种发送方式 
- [x] 增加查看 `附件` 功能，支持附件图片显示
- [x] 支持 S3 附件存储和删除功能
- [x] 垃圾邮件检测和黑白名单配置
- [x] 邮件转发功能，支持全局转发地址

### 用户管理

- [x] 使用 `凭证` 重新登录之前的邮箱
- [x] 添加完整的用户注册登录功能，可绑定邮箱地址，绑定后可自动获取邮箱JWT凭证切换不同邮箱
- [x] 支持 `OAuth2` 第三方登录（Github、Authentik 等）
- [x] 支持 `Passkey` 无密码登录
- [x] 用户角色管理，支持多角色域名和前缀配置
- [x] 用户收件箱查看，支持地址和关键词过滤

### 管理功能

- [x] 完整的 admin 控制台
- [x] `admin` 后台创建无前缀邮箱
- [x] admin 用户管理页面，增加用户地址查看功能
- [x] 定时清理功能，支持多种清理策略
- [x] 获取自定义名字的邮箱，`admin` 可配置黑名单
- [x] 增加访问密码，可作为私人站点

### 多语言与界面

- [x] 前后台均支持多语言
- [x] 现代化 UI 设计，支持响应式布局
- [x] 支持 Google Ads 集成
- [x] 使用 shadow DOM 防止样式污染
- [x] 支持 URL JWT 参数自动登录

### 集成与扩展

- [x] 完整的 `Telegram Bot` 支持，以及 `Telegram` 推送，Telegram Bot 小程序
- [x] 添加 `SMTP proxy server`，支持 `SMTP` 发送邮件，`IMAP` 查看邮件
- [x] Webhook 支持，消息推送集成
- [x] 支持 `CF Turnstile` 人机验证
- [x] 限流配置，防止滥用
- [x] **Agent 友好**：内置 [`cf-temp-mail-agent-mail`](skills/cf-temp-mail-agent-mail/SKILL.md) skill，AI agent 可直接消费邮箱，详见 [文档](vitepress-docs/docs/zh/guide/feature/agent-email.md)
- [x] 社区移动端管理客户端：[CloudMail](https://github.com/Lur1N77777/CloudMail) 基于 Expo / React Native，面向本项目兼容 API，提供 Android 管理员后台、地址管理、收件/发件/未知邮件、验证码快捷复制、OLED 黑主题和本地分组。

</details>

## 技术架构

<details>
<summary>技术架构详情（点击收缩/展开）</summary>

### 系统架构

- **数据库**: Cloudflare D1 作为主数据库
- **前端部署**: 使用 Cloudflare Pages 部署前端
- **后端部署**: 使用 Cloudflare Workers 部署后端
- **邮件转发**: 使用 Cloudflare Email Routing

### 技术栈

- **前端**: Vue 3 + Vite + TypeScript
- **后端**: TypeScript + Cloudflare Workers
- **邮件解析**: Rust WASM (mail-parser-wasm)
- **数据库**: Cloudflare D1 (SQLite)
- **存储**: Cloudflare KV + R2 (可选 S3)
- **代理服务**: Python SMTP/IMAP Proxy Server

### 主要组件

- **Worker**: 核心后端服务
- **Frontend**: Vue 3 用户界面
- **Mail Parser WASM**: Rust 邮件解析模块
- **SMTP Proxy Server**: Python 邮件代理服务
- **Pages Functions**: Cloudflare Pages 中间件
- **Documentation**: VitePress 文档站点

</details>

### 提醒

- 在Resend添加域名记录时，如果您域名解析服务商正在托管您的3级域名a.b.com，请删除Resend生成的默认name中二级域名前缀b，否则将会添加a.b.b.com，导致验证失败。添加记录后，可通过
```bash
nslookup -qt="mx" a.b.com 1.1.1.1
```
进行验证。 

## 常见问题（FAQ）

### 部署后访问域名只显示 `ok`，看不到前端页面

**根因**：`wrangler.toml` 缺少 `[assets]` 配置，Worker 不知道要 serve 前端静态文件。

**解决**：在仓库 Secrets → Actions 的 `BACKEND_TOML` 中确保包含以下 `[assets]` 配置块：

```toml
[assets]
directory = "../frontend/dist/"
binding = "ASSETS"
run_worker_first = true
```

同时确保 `USE_WORKER_ASSETS` secret 设为 `true`。

### 应该跑哪个 Workflow？

- ✅ **新手推荐**：`Deploy Backend` — 自动构建前端 + 部署 Worker，Worker 自己 serve 完整页面
- ❌ `Deploy Frontend` — 推送到 Cloudflare Pages，需要额外配置 Pages 项目 + Pages 权限
- ❌ `Deploy Frontend with page function` — 使用 Pages Functions，同样需要 Pages 相关配置

### 部署后看不到变化？

1. 确认 `USE_WORKER_ASSETS` = `true`
2. 确认 `BACKEND_TOML` 中有 `[assets]` 配置块
3. 确认跑的是 `Deploy Backend` workflow（不是前端 workflow）
4. 等 workflow 完全跑完再刷新页面

### 为什么 Workflow 报 `fatal: couldn't find remote ref`？

如果你通过 push tag 触发 workflow 后立即删除了 tag，GitHub Actions runner 可能还没 fetch 到 tag。等 workflow 跑完再删 tag。

## 加入社区

- [Telegram](https://t.me/cloudflare_temp_email)
