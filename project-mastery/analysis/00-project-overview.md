# 项目概览与技术栈分析 | Project Overview & Tech Stack Analysis

## 📊 项目基本信息 | Basic Information

**项目名称 | Project Name:** Signalist - Stock Tracker App
**项目类型 | Project Type:** 全栈Web应用 | Full-Stack Web Application
**开发框架 | Framework:** Next.js 15 (App Router)
**语言 | Language:** TypeScript 5
**架构模式 | Architecture Pattern:** Server Components + Server Actions (Next.js 15 Pattern)

## 🎯 项目简介 | Project Description

这是一个AI驱动的现代化股票市场追踪应用，具有以下核心功能：
- 实时股票价格追踪与交互式图表
- 个性化监控列表与价格预警
- 公司财务数据与市场新闻
- AI驱动的市场摘要与邮件通知
- 事件驱动的自动化工作流

This is an AI-powered modern stock market tracking application with:
- Real-time stock price tracking with interactive charts
- Personalized watchlists and price alerts
- Company financial data and market news
- AI-driven market summaries and email notifications
- Event-driven automated workflows

## 🛠️ 完整技术栈清单 | Complete Tech Stack

### 核心框架 | Core Framework

| 技术 | 版本 | 作用 | 代码位置 |
|------|------|------|----------|
| Next.js | 15.5.2 | React全栈框架，提供SSR/SSG/API路由 | package.json:29 |
| React | 19.1.0 | UI组件库 | package.json:32 |
| TypeScript | 5 | 类型安全的JavaScript | package.json:52 |
| Node.js | ≥20 | 运行时环境 | package.json:42 |

### 数据库 | Database

| 技术 | 版本 | 作用 | 代码位置 |
|------|------|------|----------|
| MongoDB | 6.19.0 | NoSQL数据库驱动 | package.json:27 |
| Mongoose | 8.18.0 | MongoDB ODM (对象文档映射) | package.json:28, database/mongoose.ts |

**数据库连接配置文件:**
- `database/mongoose.ts:1-37` - 连接管理与缓存

### 认证系统 | Authentication

| 技术 | 版本 | 作用 | 代码位置 |
|------|------|------|----------|
| Better Auth | 1.3.7 | 现代化认证库 (邮箱密码登录) | package.json:20, lib/better-auth/auth.ts |

**认证配置:**
- 邮箱密码登录 (lib/better-auth/auth.ts:20-27)
- 自动登录功能 (lib/better-auth/auth.ts:26)
- 密码长度: 8-128字符 (lib/better-auth/auth.ts:24-25)
- MongoDB适配器集成 (lib/better-auth/auth.ts:17)

### UI组件库 | UI Components

