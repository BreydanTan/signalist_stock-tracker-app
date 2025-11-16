# Signalist 股票追踪应用 - 技术规格文档

## 📋 项目概览

**项目名称：** Signalist Stock Tracker App
**项目类型：** AI驱动的股票市场追踪全栈Web应用
**技术栈：** Next.js 15 + React 19 + TypeScript + MongoDB + Tailwind CSS

### 主要功能

1. **实时股票追踪** - 通过TradingView图表查看实时价格和历史数据
2. **智能搜索** - 快速搜索股票（Cmd/Ctrl+K快捷键）
3. **个人监控列表** - 创建和管理关注的股票列表
4. **公司深度分析** - 查看财务数据、技术分析、公司简介
5. **AI驱动通知** - 个性化欢迎邮件和每日市场新闻摘要
6. **事件驱动架构** - 自动化工作流处理异步任务

---

## 🏗️ 系统架构

### 整体架构图

```
┌─────────────────────────────────────────────────┐
│              用户浏览器 (Browser)                 │
│  ┌──────────────────────────────────────────┐   │
│  │ React 19 UI (Server + Client Components) │   │
│  └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
                        ↕ HTTPS
┌─────────────────────────────────────────────────┐
│          Next.js 15 App Router (Vercel)          │
│  ┌────────────┐  ┌──────────────┐               │
│  │ Server     │  │ API Routes   │               │
│  │ Components │  │ /api/inngest │               │
│  │ + Actions  │  └──────────────┘               │
│  └────────────┘                                  │
└─────────────────────────────────────────────────┘
        ↕                ↕               ↕
┌─────────────┐  ┌──────────────┐  ┌──────────┐
│   MongoDB   │  │   Finnhub    │  │ Inngest  │
│  (数据库)   │  │   (股票API)  │  │ (事件)   │
└─────────────┘  └──────────────┘  └──────────┘
                        ↕               ↕
                ┌──────────────┐  ┌──────────┐
                │  TradingView │  │  Gemini  │
                │   (图表)     │  │   (AI)   │
                └──────────────┘  └──────────┘
```

### 技术栈详解

| 层次 | 技术 | 版本 | 作用 |
|------|------|------|------|
| **前端** | Next.js | 15.5.2 | React全栈框架，SSR/SSG |
| | React | 19.1.0 | UI组件库 |
| | TypeScript | 5 | 类型安全 |
| | Tailwind CSS | 4 | CSS框架 |
| | Shadcn UI | - | UI组件库 |
| **后端** | Next.js API Routes | 15.5.2 | API端点 |
| | Server Actions | - | 服务器端函数 |
| **数据库** | MongoDB | 6.19.0 | NoSQL数据库 |
| | Mongoose | 8.18.0 | ODM框架 |
| **认证** | Better Auth | 1.3.7 | 邮箱密码认证 |
| **事件驱动** | Inngest | 3.40.1 | 工作流平台 |
| **外部API** | Finnhub | - | 股票数据 |
| | TradingView | - | 图表组件 |
| | Gemini | - | AI内容生成 |
| **邮件** | Nodemailer | 7.0.6 | 邮件发送 |

---

## 📊 数据库设计

### ER关系图

```
┌─────────────────────┐
│  user (Better Auth) │
│─────────────────────│
│ id: String (PK)     │───┐
│ email: String (UK)  │   │ 1对多关系
│ name: String        │   │ (One-to-Many)
│ emailVerified: Bool │   │
│ createdAt: Date     │   │
│ updatedAt: Date     │   │
└─────────────────────┘   │
                          │
                          │
┌─────────────────────┐   │
│   watchlists        │   │
│─────────────────────│   │
│ _id: ObjectId (PK)  │   │
│ userId: String (FK) │───┘
│ symbol: String      │
│ company: String     │
│ addedAt: Date       │
│                     │
│ UNIQUE(userId,      │
│        symbol)      │
└─────────────────────┘
```

### 数据模型详解

