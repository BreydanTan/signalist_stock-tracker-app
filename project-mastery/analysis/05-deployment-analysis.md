# 部署配置分析 | Deployment Configuration Analysis

## 🚀 部署概览 | Deployment Overview

本项目是一个Next.js 15全栈应用，支持多种部署方式。

---

## 📦 构建配置 | Build Configuration

### package.json脚本

**文件位置:** `package.json:5-11`

```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build --turbopack",
    "start": "next start",
    "lint": "eslint",
    "test:db": "node scripts/test-db.mjs"
  }
}
```

**脚本说明:**

| 命令 | 用途 | Turbopack |
|------|------|-----------|
| `npm run dev` | 开发服务器 | ✅ 启用 |
| `npm run build` | 生产构建 | ✅ 启用 |
| `npm start` | 启动生产服务器 | - |
| `npm run lint` | ESLint检查 | - |
| `npm run test:db` | 测试数据库连接 | - |

**Turbopack说明:**
- Next.js 15新增的打包工具
- 比Webpack快10倍+
- 开发和构建都使用

---

### Next.js配置

**文件位置:** `next.config.ts`

```typescript
const nextConfig: NextConfig = {
  eslint: {
      ignoreDuringBuilds: true,  // ⚠️ 跳过ESLint检查
  },
  typescript: {
      ignoreBuildErrors: true    // ⚠️ 跳过TypeScript错误
    }
};
```

**⚠️ 警告:**
- 生产构建会忽略类型错误和ESLint错误
- 建议修改为仅在CI中禁用

**建议配置:**
```typescript
const nextConfig: NextConfig = {
  eslint: {
      ignoreDuringBuilds: process.env.CI === 'true',
  },
  typescript: {
      ignoreBuildErrors: process.env.CI === 'true',
    }
};
```

---

### TypeScript配置