| 技术 | 版本 | 作用 | 代码位置 |
|------|------|------|----------|
| Shadcn UI (Radix UI) | 多个版本 | 无头UI组件库 | components/ui/* |
| @radix-ui/react-avatar | 1.1.10 | 头像组件 | package.json:13 |
| @radix-ui/react-dialog | 1.1.15 | 对话框组件 | package.json:14 |
| @radix-ui/react-dropdown-menu | 2.1.16 | 下拉菜单 | package.json:15 |
| @radix-ui/react-label | 2.1.7 | 标签组件 | package.json:16 |
| @radix-ui/react-popover | 1.1.15 | 弹出层 | package.json:17 |
| @radix-ui/react-select | 2.2.6 | 选择框 | package.json:18 |
| @radix-ui/react-slot | 1.2.3 | 插槽组件 | package.json:19 |
| cmdk | 1.1.1 | 命令菜单 (搜索功能) | package.json:23 |
| Lucide React | 0.542.0 | 图标库 | package.json:26 |

### 样式系统 | Styling

| 技术 | 版本 | 作用 | 代码位置 |
|------|------|------|----------|
| Tailwind CSS | 4 | 实用优先的CSS框架 | package.json:49 |
| @tailwindcss/postcss | 4 | PostCSS插件 | package.json:41 |
| tw-animate-css | 1.3.7 | Tailwind动画 | package.json:51 |
| class-variance-authority | 0.7.1 | 条件类名工具 | package.json:21 |
| clsx | 2.1.1 | 类名合并工具 | package.json:22 |
| tailwind-merge | 3.3.1 | Tailwind类名优化 | package.json:37 |
| next-themes | 0.4.6 | 主题切换 (暗色模式) | package.json:30 |

**默认主题:** Dark Mode (app/layout.tsx:27)

### 表单处理 | Form Handling

| 技术 | 版本 | 作用 | 代码位置 |
|------|------|------|----------|
| React Hook Form | 7.62.0 | 表单状态管理 | package.json:34 |
| react-select-country-list | 2.2.3 | 国家选择器 | package.json:35 |

### 通知系统 | Notifications

| 技术 | 版本 | 作用 | 代码位置 |
|------|------|------|----------|
| Sonner | 2.0.7 | Toast通知组件 | package.json:36, app/layout.tsx:32 |

### 事件驱动工作流 | Event-Driven Workflows

| 技术 | 版本 | 作用 | 代码位置 |
|------|------|------|----------|
| Inngest | 3.40.1 | 事件驱动工作流平台 | package.json:25, lib/inngest/* |

**Inngest功能:**
- 用户注册欢迎邮件 (lib/inngest/functions.ts:9-49)
- 每日市场新闻摘要 (lib/inngest/functions.ts:51-120)
- AI内容生成集成 (Gemini API) (lib/inngest/client.ts:5)

### 邮件服务 | Email Service

| 技术 | 版本 | 作用 | 代码位置 |
|------|------|------|----------|
| Nodemailer | 7.0.6 | 邮件发送库 | package.json:31, lib/nodemailer/* |

### 外部API集成 | External APIs

| 服务 | 作用 | 代码位置 |
|------|------|----------|
| Finnhub API | 实时股票数据、财务数据、市场新闻 | lib/actions/finnhub.actions.ts |
| Gemini API | AI内容生成 (邮件摘要、个性化文本) | lib/inngest/client.ts:5 |
| TradingView Widgets | 股票图表与市场可视化 | components/TradingViewWidget.tsx |

### 开发工具 | Development Tools

| 技术 | 版本 | 作用 |
|------|------|------|
| ESLint | 9 | 代码质量检查 |
| eslint-config-next | 15.5.2 | Next.js ESLint配置 |
| ts-node | 10.9.2 | TypeScript执行器 |
| dotenv | 16.4.5 | 环境变量管理 |

## 📁 项目目录结构 | Project Structure

```
signalist_stock-tracker-app/
│
├── app/                          # Next.js 15 App Router
│   ├── (auth)/                   # 认证路由组
│   │   ├── layout.tsx            # 认证布局
│   │   ├── sign-in/page.tsx      # 登录页面
│   │   └── sign-up/page.tsx      # 注册页面
│   │
│   ├── (root)/                   # 主应用路由组
│   │   ├── layout.tsx            # 主应用布局 (含Header导航)
│   │   ├── page.tsx              # 首页 (市场概览、热力图)
│   │   └── stocks/
│   │       └── [symbol]/page.tsx # 股票详情页 (动态路由)
│   │
│   ├── api/                      # API路由
│   │   └── inngest/route.ts      # Inngest webhook端点
│   │
│   ├── layout.tsx                # 根布局 (全局样式、字体、Toaster)
│   ├── globals.css               # 全局CSS样式
│   └── favicon.ico               # 网站图标
│
├── components/                   # React组件
│   ├── ui/                       # Shadcn UI基础组件
│   │   ├── avatar.tsx
│   │   ├── button.tsx
│   │   ├── command.tsx           # 命令菜单组件
│   │   ├── dialog.tsx
│   │   ├── dropdown-menu.tsx
│   │   ├── input.tsx
│   │   ├── label.tsx
│   │   ├── popover.tsx
│   │   ├── select.tsx
│   │   └── sonner.tsx            # Toast通知
│   │
│   ├── forms/                    # 表单组件
│   │   ├── CountrySelectField.tsx
│   │   ├── InputField.tsx
│   │   ├── SelectField.tsx
│   │   └── FooterLink.tsx
│   │
│   ├── Header.tsx                # 导航头部
│   ├── NavItems.tsx              # 导航菜单项
│   ├── SearchCommand.tsx         # 股票搜索命令面板
│   ├── TradingViewWidget.tsx     # TradingView图表组件
│   ├── UserDropdown.tsx          # 用户下拉菜单
│   └── WatchlistButton.tsx       # 监控列表按钮
│
├── database/                     # 数据库层
│   ├── mongoose.ts               # MongoDB连接管理
│   └── models/
│       └── watchlist.model.ts    # 监控列表数据模型
│
├── lib/                          # 核心库与工具
│   ├── actions/                  # Next.js Server Actions
│   │   ├── auth.actions.ts       # 认证操作 (登录/注册/登出)
│   │   ├── finnhub.actions.ts    # Finnhub API调用 (搜索股票、获取新闻)
│   │   ├── user.actions.ts       # 用户操作
│   │   └── watchlist.actions.ts  # 监控列表操作
│   │
│   ├── better-auth/
│   │   └── auth.ts               # Better Auth配置
│   │
│   ├── inngest/
│   │   ├── client.ts             # Inngest客户端实例
│   │   ├── functions.ts          # Inngest工作流函数
│   │   └── prompts.ts            # AI提示词模板
│   │
│   ├── nodemailer/
│   │   ├── index.ts              # 邮件发送函数
│   │   └── templates.ts          # 邮件HTML模板
│   │
│   ├── constants.ts              # 全局常量 (股票符号、Widget配置)
│   └── utils.ts                  # 工具函数 (日期格式化、类名合并等)
│
├── hooks/                        # React自定义Hooks
│   ├── useDebounce.ts            # 防抖Hook
│   └── useTradingViewWidget.tsx  # TradingView组件Hook
│
├── middleware/
│   └── index.ts                  # Next.js中间件 (路由保护)
│
├── public/                       # 静态资源
│   ├── assets/
│   │   ├── icons/                # SVG图标
│   │   └── images/               # 图片资源
│   └── readme/                   # README相关图片
│
├── scripts/                      # 脚本工具
│   ├── test-db.ts
│   └── test-db.mjs               # 数据库连接测试脚本
│
├── types/
│   └── global.d.ts               # 全局TypeScript类型定义
│
├── components.json               # Shadcn UI配置
├── eslint.config.mjs             # ESLint配置
├── next.config.ts                # Next.js配置
├── package.json                  # NPM依赖清单
├── postcss.config.mjs            # PostCSS配置
├── tsconfig.json                 # TypeScript配置
└── README.md                     # 项目文档
```

## ⚙️ 架构模式分析 | Architecture Pattern

### 整体架构 | Overall Architecture

**模式名称:** Next.js 15 App Router + Server Actions + Server Components

**架构层次:**

```
┌─────────────────────────────────────────────────────┐
│              Client Layer (浏览器)                   │
│  ┌─────────────────────────────────────────────┐   │
│  │ React Components (app/**/*.tsx)             │   │
│  │ - Server Components (默认)                   │   │
│  │ - Client Components ('use client')          │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
                        ↕