#### user集合（Better Auth自动创建）

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| id | String | ✅ | Better Auth生成的用户ID |
| email | String | ✅ | 用户邮箱（唯一） |
| name | String | ✅ | 用户全名 |
| emailVerified | Boolean | ❌ | 邮箱验证状态 |
| createdAt | Date | ✅ | 创建时间 |
| updatedAt | Date | ✅ | 更新时间 |

**示例文档：**
```json
{
  "id": "user_cm3kx9y8z0001",
  "email": "john@example.com",
  "name": "John Doe",
  "emailVerified": false,
  "createdAt": "2025-11-16T08:00:00Z",
  "updatedAt": "2025-11-16T08:00:00Z"
}
```

#### watchlists集合

| 字段 | 类型 | 必需 | 约束 | 说明 |
|------|------|------|------|------|
| _id | ObjectId | ✅ | 自动生成 | MongoDB唯一ID |
| userId | String | ✅ | 索引 | 关联user.id |
| symbol | String | ✅ | uppercase, trim | 股票代码 |
| company | String | ✅ | trim | 公司名称 |
| addedAt | Date | ✅ | default:Date.now | 添加时间 |

**复合唯一索引：** `{ userId: 1, symbol: 1 }`
**作用：** 防止用户重复添加相同股票

**示例文档：**
```json
{
  "_id": "ObjectId('673d5f8a1234567890abcdef')",
  "userId": "user_cm3kx9y8z0001",
  "symbol": "AAPL",
  "company": "Apple Inc.",
  "addedAt": "2025-11-16T08:30:00Z"
}
```

---

## 🔄 核心业务流程

### 流程1：用户注册流程

```
┌─────────┐
│ 用户    │
│ 访问注册页│
└────┬────┘
     │
     ▼
┌─────────────────────────┐
│ 填写注册表单            │
│ - 邮箱                  │
│ - 密码（≥8字符）        │
│ - 全名                  │
│ - 国家                  │
│ - 投资目标              │
│ - 风险承受能力          │
│ - 偏好行业              │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ 客户端验证              │
│ - Email格式检查         │
│ - 密码长度检查          │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ Server Action:          │
│ signUpWithEmail()       │
│ 位置: lib/actions/      │
│       auth.actions.ts:7 │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ Better Auth API         │
│ - 创建用户账户          │
│ - 密码哈希存储          │
│ - 写入MongoDB           │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ 触发Inngest事件         │
│ 'app/user.created'      │
│ 包含用户偏好数据        │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ Inngest工作流           │
│ sendSignUpEmail()       │
│ 位置: lib/inngest/      │
│       functions.ts:9    │
└────┬────────────────────┘
     │
     ├─────────────────────▼
     │              ┌──────────────────┐
     │              │ Gemini AI生成    │
     │              │ 个性化欢迎文本   │
     │              └─────┬────────────┘
     │                    │
     │                    ▼
     │              ┌──────────────────┐
     │              │ Nodemailer发送   │
     │              │ 欢迎邮件         │
     │              └──────────────────┘
     │
     ▼
┌─────────────────────────┐
│ 返回成功                │
│ 自动登录                │
│ 重定向到首页            │
└─────────────────────────┘
```

**涉及代码位置：**
- 表单组件：`app/(auth)/sign-up/page.tsx`
- Server Action：`lib/actions/auth.actions.ts:7-23`
- Inngest函数：`lib/inngest/functions.ts:9-49`
- 邮件发送：`lib/nodemailer/index.ts:12-26`

---

### 流程2：股票搜索流程

