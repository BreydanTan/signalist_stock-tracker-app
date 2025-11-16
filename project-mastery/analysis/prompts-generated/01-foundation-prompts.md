# 项目重建提示词 - 第1部分：基础框架 | Rebuild Prompts - Part 1: Foundation

## 使用说明 | Instructions

这些提示词可以直接复制粘贴给AI助手（如Claude、ChatGPT等），按顺序执行即可重建项目。

---

## 提示词 1.1：初始化Next.js项目

```
创建一个新的Next.js 15项目，使用以下配置：

项目名称：signalist-stock-tracker
框架：Next.js 15.5.2 (App Router)
语言：TypeScript 5
包管理器：npm

执行步骤：
1. 运行 `npx create-next-app@15.5.2 signalist-stock-tracker`
2. 配置选项：
   - TypeScript: Yes
   - ESLint: Yes
   - Tailwind CSS: Yes
   - src/ directory: No
   - App Router: Yes
   - Turbopack: Yes
   - Import alias: @/*

3. 安装精确版本的核心依赖：
```bash
cd signalist-stock-tracker

npm install react@19.1.0 react-dom@19.1.0
npm install next@15.5.2
npm install typescript@5
```

完成后告诉我结果。
```

---

## 提示词 1.2：安装UI组件库

```
在当前Next.js项目中安装Shadcn UI和相关依赖：

1. 安装Shadcn CLI和Radix UI组件：
```bash
npm install @radix-ui/react-avatar@1.1.10
npm install @radix-ui/react-dialog@1.1.15
npm install @radix-ui/react-dropdown-menu@2.1.16
npm install @radix-ui/react-label@2.1.7
npm install @radix-ui/react-popover@1.1.15
npm install @radix-ui/react-select@2.2.6
npm install @radix-ui/react-slot@1.2.3
```

2. 安装其他UI依赖：
```bash
npm install cmdk@1.1.1                      # 命令面板
npm install lucide-react@0.542.0            # 图标库
npm install sonner@2.0.7                    # Toast通知
npm install class-variance-authority@0.7.1  # 条件类名
npm install clsx@2.1.1                      # 类名合并
npm install tailwind-merge@3.3.1            # Tailwind优化
npm install next-themes@0.4.6               # 主题切换
```

3. 创建 `components.json` 配置文件，内容如下：
```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.ts",
    "css": "app/globals.css",
    "baseColor": "slate",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils"
  }
}
```

完成后确认。
```

---

## 提示词 1.3：配置Tailwind CSS

```
更新Tailwind CSS配置以支持暗色模式和自定义样式：

1. 修改 `tailwind.config.ts`：
```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  darkMode: ["class"],
  content: [
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      colors: {
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
      },
    },
  },
  plugins: [],
};
export default config;
```

2. 修改 `app/globals.css`，添加暗色主题变量：
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
  }
}

body {
  color: var(--foreground);
  background: var(--background);
}
```

完成后确认。
```

---

## 提示词 1.4：安装数据库和认证依赖

```
安装MongoDB、Mongoose和Better Auth相关依赖：

1. 数据库依赖：
```bash
npm install mongodb@6.19.0
npm install mongoose@8.18.0
```

2. 认证依赖：
```bash
npm install better-auth@1.3.7
```

3. 表单处理依赖：
```bash
npm install react-hook-form@7.62.0
npm install react-select-country-list@2.2.3
```

完成后确认。
```

---

## 提示词 1.5：安装Inngest和邮件服务

```
安装事件驱动和邮件服务依赖：

1. Inngest（事件驱动工作流）：
```bash
npm install inngest@3.40.1
```

2. Nodemailer（邮件服务）：
```bash
npm install nodemailer@7.0.6
npm install --save-dev @types/nodemailer@7.0.1
```

3. 其他工具库：
```bash
npm install dotenv@16.4.5
```

完成后确认所有依赖安装成功。
```

---

## 提示词 1.6：配置TypeScript

```
更新TypeScript配置以支持路径别名和严格模式：

修改 `tsconfig.json`：
```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

完成后确认。
```

---

## 提示词 1.7：配置Next.js

```
更新Next.js配置以启用Turbopack并配置构建选项：

修改 `next.config.ts`：
```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  eslint: {
      ignoreDuringBuilds: true,
  },
  typescript: {
      ignoreBuildErrors: true
    }
};

export default nextConfig;
```

修改 `package.json` 的scripts部分：
```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build --turbopack",
    "start": "next start",
    "lint": "eslint"
  }
}
```

完成后确认。
```

---

## 提示词 1.8：创建环境变量模板

```
创建环境变量模板文件 `.env.example`：

```env
# 运行环境
NODE_ENV=development
NEXT_PUBLIC_BASE_URL=http://localhost:3000

# MongoDB数据库
MONGODB_URI=your-mongodb-connection-string

# Better Auth认证
BETTER_AUTH_SECRET=your-secret-min-32-characters
BETTER_AUTH_URL=http://localhost:3000

# Finnhub股票数据API
FINNHUB_API_KEY=your-finnhub-api-key
NEXT_PUBLIC_FINNHUB_API_KEY=your-finnhub-api-key
FINNHUB_BASE_URL=https://finnhub.io/api/v1

# Google Gemini AI
GEMINI_API_KEY=your-gemini-api-key

# Nodemailer邮件服务
NODEMAILER_EMAIL=your-email@gmail.com
NODEMAILER_PASSWORD=your-gmail-app-password
```

然后创建实际的 `.env.local` 文件（添加到.gitignore）：
1. 复制 `.env.example` 为 `.env.local`
2. 填写实际的API密钥和配置

确认 `.gitignore` 包含：
```
.env
.env.local
.env*.local
```

完成后确认。
```

---

## 检查点 1 | Checkpoint 1

✅ 完成以上所有提示词后，你应该有：

1. **项目结构：**
   ```
   signalist-stock-tracker/
   ├── app/
   ├── components/
   ├── lib/
   ├── public/
   ├── .env.local
   ├── .env.example
   ├── components.json
   ├── next.config.ts
   ├── package.json
   ├── tailwind.config.ts
   └── tsconfig.json
   ```

2. **依赖安装完成：**
   - Next.js 15.5.2
   - React 19.1.0
   - TypeScript 5
   - Shadcn UI组件
   - MongoDB + Mongoose
   - Better Auth
   - Inngest
   - Nodemailer

3. **配置文件：**
   - TypeScript配置
   - Tailwind CSS配置
   - Next.js配置
   - 环境变量模板

**验证命令：**
```bash
npm run dev
```

应该能成功启动开发服务器在 http://localhost:3000

---

**下一步:** 前往 `02-backend-prompts.md` 创建数据库模型和Server Actions
