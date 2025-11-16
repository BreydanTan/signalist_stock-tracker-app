# 后端架构深度分析 | Backend Architecture Deep Analysis

## 🏗️ 后端架构概览 | Backend Architecture Overview

**架构模式:** Next.js 15 App Router + Server Actions Pattern
**后端类型:** Serverless Functions (Next.js API Routes + Server Actions)
**数据获取:** Server Components + Server Actions
**事件驱动:** Inngest Workflow Platform

### 架构特点 | Architecture Characteristics

✅ **无需传统REST API:** 使用Server Actions代替API端点
✅ **类型安全:** TypeScript端到端类型检查
✅ **自动优化:** Next.js自动code-splitting和optimization
✅ **事件驱动:** Inngest处理异步任务和定时任务

---

## 📡 API端点完整清单 | Complete API Endpoints

### API Routes

项目只有 **1个API路由**:

| HTTP方法 | 路径 | 作用 | 代码位置 |
|---------|------|------|----------|
| GET, POST, PUT | `/api/inngest` | Inngest Webhook端点 | app/api/inngest/route.ts:5-8 |

#### `/api/inngest` - Inngest Webhook

**完整路径:** `http://localhost:3000/api/inngest`

**支持方法:**
- `GET` - Inngest健康检查
- `POST` - 接收Inngest事件
- `PUT` - 更新Inngest配置

**实现代码:**
```typescript
// app/api/inngest/route.ts:5-8
export const { GET, POST, PUT } = serve({
    client: inngest,
    functions: [sendSignUpEmail, sendDailyNewsSummary],
})
```

**注册的Inngest函数:**
1. `sendSignUpEmail` - 用户注册欢迎邮件
2. `sendDailyNewsSummary` - 每日市场新闻摘要

**认证:** 无需认证 (Inngest内部通信)

**请求体格式:** Inngest事件格式 (JSON)

**响应格式:** JSON

---

## ⚡ Server Actions 清单 | Server Actions

Server Actions是Next.js 15的核心特性，允许在服务器端执行函数而无需创建API端点。

### 总览 | Overview

| 文件 | Actions数量 | 功能域 |
|------|------------|--------|
| lib/actions/auth.actions.ts | 3 | 用户认证 |
| lib/actions/watchlist.actions.ts | 1 | 监控列表 |
| lib/actions/user.actions.ts | 1 | 用户管理 |
| lib/actions/finnhub.actions.ts | 2 | 股票数据 |

**总计:** 7个Server Actions

---

### 1. 认证相关 (Auth Actions)

**文件位置:** `lib/actions/auth.actions.ts`

#### 1.1 signUpWithEmail

**函数签名:**
```typescript
async ({
  email,
  password,
  fullName,
  country,
  investmentGoals,
  riskTolerance,
  preferredIndustry
}: SignUpFormData) => Promise<{ success: boolean; data?: any; error?: string }>
```

**功能:** 用户邮箱注册

**代码位置:** lib/actions/auth.actions.ts:7-23

**执行流程:**
```
1. 接收注册表单数据
   ↓
2. 调用 Better Auth API
   auth.api.signUpEmail({ email, password, name: fullName })
   ↓
3. 创建用户账户 (写入MongoDB user集合)
   ↓
4. 触发 Inngest 事件 'app/user.created'
   (发送欢迎邮件)
   ↓
5. 返回成功/失败结果
```

**触发的Inngest事件:**
```typescript
// lib/actions/auth.actions.ts:12-15
await inngest.send({
    name: 'app/user.created',
    data: {
      email,
      name: fullName,
      country,
      investmentGoals,
      riskTolerance,
      preferredIndustry
    }
})
```

**返回值:**
- 成功: `{ success: true, data: response }`
- 失败: `{ success: false, error: 'Sign up failed' }`

**错误处理:**
- try-catch包裹
- 捕获所有异常并返回错误消息
- 控制台日志记录错误

---

#### 1.2 signInWithEmail

**函数签名:**
```typescript
async ({ email, password }: SignInFormData) => Promise<{ success: boolean; data?: any; error?: string }>
```

**功能:** 用户邮箱登录

**代码位置:** lib/actions/auth.actions.ts:25-34