```
┌─────────┐
│ 用户    │
│按Cmd+K │
└────┬────┘
     │
     ▼
┌─────────────────────────┐
│ 打开搜索对话框          │
│ 显示热门股票（前10个）   │
│ 来源：POPULAR_STOCK_    │
│       SYMBOLS常量        │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ 用户输入 "apple"        │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ 防抖延迟 300ms          │
│ Hook: useDebounce       │
│ 位置: hooks/            │
│       useDebounce.ts    │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ Server Action:          │
│ searchStocks("apple")   │
│ 位置: lib/actions/      │
│     finnhub.actions.ts  │
│                :101     │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ Finnhub API调用         │
│ GET /search?q=apple     │
│ 缓存：30分钟            │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ 返回搜索结果            │
│ [                       │
│   {symbol:"AAPL",       │
│    name:"Apple Inc."},  │
│   ...                   │
│ ]                       │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ 显示结果列表            │
│ 用户点击 AAPL           │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ 导航到                  │
│ /stocks/AAPL            │
└─────────────────────────┘
```

**涉及代码位置：**
- 搜索组件：`components/SearchCommand.tsx`
- 防抖Hook：`hooks/useDebounce.ts`
- Server Action：`lib/actions/finnhub.actions.ts:101-180`
- 常量定义：`lib/constants.ts:265-325`

---

### 流程3：每日新闻推送流程

```
┌──────────────────┐
│ Cron触发器       │
│ 每天UTC 12:00    │
│ (北京时间20:00)  │
└────┬─────────────┘
     │
     ▼
┌─────────────────────────┐
│ Inngest函数             │
│ sendDailyNewsSummary()  │
│ 位置: lib/inngest/      │
│       functions.ts:51   │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ Step 1: 获取所有用户    │
│ getAllUsersForNewsEmail│
│ 查询：db.collection(   │
│   'user').find({       │
│   email:{$exists:true}│
│ })                     │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ Step 2: 对每个用户      │
│ ┌─────────────────────┐ │
│ │ 2.1 获取监控列表    │ │
│ │ getWatchlistSymbols │ │
│ │ ByEmail(email)      │ │
│ │ → ["AAPL","GOOGL"] │ │
│ └─────┬───────────────┘ │
│       │                 │
│       ▼                 │
│ ┌─────────────────────┐ │
│ │ 2.2 获取股票新闻    │ │
│ │ getNews(symbols)    │ │
│ │ Finnhub API:        │ │
│ │ /company-news       │ │
│ └─────┬───────────────┘ │
│       │                 │
│       ▼                 │
│ ┌─────────────────────┐ │
│ │ 2.3 AI生成摘要      │ │
│ │ step.ai.infer()     │ │
│ │ Gemini API          │ │
│ │ → HTML格式新闻      │ │
│ └─────────────────────┘ │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ Step 3: 批量发送邮件    │
│ Promise.all([           │
│   sendNewsSummaryEmail  │
│   (user1),              │
│   sendNewsSummaryEmail  │
│   (user2),              │
│   ...                   │
│ ])                      │
└────┬────────────────────┘
     │
     ▼
┌─────────────────────────┐
│ 所有用户收到邮件        │
│ 主题：📈 Market News   │
│      Summary Today      │
└─────────────────────────┘
```

**涉及代码位置：**
- Inngest函数：`lib/inngest/functions.ts:51-120`
- 获取用户：`lib/actions/user.actions.ts:5-25`
- 获取监控列表：`lib/actions/watchlist.actions.ts:6-28`
- 获取新闻：`lib/actions/finnhub.actions.ts:25-99`
- 发送邮件：`lib/nodemailer/index.ts:28-44`

---

## 🔌 API接口文档

### 1. 用户认证API

#### 注册 (Server Action)

**函数名：** `signUpWithEmail`
**文件位置：** `lib/actions/auth.actions.ts:7-23`

**请求参数：**
```typescript
{
  email: string;              // 邮箱地址
  password: string;           // 密码（≥8字符）
  fullName: string;           // 全名
  country: string;            // 国家
  investmentGoals: string;    // 投资目标
  riskTolerance: string;      // 风险承受能力
  preferredIndustry: string;  // 偏好行业
}
```

**返回值：**
```typescript
{
  success: boolean;
  data?: object;   // Better Auth响应
  error?: string;  // 错误消息
}
```

