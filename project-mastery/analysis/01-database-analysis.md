# 数据库深度分析 | Database Deep Analysis

## 📊 数据库概览 | Database Overview

**数据库类型:** MongoDB (NoSQL文档数据库)
**ODM框架:** Mongoose 8.18.0
**连接方式:** MongoDB Atlas (云数据库) 或 本地MongoDB
**连接管理:** 连接池缓存机制 (database/mongoose.ts:12-36)

## 🗄️ 数据库集合清单 | Collections List

项目使用 **2个主要集合 (Collections)**:

| 集合名称 | 创建方式 | 作用 | 代码位置 |
|----------|----------|------|----------|
| `user` | Better Auth自动创建 | 存储用户账户信息 | lib/better-auth/auth.ts:17 |
| `watchlists` | Mongoose模型定义 | 存储用户股票监控列表 | database/models/watchlist.model.ts:24 |

**其他Better Auth集合 (自动创建):**
- `session` - 用户会话管理
- `account` - 第三方账户关联 (如OAuth)
- `verification` - 邮箱验证令牌

## 📋 数据模型详细分析 | Detailed Data Models

### 1. User Collection (用户表)

**集合名称:** `user`
**管理方式:** Better Auth自动管理
**访问方式:** `db.collection('user')`

#### 字段结构 | Schema

| 字段名 | 类型 | 必需 | 说明 | 示例值 |
|--------|------|------|------|--------|
| `_id` | ObjectId | 是 | MongoDB自动生成的唯一ID | ObjectId("...") |
| `id` | String | 是 | Better Auth生成的用户ID | "user_abc123..." |
| `email` | String | 是 | 用户邮箱 (唯一) | "user@example.com" |
| `name` | String | 是 | 用户全名 | "John Doe" |
| `emailVerified` | Boolean | 否 | 邮箱是否已验证 | false |
| `image` | String | 否 | 用户头像URL | null |
| `createdAt` | Date | 是 | 创建时间 | ISODate("2025-11-16...") |
| `updatedAt` | Date | 是 | 更新时间 | ISODate("2025-11-16...") |

**自定义字段 (注册时存储):**

| 字段名 | 类型 | 说明 | 代码位置 |
|--------|------|------|----------|
| `country` | String | 用户所在国家 | lib/actions/auth.actions.ts:7 |
| `investmentGoals` | String | 投资目标 | lib/actions/auth.actions.ts:7 |
| `riskTolerance` | String | 风险承受能力 | lib/actions/auth.actions.ts:7 |
| `preferredIndustry` | String | 偏好行业 | lib/actions/auth.actions.ts:7 |

**注意:** 这些自定义字段通过Inngest事件异步存储，实际存储位置待确认。

#### 索引 | Indexes

Better Auth自动创建以下索引：
- `email` (唯一索引) - 确保邮箱唯一性
- `id` (唯一索引) - Better Auth用户ID

#### 数据访问代码位置

```typescript
// 查询单个用户
lib/actions/watchlist.actions.ts:15
const user = await db.collection('user').findOne({ email });

// 查询所有用户 (用于邮件推送)
lib/actions/user.actions.ts:11-14
const users = await db.collection('user').find(
  { email: { $exists: true, $ne: null }},
  { projection: { _id: 1, id: 1, email: 1, name: 1, country: 1 }}
).toArray();
```

---

### 2. Watchlist Collection (监控列表表)

**集合名称:** `watchlists`
**管理方式:** Mongoose模型
**模型名称:** `Watchlist`
**定义位置:** database/models/watchlist.model.ts

#### 完整Schema定义

```typescript
interface WatchlistItem extends Document {
  userId: string;      // 用户ID (来自Better Auth)
  symbol: string;      // 股票代码 (如 "AAPL")
  company: string;     // 公司名称
  addedAt: Date;       // 添加时间
}
```

#### 字段结构 | Schema

| 字段名 | 类型 | 必需 | 约束 | 说明 | 代码位置 |
|--------|------|------|------|------|----------|
| `_id` | ObjectId | 是 | 自动生成 | MongoDB唯一ID | - |
| `userId` | String | 是 | 索引 | 关联到user.id | watchlist.model.ts:12 |
| `symbol` | String | 是 | uppercase, trim | 股票代码 (大写) | watchlist.model.ts:13 |
| `company` | String | 是 | trim | 公司名称 | watchlist.model.ts:14 |
| `addedAt` | Date | 是 | default: Date.now | 添加时间 | watchlist.model.ts:15 |