**执行流程:**
```
1. 接收登录凭证
   ↓
2. 调用 Better Auth API
   auth.api.signInEmail({ email, password })
   ↓
3. 验证密码并创建session
   ↓
4. 设置session cookie
   ↓
5. 返回用户数据
```

**认证机制:** Session-based (Better Auth管理)

**返回值:**
- 成功: `{ success: true, data: response }`
- 失败: `{ success: false, error: 'Sign in failed' }`

**Session存储:** MongoDB `session` 集合

---

#### 1.3 signOut

**函数签名:**
```typescript
async () => Promise<void | { success: boolean; error?: string }>
```

**功能:** 用户登出

**代码位置:** lib/actions/auth.actions.ts:36-43

**执行流程:**
```
1. 获取当前请求headers
   await headers()
   ↓
2. 调用 Better Auth API
   auth.api.signOut({ headers })
   ↓
3. 清除session cookie
   ↓
4. 删除session记录 (MongoDB)
```

**错误处理:**
- 捕获异常并返回错误
- 不抛出异常（静默失败）

---

### 2. 监控列表相关 (Watchlist Actions)

**文件位置:** `lib/actions/watchlist.actions.ts`

#### 2.1 getWatchlistSymbolsByEmail

**函数签名:**
```typescript
async (email: string): Promise<string[]>
```

**功能:** 根据用户邮箱获取监控列表股票代码

**代码位置:** lib/actions/watchlist.actions.ts:6-28

**执行流程:**
```
1. 连接MongoDB数据库
   ↓
2. 根据email查询user集合
   db.collection('user').findOne({ email })
   ↓
3. 获取userId (user.id 或 user._id)
   ↓
4. 查询watchlists集合
   Watchlist.find({ userId }, { symbol: 1 })
   ↓
5. 提取symbol数组
   ["AAPL", "GOOGL", "MSFT"]
   ↓
6. 返回股票代码数组
```

**性能优化:**
- 使用 `.lean()` 返回纯对象（减少内存）
- Projection `{ symbol: 1 }` 只查询需要的字段

**错误处理:**
- 邮箱为空 → 返回 `[]`
- 用户不存在 → 返回 `[]`
- 数据库错误 → 捕获异常，返回 `[]`

**返回值示例:**
```typescript
["AAPL", "GOOGL", "MSFT", "TSLA"]
```

**调用场景:**
- 每日新闻摘要生成 (获取用户关注的股票)
- 首页监控列表显示

---

### 3. 用户管理相关 (User Actions)

**文件位置:** `lib/actions/user.actions.ts`

#### 3.1 getAllUsersForNewsEmail

**函数签名:**
```typescript
async (): Promise<UserForNewsEmail[]>
```

**功能:** 获取所有需要接收新闻邮件的用户

**代码位置:** lib/actions/user.actions.ts:5-25

**执行流程:**
```
1. 连接MongoDB数据库
   ↓
2. 查询user集合
   db.collection('user').find({
     email: { $exists: true, $ne: null }
   })
   ↓
3. 投影查询字段
   { _id, id, email, name, country }
   ↓
4. 过滤无效用户 (email和name必须存在)
   ↓
5. 格式化返回数据
   { id, email, name }
```

**查询条件:**
- email字段存在且不为null

**Projection字段:**
```typescript
{
  _id: 1,
  id: 1,
  email: 1,
  name: 1,
  country: 1
}
```

**数据清洗:**
```typescript
// lib/actions/user.actions.ts:16-20
return users.filter((user) => user.email && user.name).map((user) => ({
    id: user.id || user._id?.toString() || '',
    email: user.email,
    name: user.name
}))
```

**返回值示例:**
```typescript
[
  { id: "user_xxx", email: "user1@example.com", name: "John Doe" },
  { id: "user_yyy", email: "user2@example.com", name: "Jane Smith" }
]
```

**调用场景:**
- 每日新闻摘要Cron任务

---

### 4. 股票数据相关 (Finnhub Actions)

**文件位置:** `lib/actions/finnhub.actions.ts`

#### 4.1 getNews

**函数签名:**
```typescript
async (symbols?: string[]): Promise<MarketNewsArticle[]>
```

**功能:** 获取市场新闻或特定股票新闻

**代码位置:** lib/actions/finnhub.actions.ts:25-99

**参数说明:**
- `symbols` (可选): 股票代码数组，如 `["AAPL", "GOOGL"]`
- 无参数时返回通用市场新闻