**调用示例：**
```typescript
const result = await signUpWithEmail({
  email: "john@example.com",
  password: "SecurePass123",
  fullName: "John Doe",
  country: "United States",
  investmentGoals: "Growth",
  riskTolerance: "Medium",
  preferredIndustry: "Technology"
});

if (result.success) {
  // 注册成功，自动登录
  router.push('/');
} else {
  // 显示错误
  toast.error(result.error);
}
```

---

#### 登录 (Server Action)

**函数名：** `signInWithEmail`
**文件位置：** `lib/actions/auth.actions.ts:25-34`

**请求参数：**
```typescript
{
  email: string;     // 邮箱地址
  password: string;  // 密码
}
```

**返回值：** 同注册API

**调用示例：**
```typescript
const result = await signInWithEmail({
  email: "john@example.com",
  password: "SecurePass123"
});
```

---

### 2. 股票数据API

#### 搜索股票 (Server Action)

**函数名：** `searchStocks`
**文件位置：** `lib/actions/finnhub.actions.ts:101-180`

**请求参数：**
```typescript
query?: string;  // 搜索关键词（可选）
```

**返回值：**
```typescript
StockWithWatchlistStatus[] = [
  {
    symbol: string;        // 股票代码（大写）
    name: string;          // 公司名称
    exchange: string;      // 交易所
    type: string;          // 类型（如"Common Stock"）
    isInWatchlist: boolean;// 是否在监控列表
  }
]
```

**调用示例：**
```typescript
// 获取热门股票（无参数）
const popularStocks = await searchStocks();
// 返回前10只热门股票

// 搜索特定股票
const results = await searchStocks("apple");
// 返回匹配"apple"的前15个结果
```

---

#### 获取市场新闻 (Server Action)

**函数名：** `getNews`
**文件位置：** `lib/actions/finnhub.actions.ts:25-99`

**请求参数：**
```typescript
symbols?: string[];  // 股票代码数组（可选）
```

**返回值：**
```typescript
MarketNewsArticle[] = [
  {
    id: number;
    headline: string;    // 标题
    summary: string;     // 摘要（150-200字符）
    source: string;      // 来源
    url: string;         // 文章链接
    datetime: number;    // 时间戳
    image: string;       // 图片URL
    category: string;    // 分类
    related: string;     // 相关股票代码
  }
]
```

**调用示例：**
```typescript
// 获取通用市场新闻
const generalNews = await getNews();

// 获取特定股票新闻
const appleNews = await getNews(["AAPL", "MSFT"]);
// 返回AAPL和MSFT的新闻（round-robin算法，最多6篇）
```

---

### 3. 监控列表API

#### 获取用户监控列表 (Server Action)

**函数名：** `getWatchlistSymbolsByEmail`
**文件位置：** `lib/actions/watchlist.actions.ts:6-28`

**请求参数：**
```typescript
email: string;  // 用户邮箱
```

**返回值：**
```typescript
string[]  // 股票代码数组
```

**调用示例：**
```typescript
const symbols = await getWatchlistSymbolsByEmail("john@example.com");
// 返回: ["AAPL", "GOOGL", "MSFT"]
```

---

### 4. Inngest Webhook API

**路径：** `/api/inngest`
**方法：** `GET`, `POST`, `PUT`
**文件位置：** `app/api/inngest/route.ts`

**说明：** 这是Inngest平台的webhook端点，用于接收和处理事件。

**注册的事件：**
1. `app/user.created` - 用户注册后触发
2. `app/send.daily.news` - 手动触发每日新闻（或Cron自动触发）

**不需要手动调用**，由Inngest平台自动管理。

---

## ⚙️ 环境配置指南

### 必需的环境变量

创建 `.env.local` 文件：

```env
# 运行环境
NODE_ENV=development
NEXT_PUBLIC_BASE_URL=http://localhost:3000

# MongoDB数据库
MONGODB_URI=your-mongodb-uri-here

# Better Auth认证
BETTER_AUTH_SECRET=your-secret-min-32-characters
BETTER_AUTH_URL=http://localhost:3000

# Finnhub股票API
FINNHUB_API_KEY=your-finnhub-api-key
NEXT_PUBLIC_FINNHUB_API_KEY=your-finnhub-api-key
FINNHUB_BASE_URL=https://finnhub.io/api/v1

# Google Gemini AI
GEMINI_API_KEY=your-gemini-api-key

# Nodemailer邮件服务
NODEMAILER_EMAIL=your-email@gmail.com
NODEMAILER_PASSWORD=your-gmail-app-password
```