#### 字段约束详解

**`userId` 字段:**
- 类型: String
- 必需: true
- 索引: true (单字段索引) - watchlist.model.ts:12
- 作用: 关联到 `user` 集合的 `id` 字段
- 示例: "user_abc123..."

**`symbol` 字段:**
- 类型: String
- 必需: true
- 自动转换: uppercase (自动转为大写) - watchlist.model.ts:13
- 自动处理: trim (去除首尾空格) - watchlist.model.ts:13
- 示例: "AAPL", "GOOGL", "MSFT"

**`company` 字段:**
- 类型: String
- 必需: true
- 自动处理: trim - watchlist.model.ts:14
- 示例: "Apple Inc.", "Alphabet Inc."

**`addedAt` 字段:**
- 类型: Date
- 默认值: Date.now (当前时间) - watchlist.model.ts:15
- 示例: ISODate("2025-11-16T12:30:00Z")

#### 索引 | Indexes

**复合唯一索引 (Compound Unique Index):**

```typescript
// 代码位置: watchlist.model.ts:21
{ userId: 1, symbol: 1 } // unique: true
```

**作用:** 确保同一用户不能添加重复的股票代码

**索引说明:**
- `userId: 1` - 升序索引
- `symbol: 1` - 升序索引
- `unique: true` - 唯一性约束

**实际效果:**
- ✅ 允许: 用户A添加AAPL，用户B也添加AAPL
- ❌ 禁止: 用户A重复添加AAPL

#### 示例文档 | Sample Document

```json
{
  "_id": ObjectId("673d5f8a1234567890abcdef"),
  "userId": "user_cm3kx9y8z0001abcdefghijk",
  "symbol": "AAPL",
  "company": "Apple Inc.",
  "addedAt": ISODate("2025-11-16T08:30:00.000Z")
}
```

---

## 🔗 数据关系图 | Entity Relationship Diagram

### ASCII ER图

```
┌─────────────────────────────────┐
│         user (Better Auth)       │
│─────────────────────────────────│
│ PK  _id: ObjectId               │
│ UK  id: String                  │ ◄──────┐
│ UK  email: String               │        │ 1对多关系
│     name: String                │        │ (One-to-Many)
│     emailVerified: Boolean      │        │
│     createdAt: Date             │        │
│     updatedAt: Date             │        │
│                                 │        │
│ [自定义字段]                     │        │
│     country: String?            │        │
│     investmentGoals: String?    │        │
│     riskTolerance: String?      │        │
│     preferredIndustry: String?  │        │
└─────────────────────────────────┘        │
                                           │
                                           │
                                           │
┌─────────────────────────────────┐        │
│         watchlists               │        │
│─────────────────────────────────│        │
│ PK  _id: ObjectId               │        │
│ FK  userId: String ──────────────────────┘
│ IDX symbol: String (uppercase)  │
│     company: String             │
│     addedAt: Date               │
│                                 │
│ [复合唯一索引]                   │
│ UNIQUE (userId, symbol)         │
└─────────────────────────────────┘
```

### 关系说明 | Relationship Details

**1对多关系 (One-to-Many): User → Watchlists**

- 一个用户 (User) 可以有多个监控项 (Watchlist Items)
- 一个监控项只属于一个用户
- 外键字段: `watchlists.userId` → `user.id`

**关系类型:** 软关联 (Soft Reference)
- MongoDB不强制外键约束
- 应用层负责数据一致性

---

## 🔄 数据流分析 | Data Flow Analysis

### 1. 用户注册流程 | User Registration Flow

```
用户提交注册表单
  ↓
lib/actions/auth.actions.ts:7-23 (signUpWithEmail)
  ↓
Better Auth API: auth.api.signUpEmail()
  ↓
MongoDB: 插入新记录到 'user' 集合
  {
    id: "user_xxx",
    email: "user@example.com",
    name: "John Doe",
    emailVerified: false,
    createdAt: Date.now()
  }
  ↓
触发 Inngest 事件: 'app/user.created'
  ↓
lib/inngest/functions.ts:9-49 (sendSignUpEmail)
  ↓
AI生成个性化欢迎文本
  ↓
发送欢迎邮件 (Nodemailer)
```

**代码位置:**
- 注册入口: lib/actions/auth.actions.ts:7-23
- Inngest事件: lib/actions/auth.actions.ts:12-15
- 邮件发送: lib/inngest/functions.ts:35-42

---

### 2. 添加股票到监控列表 | Add to Watchlist Flow