**执行流程:**

**情况1: 有指定股票代码**
```
1. 清理股票代码 (去空格、转大写)
   ↓
2. 并行获取每只股票的新闻
   Finnhub API: /company-news?symbol=AAPL&from=X&to=Y
   ↓
3. Round-robin算法选择6篇文章
   (每只股票轮流选1篇，直到6篇)
   ↓
4. 按时间排序 (最新在前)
   ↓
5. 返回格式化文章数组
```

**情况2: 无指定股票 (通用新闻)**
```
1. 调用Finnhub通用新闻API
   /news?category=general
   ↓
2. 去重 (基于id + url + headline)
   ↓
3. 取前6篇
   ↓
4. 格式化并返回
```

**Finnhub API调用:**
```typescript
// 公司新闻
const url = `${FINNHUB_BASE_URL}/company-news?symbol=${sym}&from=${range.from}&to=${range.to}&token=${token}`;

// 通用新闻
const url = `${FINNHUB_BASE_URL}/news?category=general&token=${token}`;
```

**数据验证:**
```typescript
// lib/utils.ts:76-77
export const validateArticle = (article: RawNewsArticle) =>
    article.headline && article.summary && article.url && article.datetime;
```

**返回值结构:**
```typescript
{
  id: number,
  headline: string,
  summary: string (最多150-200字符),
  source: string,
  url: string,
  datetime: number,
  image: string,
  category: string,
  related: string (股票代码)
}
```

**错误处理:**
- API密钥缺失 → 抛出异常
- 网络错误 → 捕获并抛出
- 文章验证失败 → 跳过该文章

**缓存策略:**
- 使用Next.js fetch缓存
- `revalidate: 300` (5分钟)

---

#### 4.2 searchStocks

**函数签名:**
```typescript
cache(async (query?: string): Promise<StockWithWatchlistStatus[]>)
```

**功能:** 搜索股票或获取热门股票列表

**代码位置:** lib/actions/finnhub.actions.ts:101-180

**React缓存:** 使用 `cache()` 包裹，同一请求期间复用结果

**参数说明:**
- `query` (可选): 搜索关键词
- 无参数时返回前10只热门股票

**执行流程:**

**情况1: 无搜索词 (热门股票)**
```
1. 获取前10个热门股票代码
   POPULAR_STOCK_SYMBOLS.slice(0, 10)
   ↓
2. 并行获取每只股票的profile2数据
   Finnhub API: /stock/profile2?symbol=AAPL
   ↓
3. 提取公司名称和交易所信息
   ↓
4. 格式化为标准格式
   ↓
5. 返回结果 (最多15条)
```

**情况2: 有搜索词**
```
1. 调用Finnhub搜索API
   /search?q=apple
   ↓
2. 获取搜索结果
   ↓
3. 格式化数据 (统一大写、提取交易所)
   ↓
4. 返回前15条结果
```

**Finnhub API调用:**
```typescript
// 获取股票Profile
const url = `${FINNHUB_BASE_URL}/stock/profile2?symbol=${sym}&token=${token}`;

// 搜索股票
const url = `${FINNHUB_BASE_URL}/search?q=${encodeURIComponent(trimmed)}&token=${token}`;
```

**数据格式化:**
```typescript
// lib/actions/finnhub.actions.ts:164-171
{
  symbol: "AAPL",         // 统一转为大写
  name: "Apple Inc.",
  exchange: "NASDAQ",
  type: "Common Stock",
  isInWatchlist: false    // 前端组件会更新此字段
}
```

**热门股票列表:**
- 来源: `lib/constants.ts:265-325`
- 数量: 50只股票
- 分类: 科技巨头、新兴科技、消费应用、国际公司

**缓存策略:**
- Profile2: `revalidate: 3600` (1小时)
- 搜索结果: `revalidate: 1800` (30分钟)

**错误处理:**
- API密钥缺失 → 返回空数组 `[]`
- 网络错误 → 捕获异常，返回空数组

---

## 🔄 中间件链分析 | Middleware Chain

**文件位置:** `middleware/index.ts`

### 中间件执行流程