### 环境变量获取方式

#### 1. MongoDB URI

**方式1：MongoDB Atlas（推荐）**

1. 访问 https://www.mongodb.com/cloud/atlas
2. 注册并创建免费集群
3. 创建数据库用户
4. 配置网络访问（允许所有IP：0.0.0.0/0）
5. 获取连接字符串

**示例连接字符串：**
```
mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/signalist?retryWrites=true&w=majority
```

**方式2：本地MongoDB**
```
mongodb://localhost:27017/signalist
```

---

#### 2. Better Auth Secret

**生成方式（在终端执行）：**
```bash
# macOS/Linux
openssl rand -base64 32

# Windows (PowerShell)
[Convert]::ToBase64String((1..32|%{Get-Random -Max 256}))
```

**示例：**
```
BETTER_AUTH_SECRET=XyZ123AbC456DeF789GhI012JkL345MnO678PqR901StU234
```

---

#### 3. Finnhub API Key

1. 访问 https://finnhub.io/register
2. 注册免费账户
3. 在Dashboard获取API密钥

**免费层级限制：**
- 60 API calls/minute
- 项目使用缓存策略，实际调用频率很低

---

#### 4. Gemini API Key

1. 访问 https://aistudio.google.com/
2. 登录Google账户
3. 创建API密钥

**免费层级：**
- 每分钟15次请求
- 每天1500次请求
- 项目使用频率低于限制

---

#### 5. Gmail应用专用密码（Nodemailer）

1. 登录Gmail账户
2. 访问 https://myaccount.google.com/security
3. 启用两步验证
4. 搜索"应用专用密码"
5. 创建新密码（选择"邮件"和"其他"）
6. 复制16位密码

**配置示例：**
```env
NODEMAILER_EMAIL=yourname@gmail.com
NODEMAILER_PASSWORD=abcd efgh ijkl mnop
```

---

## 🚀 本地开发指南

### 开发环境搭建

**前置要求：**
- Node.js ≥ 20.0.0
- npm ≥ 8.0.0
- Git

**步骤1：克隆项目**
```bash
git clone https://github.com/adrianhajdin/signalist_stock-tracker-app.git
cd signalist_stock-tracker-app
```

**步骤2：安装依赖**
```bash
npm install
```

**步骤3：配置环境变量**
```bash
# 复制环境变量模板
cp .env.example .env.local

# 编辑.env.local，填写实际的API密钥
nano .env.local  # 或使用你喜欢的编辑器
```

**步骤4：启动开发服务器**
```bash
# 终端1：启动Next.js开发服务器
npm run dev

# 终端2：启动Inngest开发服务器
npx inngest-cli@latest dev
```

**访问应用：**
- 主应用：http://localhost:3000
- Inngest Dev Server：http://localhost:8288

---

### 常见开发任务

#### 添加新的数据模型

**示例：添加Alert（价格提醒）模型**

1. 创建文件 `database/models/alert.model.ts`：

```typescript
import { Schema, model, models, type Model } from 'mongoose';

export interface Alert extends Document {
  userId: string;
  symbol: string;
  alertType: 'upper' | 'lower';  // 高于/低于
  threshold: number;              // 触发价格
  createdAt: Date;
}

const AlertSchema = new Schema<Alert>({
  userId: { type: String, required: true, index: true },
  symbol: { type: String, required: true, uppercase: true },
  alertType: { type: String, enum: ['upper', 'lower'], required: true },
  threshold: { type: Number, required: true },
  createdAt: { type: Date, default: Date.now },
});

// 索引优化
AlertSchema.index({ userId: 1, symbol: 1 });

export const AlertModel: Model<Alert> =
  models?.Alert || model<Alert>('Alert', AlertSchema);
```