**前提:** 用户已登录，获取股票信息

```
用户点击"添加到监控列表"按钮
  ↓
调用 Server Action: addToWatchlist(userId, symbol, company)
  ↓
数据库查询: 检查是否已存在
  Watchlist.findOne({ userId, symbol })
  ↓
IF 不存在:
  ↓
  创建新文档
  Watchlist.create({
    userId: "user_xxx",
    symbol: "AAPL",      // 自动转为大写
    company: "Apple Inc.",
    addedAt: Date.now()
  })
  ↓
  MongoDB插入文档到 'watchlists' 集合
  ↓
  返回成功 { success: true }
  ↓
ELSE (已存在):
  ↓
  返回错误 { success: false, error: "已在监控列表中" }
```

**复合唯一索引保护:**
- MongoDB会自动拒绝 (userId, symbol) 重复的插入
- 错误代码: E11000 duplicate key error

**代码位置:**
- 添加操作: 需要查找实际的 Server Action (未在当前分析文件中找到)
- 模型定义: database/models/watchlist.model.ts:10-24

---

### 3. 获取用户监控列表 | Get User Watchlist Flow

```
用户访问首页/股票详情页
  ↓
调用 Server Action: getWatchlistSymbolsByEmail(email)
  ↓
lib/actions/watchlist.actions.ts:6-28
  ↓
1. 查询 user 集合
   db.collection('user').findOne({ email })
  ↓
2. 获取 userId
  ↓
3. 查询 watchlists 集合
   Watchlist.find({ userId }, { symbol: 1 })
  ↓
4. 提取 symbol 数组
   ["AAPL", "GOOGL", "MSFT"]
  ↓
5. 返回给前端组件
```

**性能优化:**
- 使用 `.lean()` 返回纯JavaScript对象 (不是Mongoose文档)
- 使用 projection `{ symbol: 1 }` 只返回需要的字段

**代码位置:**
- lib/actions/watchlist.actions.ts:6-28

---

### 4. 每日新闻推送流程 | Daily News Digest Flow

```
Inngest Cron触发 (每天12:00 UTC)
  ↓
lib/inngest/functions.ts:51-120 (sendDailyNewsSummary)
  ↓
Step 1: 获取所有用户
  db.collection('user').find({ email: { $exists: true }})
  ↓
Step 2: 对每个用户:
  ├─ 查询监控列表
  │  Watchlist.find({ userId })
  │  → ["AAPL", "GOOGL"]
  │  ↓
  ├─ 获取这些股票的新闻
  │  Finnhub API: /company-news?symbol=AAPL
  │  ↓
  └─ AI生成新闻摘要
     Gemini API (step.ai.infer)
  ↓
Step 3: 批量发送邮件
  Nodemailer.sendMail(newsContent)
```

**数据流经过:**
1. `user` 集合 → 获取用户列表
2. `watchlists` 集合 → 获取每个用户的股票
3. Finnhub API → 获取股票新闻
4. Gemini AI → 生成摘要
5. Nodemailer → 发送邮件

**代码位置:**
- lib/inngest/functions.ts:51-120

---

## 📊 数据操作模式 | Data Operation Patterns

### CRUD操作总结 | CRUD Operations

#### Create (创建)

```typescript
// 1. 创建用户 (Better Auth自动处理)
await auth.api.signUpEmail({ body: { email, password, name } });

// 2. 添加监控项 (需要创建此Server Action)
await Watchlist.create({ userId, symbol, company });
```

#### Read (读取)

```typescript
// 1. 查询用户
const user = await db.collection('user').findOne({ email });

// 2. 查询用户的监控列表
const items = await Watchlist.find({ userId }).lean();

// 3. 查询所有用户 (批量操作)
const users = await db.collection('user').find({
  email: { $exists: true }
}).toArray();
```

#### Update (更新)

```typescript
// 未在代码中发现更新操作
// 可能的场景:
// - 更新用户资料
// - 修改监控项的备注 (如果添加此字段)
```

#### Delete (删除)

```typescript
// 需要查找删除监控项的Server Action
// 预期代码:
await Watchlist.findOneAndDelete({ userId, symbol });
```

---

## 🔒 数据验证与约束 | Validation & Constraints

### Schema级别验证

| 集合 | 字段 | 验证规则 | 代码位置 |
|------|------|----------|----------|
| watchlists | userId | required | watchlist.model.ts:12 |
| watchlists | symbol | required, uppercase, trim | watchlist.model.ts:13 |
| watchlists | company | required, trim | watchlist.model.ts:14 |