```
用户请求 URL
    ↓
middleware/index.ts:4-12 执行
    ↓
检查 session cookie (Better Auth)
    ↓
    ├─ 有cookie
    │    ↓
    │  NextResponse.next()
    │  (继续访问页面)
    │
    └─ 无cookie
         ↓
      NextResponse.redirect("/")
      (重定向到首页)
```

### 中间件配置

**代码:**
```typescript
// middleware/index.ts:14-18
export const config = {
    matcher: [
        '/((?!api|_next/static|_next/image|favicon.ico|sign-in|sign-up|assets).*)',
    ],
};
```

**保护的路由:** 除了以下路径外的所有路由
- `/api/*` - API端点
- `/_next/static/*` - Next.js静态资源
- `/_next/image/*` - Next.js图片优化
- `/favicon.ico` - 网站图标
- `/sign-in` - 登录页
- `/sign-up` - 注册页
- `/assets/*` - 静态资源

**开放访问的路由:**
- `/sign-in` - 登录页
- `/sign-up` - 注册页
- `/` - 首页 (未登录时显示为登录页)

**受保护的路由示例:**
- `/stocks/AAPL` - 需要登录
- `/watchlist` - 需要登录
- `/search` - 需要登录

### Session验证

**验证方式:**
```typescript
// middleware/index.ts:5
const sessionCookie = getSessionCookie(request);
```

**Session Cookie名称:** Better Auth默认 (`better-auth.session_token`)

**Cookie属性:**
- HttpOnly: true (防止XSS)
- Secure: true (仅HTTPS，生产环境)
- SameSite: Lax (CSRF防护)

---

## 🧩 业务逻辑分层 | Business Logic Layers

### 架构层次

```
┌─────────────────────────────────────┐
│     UI Layer (React Components)      │
│  app/(root)/page.tsx, etc.          │
└─────────────────────────────────────┘
                ↓
┌─────────────────────────────────────┐
│   Server Actions Layer               │
│  lib/actions/*.ts                   │
│  - 参数验证                          │
│  - 业务逻辑编排                       │
│  - 错误处理                          │
└─────────────────────────────────────┘
                ↓
┌─────────────────────────────────────┐
│   Data Access Layer                  │
│  database/models/*.ts                │
│  - Mongoose模型                      │
│  - MongoDB查询                       │
└─────────────────────────────────────┘
                ↓
┌─────────────────────────────────────┐
│   External Services Layer            │
│  - Finnhub API (股票数据)            │
│  - Better Auth (认证)                │
│  - Inngest (事件驱动)                │
│  - Nodemailer (邮件)                 │
└─────────────────────────────────────┘
```

### 业务逻辑示例：用户注册流程

**涉及的层次:**

1. **UI Layer**
   - `app/(auth)/sign-up/page.tsx` - 注册表单

2. **Server Actions Layer**
   - `lib/actions/auth.actions.ts:signUpWithEmail`
   - 验证输入
   - 调用Better Auth
   - 触发Inngest事件

3. **External Services**
   - Better Auth - 创建用户账户
   - MongoDB - 存储用户数据
   - Inngest - 异步发送邮件

4. **Event-Driven Workflow**
   - `lib/inngest/functions.ts:sendSignUpEmail`
   - AI生成个性化内容
   - Nodemailer发送邮件

---

## 🌐 外部服务集成 | External Services Integration

### 1. Finnhub API (股票数据)

**官网:** https://finnhub.io
**用途:** 实时股票数据、财务数据、市场新闻

#### API端点使用

| 端点 | 用途 | 代码位置 | 缓存时间 |
|------|------|----------|----------|
| `/company-news` | 公司新闻 | finnhub.actions.ts:45 | 5分钟 |
| `/news` | 通用市场新闻 | finnhub.actions.ts:79 | 5分钟 |
| `/stock/profile2` | 公司信息 | finnhub.actions.ts:120 | 1小时 |
| `/search` | 股票搜索 | finnhub.actions.ts:151 | 30分钟 |

#### 认证方式

**方法1: Query参数 (优先)**
```typescript
const url = `${FINNHUB_BASE_URL}/endpoint?token=${process.env.FINNHUB_API_KEY}`;
```

**方法2: Header (备用)**
```typescript
headers: { 'X-Finnhub-Token': process.env.FINNHUB_API_KEY }
```

**环境变量:**
- `FINNHUB_API_KEY` (服务器端)
- `NEXT_PUBLIC_FINNHUB_API_KEY` (客户端)