2. 创建Server Action `lib/actions/alert.actions.ts`：

```typescript
'use server';
import { AlertModel } from '@/database/models/alert.model';
import { connectToDatabase } from '@/database/mongoose';

export const createAlert = async (data: CreateAlertData) => {
  try {
    await connectToDatabase();
    const alert = await AlertModel.create(data);
    return { success: true, data: alert };
  } catch (error) {
    return { success: false, error: 'Failed to create alert' };
  }
}
```

---

#### 调试技巧

**1. 查看Server Action日志：**
```typescript
// 在Server Action中添加console.log
export const signUpWithEmail = async (data: SignUpFormData) => {
    console.log('Sign up attempt:', data.email);  // 查看终端输出
    // ...
}
```

**2. 查看数据库数据：**
```bash
# 使用MongoDB Compass连接数据库
# 或使用测试脚本
npm run test:db
```

**3. 调试API请求：**
```typescript
// 在浏览器Console查看网络请求
// 或在Server Action添加日志
const response = await fetch(url);
console.log('API Response:', await response.json());
```

---

## 🔧 二次开发指南

### 常见扩展场景

#### 场景1：添加价格预警功能

**需求：** 当股票价格高于/低于设定值时发送邮件通知

**开发步骤：**

1. 创建Alert数据模型（见上文）

2. 创建UI组件 `components/AlertDialog.tsx`

3. 创建Inngest函数 `lib/inngest/functions.ts`：

```typescript
export const checkPriceAlerts = inngest.createFunction(
  { id: 'check-price-alerts' },
  { cron: '*/5 * * * *' },  // 每5分钟检查一次
  async ({ step }) => {
    // 1. 获取所有活跃的alerts
    // 2. 获取这些股票的当前价格
    // 3. 检查是否触发条件
    // 4. 发送通知邮件
  }
)
```

4. 注册到Inngest路由

---

#### 场景2：添加新页面（如监控列表页）

**步骤：**

1. 创建页面文件 `app/(root)/watchlist/page.tsx`：

```typescript
import { auth } from '@/lib/better-auth/auth';
import { headers } from 'next/headers';
import { getWatchlistSymbolsByEmail } from '@/lib/actions/watchlist.actions';

export default async function WatchlistPage() {
  const session = await auth.api.getSession({ headers: await headers() });
  const symbols = await getWatchlistSymbolsByEmail(session.user.email);

  return (
    <div>
      <h1>My Watchlist</h1>
      {symbols.map(symbol => (
        <div key={symbol}>{symbol}</div>
      ))}
    </div>
  );
}
```

2. 添加导航链接 `lib/constants.ts`：

```typescript
export const NAV_ITEMS = [
    { href: '/', label: 'Dashboard' },
    { href: '/watchlist', label: 'Watchlist' },  // 新增
];
```

---

## 📈 性能优化建议

### 已实现的优化

1. **Server Components** - 减少客户端JavaScript
2. **React cache** - 同一请求周期复用数据
3. **Next.js fetch缓存** - API响应缓存
4. **防抖** - 搜索输入防抖（300ms）
5. **Lean查询** - MongoDB查询优化
6. **索引** - 数据库索引加速查询

### 可添加的优化

1. **动态导入：**
```typescript
import dynamic from 'next/dynamic';

const TradingViewWidget = dynamic(() => import('@/components/TradingViewWidget'), {
  ssr: false,
  loading: () => <div>Loading chart...</div>
});
```

2. **图片优化：**
```typescript
import Image from 'next/image';

<Image
  src="/logo.png"
  width={140}
  height={32}
  priority  // 优先加载
/>
```

3. **Redis缓存（高级）：**
```typescript
import Redis from 'ioredis';
const redis = new Redis(process.env.REDIS_URL);

// 缓存股票搜索结果
const cacheKey = `search:${query}`;
const cached = await redis.get(cacheKey);
if (cached) return JSON.parse(cached);

const results = await fetchFromAPI(query);
await redis.set(cacheKey, JSON.stringify(results), 'EX', 1800);  // 30分钟
```