### 数据库级别约束

| 约束类型 | 集合 | 字段 | 说明 |
|----------|------|------|------|
| 唯一索引 | user | email | Better Auth自动创建 |
| 唯一索引 | user | id | Better Auth自动创建 |
| 复合唯一索引 | watchlists | (userId, symbol) | watchlist.model.ts:21 |
| 单字段索引 | watchlists | userId | watchlist.model.ts:12 |

### 应用层验证

**Better Auth配置:**
```typescript
// lib/better-auth/auth.ts:24-25
minPasswordLength: 8,
maxPasswordLength: 128,
```

---

## 🗂️ 数据库迁移历史 | Migration History

**当前状态:** 无正式迁移系统
- Better Auth自动创建 `user`, `session`, `account` 等集合
- Mongoose在首次使用时自动创建 `watchlists` 集合
- 索引在模型定义时自动创建

**推荐改进:**
- 添加MongoDB迁移工具 (如 migrate-mongo)
- 版本化Schema变更

---

## 📈 性能优化建议 | Performance Optimization

### 当前优化

✅ **已实现:**
1. 连接池缓存 (database/mongoose.ts:12-36)
2. 复合索引 (userId + symbol) - 加速查询
3. Lean查询 (watchlist.actions.ts:22) - 减少内存
4. Projection限制字段 (user.actions.ts:13) - 减少数据传输

### 未来优化建议

🔄 **可改进:**
1. 添加 `symbol` 字段索引 (用于跨用户股票查询)
2. 实现查询结果缓存 (Redis)
3. 分页查询 (当监控列表项目过多时)
4. 批量操作优化 (bulkWrite)

---

## 🔍 示例查询 | Sample Queries

### 查询某用户的监控列表

```typescript
// 方法1: 使用Mongoose模型
const items = await Watchlist.find({ userId: "user_xxx" })
  .select('symbol company addedAt')
  .lean();

// 方法2: 直接使用MongoDB驱动
const items = await db.collection('watchlists').find(
  { userId: "user_xxx" }
).toArray();
```

### 检查股票是否在监控列表中

```typescript
const exists = await Watchlist.exists({
  userId: "user_xxx",
  symbol: "AAPL"
});
// 返回: { _id: ObjectId } 或 null
```

### 获取某股票的所有监控用户

```typescript
// 需求: 当AAPL价格变动时,通知所有监控此股票的用户
const watchers = await Watchlist.find({ symbol: "AAPL" })
  .distinct('userId');
// 返回: ["user_abc", "user_def", ...]
```

### 统计用户监控列表数量

```typescript
const count = await Watchlist.countDocuments({
  userId: "user_xxx"
});
// 返回: 5
```

---

## 🚨 数据一致性注意事项 | Data Consistency

### 孤立数据风险 (Orphaned Data)

**场景:** 用户删除账号后，监控列表数据仍保留

**当前状态:** ❌ 未实现级联删除

**推荐方案:**
```typescript
// 删除用户时同时删除监控列表
async function deleteUser(userId: string) {
  await db.collection('user').deleteOne({ id: userId });
  await Watchlist.deleteMany({ userId });
}
```

### 数据验证

**场景:** `watchlists.userId` 引用了不存在的用户

**当前保护:** MongoDB不强制外键约束，需应用层验证

**推荐方案:**
```typescript
// 添加监控项前验证用户存在
const userExists = await db.collection('user').findOne({ id: userId });
if (!userExists) throw new Error('User not found');
```

---

## 📊 数据模型图表 | Data Model Visualization

### 集合大小预估

| 集合 | 预计记录数 | 单文档大小 | 预计总大小 |
|------|-----------|-----------|-----------|
| user | 10,000 | ~500 bytes | ~5 MB |
| session | 20,000 | ~200 bytes | ~4 MB |
| watchlists | 100,000 | ~150 bytes | ~15 MB |

**说明:** 假设平均每用户监控10只股票

---

## 🔗 相关文档链接 | Related Documentation

- **Mongoose文档:** https://mongoosejs.com/docs/
- **Better Auth文档:** https://www.better-auth.com/docs
- **MongoDB索引最佳实践:** https://www.mongodb.com/docs/manual/indexes/

---

**文档版本:** v1.0
**生成时间:** 2025-11-16
**分析集合数:** 2个主要集合 + Better Auth集合
**下一步:** 进入后端分析阶段 → `02-backend-analysis.md`