#### 请求限制

**免费层级:**
- 60 API calls/minute
- 项目实际使用: 低频调用 + Next.js缓存

#### 错误处理

```typescript
// lib/actions/finnhub.actions.ts:15-20
const res = await fetch(url, options);
if (!res.ok) {
    const text = await res.text().catch(() => '');
    throw new Error(`Fetch failed ${res.status}: ${text}`);
}
```

---

### 2. Better Auth (认证服务)

**官网:** https://www.better-auth.com
**版本:** 1.3.7
**用途:** 邮箱密码认证、Session管理

#### 配置

**文件位置:** `lib/better-auth/auth.ts:16-29`

```typescript
betterAuth({
    database: mongodbAdapter(db),
    secret: process.env.BETTER_AUTH_SECRET,
    baseURL: process.env.BETTER_AUTH_URL,
    emailAndPassword: {
        enabled: true,
        disableSignUp: false,
        requireEmailVerification: false,
        minPasswordLength: 8,
        maxPasswordLength: 128,
        autoSignIn: true,
    },
    plugins: [nextCookies()],
})
```

#### API方法

| 方法 | 用途 | 代码位置 |
|------|------|----------|
| `auth.api.signUpEmail()` | 注册 | auth.actions.ts:9 |
| `auth.api.signInEmail()` | 登录 | auth.actions.ts:27 |
| `auth.api.signOut()` | 登出 | auth.actions.ts:38 |

#### Session管理

**存储方式:** MongoDB `session` 集合
**Cookie名称:** `better-auth.session_token`
**有效期:** 默认7天

---

### 3. Inngest (事件驱动平台)

**官网:** https://www.inngest.com
**版本:** 3.40.1
**用途:** 异步任务、定时任务、AI工作流

#### 注册的函数

**文件位置:** `lib/inngest/functions.ts`

| 函数名 | 触发方式 | 执行内容 | 代码位置 |
|--------|----------|----------|----------|
| `sendSignUpEmail` | 事件: `app/user.created` | 发送欢迎邮件 | functions.ts:9-49 |
| `sendDailyNewsSummary` | Cron: `0 12 * * *`<br>事件: `app/send.daily.news` | 发送每日新闻摘要 | functions.ts:51-120 |

#### 事件触发

**手动触发事件:**
```typescript
// lib/actions/auth.actions.ts:12-15
await inngest.send({
    name: 'app/user.created',
    data: { email, name, country, ... }
})
```

**Cron触发:**
```typescript
// lib/inngest/functions.ts:53
{ cron: '0 12 * * *' }  // 每天UTC 12:00 (北京时间20:00)
```

#### AI集成

**AI提供商:** Google Gemini
**模型:** gemini-2.5-flash-lite

**AI调用示例:**
```typescript
// lib/inngest/functions.ts:22-33
const response = await step.ai.infer('generate-welcome-intro', {
    model: step.ai.models.gemini({ model: 'gemini-2.5-flash-lite' }),
    body: {
        contents: [
            { role: 'user', parts: [{ text: prompt }] }
        ]
    }
})
```

**AI用途:**
1. 个性化欢迎邮件内容生成
2. 每日新闻摘要生成

---

### 4. Nodemailer (邮件服务)

**版本:** 7.0.6
**SMTP服务:** Gmail
**用途:** 发送欢迎邮件、新闻摘要邮件

#### 配置

**文件位置:** `lib/nodemailer/index.ts:4-10`

```typescript
nodemailer.createTransporter({
    service: 'gmail',
    auth: {
        user: process.env.NODEMAILER_EMAIL,
        pass: process.env.NODEMAILER_PASSWORD,  // Gmail应用专用密码
    }
})
```

#### 邮件函数

| 函数名 | 用途 | 代码位置 |
|--------|------|----------|
| `sendWelcomeEmail` | 发送欢迎邮件 | nodemailer/index.ts:12-26 |
| `sendNewsSummaryEmail` | 发送新闻摘要 | nodemailer/index.ts:28-44 |

#### 邮件模板

**文件位置:** `lib/nodemailer/templates.ts`

- `WELCOME_EMAIL_TEMPLATE` - 欢迎邮件HTML模板
- `NEWS_SUMMARY_EMAIL_TEMPLATE` - 新闻摘要HTML模板