┌─────────────────────────────────────────────────────┐
│            Server Actions Layer                      │
│  ┌─────────────────────────────────────────────┐   │
│  │ Server Actions ('use server')               │   │
│  │ - lib/actions/auth.actions.ts               │   │
│  │ - lib/actions/watchlist.actions.ts          │   │
│  │ - lib/actions/finnhub.actions.ts            │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
                        ↕
┌─────────────────────────────────────────────────────┐
│         External Services & Database                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
│  │ MongoDB  │  │ Finnhub  │  │ Inngest  │          │
│  │ (数据库) │  │  (API)   │  │ (事件)   │          │
│  └──────────┘  └──────────┘  └──────────┘          │
└─────────────────────────────────────────────────────┘
```

### 数据流模式 | Data Flow Pattern

**SSR (服务器端渲染) + Server Actions:**

1. **页面请求流程:**
   - 用户访问 `/stocks/AAPL`
   - Next.js Server渲染组件 (`app/(root)/stocks/[symbol]/page.tsx`)
   - Server Component直接调用Server Action获取数据
   - 返回完整的HTML给浏览器

2. **交互流程:**
   - 用户点击"添加到监控列表"按钮
   - 触发Server Action (`lib/actions/watchlist.actions.ts`)
   - Server Action更新MongoDB数据库
   - 返回结果，UI更新

### 认证架构 | Authentication Architecture

**中间件保护模式:**

```
请求 → middleware/index.ts (检查session cookie)
         ↓ 有cookie              ↓ 无cookie
    继续访问页面          重定向到 / (首页/登录)