---

## 🔒 安全最佳实践

### 已实现

✅ Session-based认证
✅ HttpOnly Cookies
✅ CSRF防护（SameSite）
✅ 密码哈希（Better Auth）
✅ 环境变量隔离

### 建议增强

1. **启用邮箱验证：**
```typescript
// lib/better-auth/auth.ts
emailAndPassword: {
    requireEmailVerification: true,  // 改为true
}
```

2. **添加速率限制：**
```bash
npm install @upstash/ratelimit @upstash/redis
```

```typescript
import { Ratelimit } from '@upstash/ratelimit';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s'),
});

// 在Server Action中使用
const { success } = await ratelimit.limit(userId);
if (!success) throw new Error('Rate limit exceeded');
```

3. **添加CSP头：**
```typescript
// next.config.ts
const nextConfig = {
  async headers() {
    return [{
      source: '/:path*',
      headers: [{
        key: 'Content-Security-Policy',
        value: "default-src 'self'; script-src 'self' 'unsafe-eval' https://s3.tradingview.com;"
      }]
    }];
  }
};
```

---

## 📊 部署指南

### 推荐：Vercel部署

**步骤1：准备代码**
```bash
git add .
git commit -m "Ready for deployment"
git push origin main
```

**步骤2：导入到Vercel**
1. 访问 https://vercel.com/new
2. 导入GitHub仓库
3. 配置项目

**步骤3：配置环境变量**

在Vercel Dashboard添加所有环境变量：
- `MONGODB_URI`
- `BETTER_AUTH_SECRET`
- `BETTER_AUTH_URL` = https://yourdomain.com
- `FINNHUB_API_KEY`
- `GEMINI_API_KEY`
- `NODEMAILER_EMAIL`
- `NODEMAILER_PASSWORD`

**步骤4：部署**
- Vercel自动构建和部署
- 每次push到main分支自动重新部署

**步骤5：配置Inngest**
1. 访问 https://app.inngest.com
2. 添加Event Key
3. 配置Webhook URL: `https://yourdomain.com/api/inngest`

---

## 📝 故障排查

### 常见问题

#### 问题1：数据库连接失败

**错误信息：**
```
MongoServerError: Authentication failed
```

**解决方案：**
1. 检查`MONGODB_URI`格式
2. 确认数据库用户名密码正确
3. MongoDB Atlas：检查IP白名单（允许0.0.0.0/0）
4. 测试连接：`npm run test:db`

---

#### 问题2：Inngest事件未触发

**症状：** 注册后未收到欢迎邮件

**检查清单：**
1. Inngest Dev Server是否运行（`npx inngest-cli@latest dev`）
2. 检查终端日志是否有错误
3. 访问 http://localhost:8288 查看事件日志
4. 确认`GEMINI_API_KEY`和`NODEMAILER_*`配置正确

---

#### 问题3：搜索功能无响应

**检查清单：**
1. 确认`FINNHUB_API_KEY`已配置
2. 检查浏览器Console是否有错误
3. 查看Network标签，API请求是否成功
4. Finnhub免费层级：确保未超出60次/分钟限制

---

## 🎓 学习资源

### 官方文档

- **Next.js:** https://nextjs.org/docs
- **React:** https://react.dev
- **Tailwind CSS:** https://tailwindcss.com/docs
- **Better Auth:** https://www.better-auth.com/docs
- **Inngest:** https://www.inngest.com/docs
- **MongoDB:** https://www.mongodb.com/docs
- **Finnhub:** https://finnhub.io/docs/api

### 推荐教程

- Next.js 15新特性
- Server Actions详解
- MongoDB索引优化
- Inngest事件驱动架构

---

**文档版本：** v1.0
**最后更新：** 2025-11-16
**作者：** AI技术文档生成器
**语言：** 简体中文