**模板变量:**
```typescript
// 欢迎邮件
{{name}}  - 用户名
{{intro}} - AI生成的个性化介绍

// 新闻摘要
{{date}}        - 日期
{{newsContent}} - AI生成的新闻内容
```

#### 发件人配置

```typescript
from: '"Signalist" <signalist@jsmastery.pro>'
```

---

### 5. TradingView Widgets (图表组件)

**用途:** 股票图表可视化
**集成方式:** 外部JavaScript脚本

**Widget类型:** (lib/constants.ts)
- `market-overview` - 市场概览
- `stock-heatmap` - 股票热力图
- `timeline` - 新闻时间线
- `market-quotes` - 市场行情
- `symbol-info` - 股票信息
- `advanced-chart` - K线图
- `technical-analysis` - 技术分析
- `company-profile` - 公司简介
- `financials` - 财务数据

**脚本URL格式:**
```typescript
`https://s3.tradingview.com/external-embedding/embed-widget-${widget-name}.js`
```

**无需API密钥:** 免费使用

---

## ❌ 错误处理模式 | Error Handling Patterns

### 模式1: Try-Catch + 返回错误对象

**使用场景:** Server Actions

**示例:**
```typescript
// lib/actions/auth.actions.ts:7-23
try {
    const response = await auth.api.signUpEmail({ body: { email, password, name } })
    return { success: true, data: response }
} catch (e) {
    console.log('Sign up failed', e)
    return { success: false, error: 'Sign up failed' }
}
```

**优点:**
- 不会中断用户体验
- 前端可以根据success字段处理

---

### 模式2: Try-Catch + 返回空数组/默认值

**使用场景:** 查询类函数

**示例:**
```typescript
// lib/actions/watchlist.actions.ts:9-27
try {
    const items = await Watchlist.find({ userId }).lean();
    return items.map((i) => String(i.symbol));
} catch (err) {
    console.error('getWatchlistSymbolsByEmail error:', err);
    return [];  // 返回空数组而非抛出异常
}
```

**优点:**
- 降级优雅 (Graceful Degradation)
- 不影响其他功能

---

### 模式3: 抛出异常

**使用场景:** 严重错误需要中断执行

**示例:**
```typescript
// lib/actions/finnhub.actions.ts:28-31
const token = process.env.FINNHUB_API_KEY ?? NEXT_PUBLIC_FINNHUB_API_KEY;
if (!token) {
    throw new Error('FINNHUB API key is not configured');
}
```

---

### 模式4: 控制台日志 + 静默失败

**使用场景:** 非关键功能

**示例:**
```typescript
// lib/inngest/functions.ts:102-104
} catch (e) {
    console.error('Failed to summarize news for : ', user.email);
    userNewsSummaries.push({ user, newsContent: null });
}
```

---

## 🔍 数据流示例 | Data Flow Examples

### 示例1: 用户注册完整流程

```
前端表单提交
  ↓
Server Action: signUpWithEmail()
  ├─ Better Auth: 创建用户
  ├─ MongoDB: 写入user集合
  └─ Inngest: 触发 'app/user.created' 事件
      ↓
Inngest Function: sendSignUpEmail()
  ├─ Step 1: AI生成个性化欢迎文本
  │   Gemini API → 返回HTML内容
  ├─ Step 2: 发送欢迎邮件
  │   Nodemailer → Gmail SMTP
  └─ 返回成功状态
      ↓
用户收到欢迎邮件
```

**时间线:**
- T+0s: 用户点击注册
- T+1s: 用户账户创建完成
- T+2s: Inngest接收事件
- T+5s: AI生成内容
- T+6s: 邮件发送
- T+10s: 用户收到邮件

---

### 示例2: 每日新闻摘要流程

```
Cron触发 (每天12:00 UTC)
  ↓
Inngest Function: sendDailyNewsSummary()
  ↓
Step 1: 获取所有用户
  getAllUsersForNewsEmail()
  MongoDB: db.collection('user').find()
  → [user1, user2, ...]
  ↓