```

**保护的路由:** 除了 `/sign-in`, `/sign-up`, `/api`, `/_next/*`, `/assets` 之外的所有路由
**源代码位置:** `middleware/index.ts:14-18`

## 📋 关键配置文件清单 | Key Configuration Files

| 文件名 | 作用 | 关键配置 |
|--------|------|----------|
| `next.config.ts` | Next.js配置 | - 禁用构建时ESLint检查 (行5)<br>- 禁用TypeScript构建错误 (行6-7) |
| `tsconfig.json` | TypeScript配置 | - 目标: ES2017<br>- 严格模式: true<br>- 路径别名: @/* → ./* |
| `package.json` | 依赖管理 | - 启用Turbopack (dev & build)<br>- 测试脚本: test:db |
| `components.json` | Shadcn配置 | UI组件库配置 |
| `eslint.config.mjs` | ESLint规则 | 代码质量规则 |
| `postcss.config.mjs` | PostCSS配置 | Tailwind CSS处理 |
| `.env` (需创建) | 环境变量 | 见下方环境变量清单 |

## 🔐 环境变量清单 | Environment Variables

**文件位置:** 项目根目录 `.env` (需手动创建)
**参考文档:** README.md:125-148

| 环境变量 | 作用 | 是否必需 | 示例值 |
|----------|------|----------|--------|
| `NODE_ENV` | 运行环境 | 否 | development / production |
| `NEXT_PUBLIC_BASE_URL` | 应用基础URL | 是 | http://localhost:3000 |
| **Finnhub API** |
| `NEXT_PUBLIC_FINNHUB_API_KEY` | Finnhub API密钥 (客户端可用) | 是 | xxx |
| `FINNHUB_API_KEY` | Finnhub API密钥 (服务器端) | 是 | xxx |
| `FINNHUB_BASE_URL` | Finnhub API基础URL | 否 | https://finnhub.io/api/v1 |
| **数据库** |
| `MONGODB_URI` | MongoDB连接字符串 | 是 | mongodb+srv://... |
| **认证** |
| `BETTER_AUTH_SECRET` | Better Auth密钥 | 是 | 随机字符串 |
| `BETTER_AUTH_URL` | Better Auth回调URL | 是 | http://localhost:3000 |
| **AI服务** |
| `GEMINI_API_KEY` | Google Gemini API密钥 | 是 | xxx |
| **邮件服务** |
| `NODEMAILER_EMAIL` | 发件邮箱地址 | 是 | your-email@gmail.com |
| `NODEMAILER_PASSWORD` | 邮箱密码/应用密码 | 是 | xxx |

**获取方式说明:**

1. **MongoDB:** 注册 [MongoDB Atlas](https://www.mongodb.com/products/platform/atlas-database)
2. **Finnhub:** 注册 [Finnhub](https://finnhub.io) 获取免费API密钥
3. **Gemini:** 访问 [Google AI Studio](https://aistudio.google.com/prompts/new_chat)
4. **Better Auth Secret:** 运行 `openssl rand -base64 32` 生成随机密钥
5. **Nodemailer (Gmail):** 启用两步验证后生成"应用专用密码"

## 🚀 启动流程 | Startup Process

**开发环境启动命令:**

```bash
# 1. 安装依赖
npm install

# 2. 配置环境变量 (创建.env文件)
# 参考上方环境变量清单

# 3. 启动开发服务器
npm run dev

# 4. (另一个终端) 启动Inngest开发服务器
npx inngest-cli@latest dev
```

**访问地址:**
- 应用主页: http://localhost:3000
- Inngest Dev Server: http://localhost:8288

**构建与部署:**

```bash
# 构建生产版本
npm run build

# 启动生产服务器
npm start
```

## 📊 项目规模统计 | Project Statistics

- **总文件数:** 65个TypeScript/TSX文件
- **组件数量:** 约20+个React组件
- **Server Actions:** 4个文件 (auth, finnhub, user, watchlist)
- **数据模型:** 1个 (Watchlist) + Better Auth内置表
- **API路由:** 1个 (Inngest webhook)
- **页面路由:** 4个 (首页、登录、注册、股票详情)
- **Inngest Functions:** 2个 (欢迎邮件、每日新闻摘要)

## 🔍 核心特性识别 | Core Features

### 1. 用户认证 (Authentication)
- 邮箱密码注册/登录
- Session管理 (Better Auth + MongoDB)
- 中间件路由保护

### 2. 股票搜索与追踪 (Stock Search & Tracking)
- 实时股票搜索 (Finnhub API)
- 个性化监控列表 (MongoDB存储)
- TradingView交互式图表

### 3. 市场数据展示 (Market Data Display)
- 市场概览 (Market Overview Widget)
- 股票热力图 (Heatmap Widget)
- 实时新闻时间线 (Timeline Widget)
- 市场行情 (Market Quotes Widget)

### 4. AI驱动工作流 (AI-Powered Workflows)
- 用户注册欢迎邮件 (AI生成个性化内容)
- 每日市场新闻摘要 (基于用户监控列表)
- 事件驱动架构 (Inngest)

### 5. 邮件通知系统 (Email Notifications)
- 欢迎邮件
- 每日新闻摘要邮件
- HTML邮件模板

## 🎨 设计系统 | Design System

- **字体:** Geist Sans + Geist Mono (Google Fonts)
- **主题:** 暗色模式 (默认)
- **UI风格:** 现代化、简洁、专业
- **响应式:** 移动端优先 (Tailwind CSS)
- **组件库:** Radix UI (无头组件) + Shadcn样式

## 📝 代码风格与规范 | Code Style

- **严格模式:** TypeScript strict mode启用
- **目标版本:** ES2017
- **模块系统:** ESNext
- **路径别名:** `@/*` 映射到项目根目录

---

**文档版本:** v1.0
**生成时间:** 2025-11-16
**分析文件数:** 12个核心文件
**下一步:** 进入数据库分析阶段 → `01-database-analysis.md`