**文件位置:** `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "strict": true,
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

**关键配置:**
- ✅ `strict: true` - 启用严格类型检查
- ✅ 路径别名 `@/*` 简化导入

---

## 🌍 环境要求 | Environment Requirements

### 运行时要求

| 依赖 | 版本要求 | 来源 |
|------|----------|------|
| Node.js | ≥20 | package.json:42 |
| npm | ≥8 | - |
| MongoDB | ≥6.0 | 推荐 |

### 系统依赖

**必需:**
- 无特殊系统依赖
- 纯JavaScript/TypeScript项目

---

## 🔧 环境变量配置 | Environment Variables

### 开发环境 (.env.local)

```env
NODE_ENV=development
NEXT_PUBLIC_BASE_URL=http://localhost:3000

# MongoDB
MONGODB_URI=mongodb://localhost:27017/signalist
# 或 MongoDB Atlas
# MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/signalist

# Better Auth
BETTER_AUTH_SECRET=your-secret-here-min-32-chars
BETTER_AUTH_URL=http://localhost:3000

# Finnhub API
FINNHUB_API_KEY=your-finnhub-api-key
NEXT_PUBLIC_FINNHUB_API_KEY=your-finnhub-api-key
FINNHUB_BASE_URL=https://finnhub.io/api/v1

# Gemini AI
GEMINI_API_KEY=your-gemini-api-key

# Nodemailer (Gmail)
NODEMAILER_EMAIL=your-email@gmail.com
NODEMAILER_PASSWORD=your-app-specific-password
```

### 生产环境

**额外变量:**
```env
NODE_ENV=production
NEXT_PUBLIC_BASE_URL=https://yourdomain.com
BETTER_AUTH_URL=https://yourdomain.com

# 确保使用生产数据库
MONGODB_URI=mongodb+srv://...
```

---

## 🏗️ 推荐部署平台 | Recommended Platforms

### 1. Vercel (推荐 ⭐)

**优势:**
- ✅ Next.js官方推荐
- ✅ 零配置部署
- ✅ 自动CI/CD
- ✅ 全球CDN
- ✅ 免费层级慷慨

**部署步骤:**

```bash
# 1. 安装Vercel CLI
npm i -g vercel

# 2. 登录
vercel login

# 3. 部署
vercel --prod
```

**或通过GitHub集成:**
1. 导入GitHub仓库
2. 配置环境变量
3. 自动部署

**环境变量配置:**
- 在Vercel Dashboard添加所有环境变量
- 不要提交.env文件

**域名配置:**
- 默认: `project-name.vercel.app`
- 自定义域名: Settings → Domains

---

### 2. Railway

**优势:**
- ✅ 支持MongoDB内置
- ✅ 简单配置
- ✅ 合理定价

**部署步骤:**
```bash
# 1. 安装Railway CLI
npm i -g @railway/cli

# 2. 登录
railway login

# 3. 初始化
railway init

# 4. 添加服务
railway add

# 5. 部署
railway up
```

---

### 3. Docker部署

**Dockerfile (推荐):**

```dockerfile
FROM node:20-alpine AS base

# Dependencies
FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Builder
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Runner
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT=3000

CMD ["node", "server.js"]
```

**docker-compose.yml:**

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongo:27017/signalist
    env_file:
      - .env.production
    depends_on:
      - mongo

  mongo:
    image: mongo:6
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

**部署命令:**
```bash
docker-compose up -d
```

---

## 🔄 CI/CD配置 | CI/CD Configuration

### GitHub Actions示例

**文件位置:** `.github/workflows/deploy.yml`

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Build project
        run: npm run build
        env:
          MONGODB_URI: ${{ secrets.MONGODB_URI }}
          BETTER_AUTH_SECRET: ${{ secrets.BETTER_AUTH_SECRET }}
          # ... 其他环境变量

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID}}
          vercel-project-id: ${{ secrets.PROJECT_ID}}
          vercel-args: '--prod'
```

---

## 📊 性能优化建议 | Performance Optimization

### 构建优化

**已实现:**
- ✅ Turbopack快速构建
- ✅ Server Components减少bundle大小

**建议添加:**

```typescript
// next.config.ts
const nextConfig = {
  images: {
    formats: ['image/avif', 'image/webp'],  // 图片优化
  },
  compress: true,  // Gzip压缩
  poweredByHeader: false,  // 移除X-Powered-By头
}
```

---

### 数据库优化

**已实现:**
- ✅ 连接池缓存
- ✅ 索引优化

**建议:**
- 使用MongoDB Atlas (云托管)
- 启用自动备份
- 配置读副本 (高流量场景)

---

## 🔧 故障排查 | Troubleshooting

### 常见问题

#### 1. 构建失败

**问题:** `Module not found`

**解决:**
```bash
rm -rf node_modules package-lock.json
npm install
npm run build
```

#### 2. 数据库连接失败

**问题:** `MongoServerError: Authentication failed`

**检查清单:**
- [ ] `MONGODB_URI` 格式正确
- [ ] 数据库用户名密码正确
- [ ] IP白名单配置 (MongoDB Atlas)
- [ ] 网络连接正常

**测试脚本:**
```bash
npm run test:db
```

#### 3. 环境变量未生效

**Vercel:**
- 重新部署: `vercel --prod --force`
- 检查环境变量范围 (Production/Preview/Development)

**本地:**
- 重启开发服务器
- 检查`.env.local`文件存在

---

## 📋 部署检查清单 | Deployment Checklist

### 部署前

- [ ] 所有测试通过
- [ ] ESLint无错误
- [ ] TypeScript无错误
- [ ] 环境变量已配置
- [ ] 数据库连接测试成功
- [ ] API密钥有效

### 首次部署

- [ ] 配置域名
- [ ] 设置SSL证书 (自动)
- [ ] 配置邮件服务
- [ ] 测试Inngest webhook
- [ ] 验证所有页面加载
- [ ] 测试登录/注册流程

### 监控配置

- [ ] 错误追踪 (Sentry推荐)
- [ ] 性能监控 (Vercel Analytics)
- [ ] 数据库监控 (MongoDB Atlas)
- [ ] Uptime监控

---

## 🚦 健康检查 | Health Checks

### 端点建议

**添加健康检查端点:**

```typescript
// app/api/health/route.ts
export async function GET() {
  try {
    await connectToDatabase();
    return Response.json({ status: 'ok', database: 'connected' });
  } catch {
    return Response.json({ status: 'error', database: 'disconnected' }, { status: 500 });
  }
}
```

**访问:** `https://yourdomain.com/api/health`

---

## 📈 扩展性考虑 | Scalability

### 当前架构限制

**单体应用:**
- Next.js全栈应用
- 单一MongoDB实例

**扩展建议 (高流量场景):**

1. **数据库层:**
   - MongoDB分片
   - 读写分离
   - Redis缓存

2. **应用层:**
   - 无服务器函数 (Vercel Edge Functions)
   - CDN缓存静态资源

3. **队列系统:**
   - Inngest (已实现) ✅
   - 可扩展到其他任务

---

**文档版本:** v1.0
**生成时间:** 2025-11-16
**推荐部署:** Vercel
**下一步:** 生成重建提示词 → `prompts-generated/`