Step 2: 对每个用户:
  ├─ 获取监控列表
  │  getWatchlistSymbolsByEmail(email)
  │  → ["AAPL", "GOOGL"]
  │  ↓
  ├─ 获取股票新闻
  │  getNews(["AAPL", "GOOGL"])
  │  Finnhub API: /company-news
  │  → [article1, article2, ...]
  │  ↓
  └─ AI生成新闻摘要
      Gemini API: step.ai.infer()
      → HTML格式的新闻摘要
  ↓
Step 3: 批量发送邮件
  Promise.all([
    sendNewsSummaryEmail(user1),
    sendNewsSummaryEmail(user2),
    ...
  ])
  ↓
所有用户收到邮件
```

**执行时间:** 约30-60秒 (取决于用户数量)

---

### 示例3: 股票搜索流程

```
用户输入 "apple"
  ↓
前端防抖 (useDebounce, 300ms)
  ↓
Server Action: searchStocks("apple")
  ├─ React cache检查 (同一请求周期内复用)
  ├─ Finnhub API: /search?q=apple
  └─ 返回结果 (缓存30分钟)
      ↓
前端展示搜索结果
  [
    { symbol: "AAPL", name: "Apple Inc.", ... },
    { symbol: "APLE", name: "Apple Hospitality REIT", ... },
    ...
  ]
```

**性能优化:**
- 防抖减少API调用
- React cache避免重复请求
- Next.js fetch缓存减少外部API调用

---

## 📊 性能优化策略 | Performance Optimization

### 1. 缓存策略

**React Server Component Cache:**
```typescript
import { cache } from 'react';
export const searchStocks = cache(async (query?: string) => { ... });
```

**Next.js Fetch Cache:**
```typescript
fetch(url, {
    cache: 'force-cache',
    next: { revalidate: 300 }  // 5分钟
})
```

### 2. 数据库优化

**Lean查询:**
```typescript
Watchlist.find({ userId }).lean()  // 返回纯对象，减少内存
```

**Projection:**
```typescript
db.collection('user').find({}, {
    projection: { _id: 1, email: 1 }  // 只查询需要的字段
})
```

**索引:**
```typescript
// watchlist.model.ts:21
{ userId: 1, symbol: 1 }, { unique: true }
```

### 3. 并发处理

**Promise.all并行调用:**
```typescript
// lib/actions/finnhub.actions.ts:41-53
await Promise.all(
    cleanSymbols.map(async (sym) => {
        const articles = await fetchJSON(url);
        // ...
    })
);
```

### 4. 错误降级

**优雅降级策略:**
- Finnhub API失败 → 返回空数组 (不影响页面渲染)
- AI生成失败 → 使用默认文本
- 邮件发送失败 → 记录日志，不阻塞流程

---

## 🛡️ 安全措施 | Security Measures

### 1. 环境变量保护

**敏感信息不硬编码:**
```typescript
const MONGODB_URI = process.env.MONGODB_URI;
const FINNHUB_API_KEY = process.env.FINNHUB_API_KEY;
const BETTER_AUTH_SECRET = process.env.BETTER_AUTH_SECRET;
```

### 2. Server Actions安全

**'use server' 指令:**
```typescript
'use server';  // 确保代码只在服务器端执行
```

**输入验证:** (应添加但当前缺失)
- 建议使用Zod进行参数验证

### 3. API密钥保护

**服务器端密钥:**
```typescript
process.env.FINNHUB_API_KEY  // 仅服务器端可用
```

**客户端密钥 (谨慎使用):**
```typescript
process.env.NEXT_PUBLIC_FINNHUB_API_KEY  // 公开可见
```

### 4. CSRF防护

**Better Auth内置:**
- Session Cookie设置SameSite属性
- 中间件验证session

---

## 📝 最佳实践总结 | Best Practices

✅ **已实现:**
1. Server Actions代替REST API
2. TypeScript类型安全
3. 错误处理try-catch
4. 数据库连接池缓存
5. 环境变量管理
6. 事件驱动架构

⚠️ **可改进:**
1. 添加输入验证 (Zod)
2. 添加速率限制
3. 实现数据库迁移系统
4. 添加单元测试
5. 实现更细粒度的错误类型
6. 添加日志系统 (Winston/Pino)
7. 实现监控和告警

---

**文档版本:** v1.0
**生成时间:** 2025-11-16
**分析Server Actions:** 7个
**分析API端点:** 1个
**分析外部服务:** 5个
**下一步:** 进入前端分析阶段 → `03-frontend-analysis.md`
