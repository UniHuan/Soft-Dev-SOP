# 🌐 Claude Code + Next.js 两天全自动 Web 应用开发 SOP

> **适用对象**: Claude Code (AI Agent) 全自动执行
> **目标**: 从零到生产部署，两天 (16 小时) 完成
> **技术栈**: Next.js 14+ (App Router) + TypeScript + Tailwind CSS + Prisma + SQLite
> **最低要求**: Node.js 20+, npm/pnpm, Vercel/GitHub 账号 (免费)

---

## 🧠 Claude Code 技能调用矩阵

| 技能标识 | 技能名称 | 说明 | Claude Code 工具 |
|---------|---------|------|-----------------|
| `[SHELL]` | Shell 执行 | 执行 npm/pnpm、npx、git、curl | `Bash` |
| `[WRITE]` | 文件写入 | 创建/修改 .tsx .ts .prisma .css 等 | `Edit` |
| `[READ]` | 文件读取 | 读取代码、日志、配置 | `Read` |
| `[DIALOG]` | 用户交互 | 提问、确认、展示 | 对话输出 |
| `[GENERATE]` | 代码生成 | 生成 React/TypeScript 代码 | `Edit` |
| `[REVIEW]` | 代码审查 | 审查代码质量、类型安全 | `Read`→分析 |
| `[DEBUG]` | 调试分析 | 分析编译/运行时错误并修复 | `Read`+`Bash`+`Edit` |
| `[RESEARCH]` | 知识检索 | 查阅 Next.js/React/Tailwind 文档 | 内置知识 |
| `[VALIDATE]` | 验证检查 | 清单逐项验证 | `Bash`+`Read` |
| `[GIT]` | 版本控制 | git add/commit/tag/push | `Bash` |

### 技能调用原则

```
1. 每个 Phase 前 → [RESEARCH] 查阅 Next.js/React/Prisma 最佳实践
2. 每次 WRITE 后 → npm run build → [DEBUG] 自动修复 TypeScript/ESLint 错误
3. 编译失败 → 读错误 → Edit 修复 → 重构建 (最多 5 次)
4. 每个 Phase 结束 → [VALIDATE] → [GIT] commit
5. 不可逆操作 → [DIALOG] 确认
6. npm install 失败 → 检查 Node.js 版本 → 用 pnpm 替代 → 重试
```

### 全局编译验证

```bash
source /tmp/sop_web.env 2>/dev/null
cd ${PROJECT_DIR}
npx tsc --noEmit 2>&1 | tail -10          # TypeScript 类型检查
npm run build 2>&1 | tail -10             # 完整构建
# [DEBUG] 如有错误, 分析并修复
```

---

## 📋 总览时间线

```
Day 1 (8h)                              Day 2 (8h)
├─ [0.0h] 环境检查 & 项目创建            ├─ [0.0h] Day 2 启动校验
├─ [0.5h] 产品需求确认                   ├─ [0.5h] 表单验证 & 错误处理
├─ [1.0h] PRD 产品需求文档               ├─ [1.5h] 认证 (NextAuth.js)
├─ [1.5h] 高保真原型 (Tailwind 快速原型) ├─ [2.5h] API 完善 & 中间件
├─ [2.5h] 数据库设计 (Prisma Schema)     ├─ [3.5h] 测试 (Vitest + Playwright)
├─ [3.5h] API 层 (Server Actions+Route)  ├─ [4.5h] SEO & 性能优化
├─ [5.5h] UI 组件库 (Tailwind + shadcn)  ├─ [5.5h] 部署配置 (Vercel)
├─ [7.0h] 页面实现 (App Router)          └─ [7.0h] 域名 & 生产上线
└─ [8.0h] 自测 & Day 1 收尾              └─ [8.0h] 归档 & 监控
```

---

## ⚠️ 执行前提

```bash
# [SHELL] 1. 确认 Node.js 20+
node -v | grep "v2[0-9]" || { echo "❌ 需要 Node.js 20+"; exit 1; }

# [SHELL] 2. 确认包管理器
which npm && echo "✅ npm $(npm -v)" || echo "❌"
which pnpm && echo "✅ pnpm $(pnpm -v)" || echo "⚠️ pnpm 推荐安装: npm i -g pnpm"

# [SHELL] 3. 确认 Git
git --version || { echo "❌ Git 未安装"; exit 1; }

# [SHELL] 4. Vercel/GitHub 账号
echo "请确认: 已注册 GitHub + Vercel 账号 (均免费)"
# [DIALOG] 等待用户确认
```

---

## Phase 0: 环境初始化 (0:00-0:30)

> **[SHELL]** + **[DIALOG]**

### Step 0.1 — 创建 Next.js 项目

```bash
PROJECT_NAME="my-web-app"
DISPLAY_NAME="我的Web应用"

cat > /tmp/sop_web.env << WEBENV
PROJECT_NAME="$PROJECT_NAME"
DISPLAY_NAME="$DISPLAY_NAME"
PROJECT_DIR="$(pwd)/$PROJECT_NAME"
WEBENV

# [SHELL] 用 create-next-app 创建项目 (TypeScript + Tailwind + App Router)
npx create-next-app@latest ${PROJECT_NAME} \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --use-pnpm \
  --no-turbopack
```

### Step 0.2 — 安装核心依赖

```bash
source /tmp/sop_web.env 2>/dev/null && cd ${PROJECT_DIR}

# [SHELL] 数据库 + 认证 + UI 组件
pnpm add prisma @prisma/client next-auth@beta bcryptjs
pnpm add -D @types/bcryptjs vitest @playwright/test

# [SHELL] 初始化 Prisma
npx prisma init --datasource-provider sqlite

# [SHELL] 安装 shadcn/ui (基于 Tailwind 的组件库)
npx shadcn-ui@latest init -y
npx shadcn-ui@latest add button card input label dialog dropdown-menu toast
```

### Step 0.3 — 首次构建验证

```bash
npm run build 2>&1 | tail -10
# [DEBUG] 修复构建错误

git init && git checkout -b main
cat > .gitignore << 'GITIGNORE'
node_modules/
.next/
*.db
.env
.env.local
GITIGNORE
git add -A && git commit -m "Initial Next.js project with Prisma + shadcn/ui"
```

---

## Phase 1: 产品需求确认 (0:30-1:00)

> **[DIALOG]** 同其他 SOP — 结构化问卷 → SPECS.md

```
确认: App 类型 / 是否需要用户系统 / 是否需要数据库 / 是否需要支付 / SEO 需求
```

---

## Phase 1.2: PRD 产品需求文档 (1:00-1:30)

> **[GENERATE]** + **[WRITE]** 7 章 PRD (同 iOS SOP Phase 1.2 模板)

---

## Phase 1.5: 高保真原型 (1:30-2:30)

> **[GENERATE]** Tailwind + shadcn/ui 快速原型，浏览器实时预览

```tsx
// src/app/prototype/page.tsx — 原型页面 (仅开发阶段, 不上线)
import { Button } from "@/components/ui/button"
import { Card } from "@/components/ui/card"
import { Input } from "@/components/ui/input"

export default function HomePrototype() {
  const filters = ["全部", "进行中", "已完成"]
  const tasks = [
    { title: "完成需求文档", priority: "高", dueDate: "6月15日" },
    { title: "设计首页布局", priority: "中", dueDate: "6月18日" },
    { title: "代码审查", priority: "高", dueDate: "6月12日" },
  ]

  return (
    <div className="min-h-screen bg-gray-50">
      <header className="bg-white shadow-sm p-4">
        <h1 className="text-2xl font-bold">我的任务</h1>
      </header>
      <div className="flex gap-2 p-4 overflow-x-auto">
        {filters.map((f, i) => (
          <Button key={f} variant={i === 0 ? "default" : "outline"} size="sm">{f}</Button>
        ))}
      </div>
      <div className="px-4 space-y-3">
        {tasks.map((t, i) => (
          <Card key={i} className="p-4 flex items-center gap-3">
            <input type="checkbox" className="w-5 h-5" />
            <div className="flex-1">
              <p className="font-medium">{t.title}</p>
              <p className="text-sm text-orange-500">截止 {t.dueDate}</p>
            </div>
            <span className="text-xs bg-red-100 text-red-600 px-2 py-0.5 rounded">
              {t.priority}
            </span>
          </Card>
        ))}
      </div>
    </div>
  )
}
```

> `npm run dev` → 浏览器打开 `http://localhost:3000/prototype` → [DIALOG] 用户确认

---

## Phase 2: 数据库设计 (2:30-3:00)

### Prisma Schema

> **[WRITE]** `prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  password  String
  createdAt DateTime @default(now())
  tasks     Task[]
}

model Task {
  id          String    @id @default(cuid())
  title       String
  description String?
  priority    Int       @default(1)   // 0=LOW 1=MEDIUM 2=HIGH 3=URGENT
  isCompleted Boolean   @default(false)
  dueDate     DateTime?
  createdAt   DateTime  @default(now())
  userId      String
  user        User      @relation(fields: [userId], references: [id])
}
```

```bash
# [SHELL] .env 配置
echo 'DATABASE_URL="file:./dev.db"' > .env

# [SHELL] 生成迁移 + Prisma Client
npx prisma migrate dev --name init
npx prisma generate
```

### 数据库服务层

> **[WRITE]** `src/lib/db.ts`:

```typescript
import { PrismaClient } from "@prisma/client"

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient }

export const prisma = globalForPrisma.prisma || new PrismaClient()

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma
```

---

## Phase 3: API 层 & Server Actions (3:00-4:00)

### Server Actions (Next.js 14 推荐方式)

> **[WRITE]** `src/actions/task-actions.ts`:

```typescript
"use server"

import { revalidatePath } from "next/cache"
import { prisma } from "@/lib/db"

export async function getTasks(userId: string) {
  return prisma.task.findMany({
    where: { userId },
    orderBy: { createdAt: "desc" }
  })
}

export async function createTask(formData: FormData) {
  const title = formData.get("title") as string
  if (!title?.trim()) throw new Error("标题不能为空")

  await prisma.task.create({
    data: {
      title,
      userId: "default-user" // Phase 5 改为真实用户ID
    }
  })
  revalidatePath("/")
}

export async function toggleTask(id: string, completed: boolean) {
  await prisma.task.update({
    where: { id },
    data: { isCompleted: completed }
  })
  revalidatePath("/")
}

export async function deleteTask(id: string) {
  await prisma.task.delete({ where: { id } })
  revalidatePath("/")
}
```

---

## Phase 4: UI 组件 & 页面 (4:00-7:00)

### TaskCard 组件

> **[WRITE]** `src/components/task-card.tsx`:

```tsx
"use client"

import { Card } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { toggleTask, deleteTask } from "@/actions/task-actions"
import { cn } from "@/lib/utils"

const priorityConfig = {
  3: { label: "紧急", color: "bg-red-100 text-red-700" },
  2: { label: "高",   color: "bg-orange-100 text-orange-700" },
  1: { label: "中",   color: "bg-blue-100 text-blue-700" },
  0: { label: "低",   color: "bg-gray-100 text-gray-600" },
}

export function TaskCard({ task }: { task: any }) {
  const p = priorityConfig[task.priority as keyof typeof priorityConfig] || priorityConfig[1]

  return (
    <Card className="p-4 flex items-center gap-3 hover:shadow-md transition-shadow">
      <input
        type="checkbox"
        checked={task.isCompleted}
        onChange={async (e) => { await toggleTask(task.id, e.target.checked) }}
        className="w-5 h-5 rounded accent-blue-500"
      />
      <div className="flex-1 min-w-0">
        <p className={cn("font-medium truncate", task.isCompleted && "line-through text-gray-400")}>
          {task.title}
        </p>
        {task.dueDate && !task.isCompleted && (
          <p className="text-sm text-orange-500">
            截止 {new Date(task.dueDate).toLocaleDateString("zh-CN")}
          </p>
        )}
      </div>
      <span className={cn("text-xs px-2 py-0.5 rounded-full", p.color)}>{p.label}</span>
      <form action={deleteTask.bind(null, task.id)}>
        <Button variant="ghost" size="icon" type="submit">🗑️</Button>
      </form>
    </Card>
  )
}
```

### 首页

> **[WRITE]** `src/app/page.tsx`:

```tsx
import { getTasks } from "@/actions/task-actions"
import { TaskCard } from "@/components/task-card"
import { AddTaskForm } from "@/components/add-task-form"
import { FilterBar } from "@/components/filter-bar"

export default async function HomePage() {
  const tasks = await getTasks("default-user")

  return (
    <div className="min-h-screen bg-gray-50">
      <header className="bg-white shadow-sm sticky top-0 z-10">
        <div className="max-w-2xl mx-auto px-4 py-3">
          <h1 className="text-xl font-bold">我的任务</h1>
        </div>
      </header>

      <main className="max-w-2xl mx-auto p-4 space-y-4">
        <FilterBar />
        <AddTaskForm />

        {tasks.length === 0 ? (
          <div className="text-center py-16 text-gray-400">
            <p className="text-lg">📋 暂无任务</p>
            <p className="text-sm mt-2">在上方输入框添加你的第一个任务</p>
          </div>
        ) : (
          <div className="space-y-3">
            {tasks.map((task) => (
              <TaskCard key={task.id} task={task} />
            ))}
          </div>
        )}
      </main>
    </div>
  )
}
```

### 依赖组件

```tsx
// src/components/add-task-form.tsx
import { Input } from "@/components/ui/input"
import { Button } from "@/components/ui/button"
import { createTask } from "@/actions/task-actions"

export function AddTaskForm() {
  return (
    <form action={createTask} className="flex gap-2">
      <Input name="title" placeholder="添加新任务..." required className="flex-1" />
      <Button type="submit">添加</Button>
    </form>
  )
}

// src/components/filter-bar.tsx
"use client"
import { Button } from "@/components/ui/button"
import { useState } from "react"

export function FilterBar() {
  const [active, setActive] = useState("all")
  const filters = [
    { key: "all", label: "全部" },
    { key: "active", label: "进行中" },
    { key: "completed", label: "已完成" },
  ]
  return (
    <div className="flex gap-2">
      {filters.map((f) => (
        <Button key={f.key} variant={active === f.key ? "default" : "outline"} size="sm"
          onClick={() => setActive(f.key)}>{f.label}</Button>
      ))}
    </div>
  )
}
```

---

## Phase 5: 认证系统 (Day 2, 0:30-2:00)

> **[WRITE]** NextAuth.js 配置:

```typescript
// src/auth.ts
import NextAuth from "next-auth"
import Credentials from "next-auth/providers/credentials"
import { prisma } from "@/lib/db"
import bcrypt from "bcryptjs"

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [
    Credentials({
      credentials: { email: {}, password: {} },
      async authorize(credentials) {
        const user = await prisma.user.findUnique({
          where: { email: credentials.email as string }
        })
        if (!user) return null
        const valid = await bcrypt.compare(credentials.password as string, user.password)
        return valid ? { id: user.id, email: user.email, name: user.name } : null
      }
    })
  ],
  pages: { signIn: "/login" }
})
```

---

## Phase 6: 安全 & 合规审计 (Day 2, 2:00-3:00)

> **[VALIDATE]** Claude Code 逐项检查安全/合规，不通过不进入 Phase 7

### 安全检查

```bash
# [SHELL] 依赖漏洞扫描
pnpm audit --audit-level=high

# [SHELL] 环境变量检查 (确保无 .env 提交)
git check-ignore .env .env.local && echo "✅ .env 已 gitignored" || echo "❌ 立即添加!"

# [SHELL] Secret 扫描 (检查代码中是否有硬编码密钥)
grep -rn "sk-\|api_key\|secret\|password" src/ --include="*.ts" --include="*.tsx" | grep -v "//\|example\|test\|\.env" | head -5
```

```typescript
// ✅ 安全实践:
// 1. CSP (Content Security Policy) — 在 next.config.js 配置
const securityHeaders = [
  { key: 'X-Content-Type-Options', value: 'nosniff' },
  { key: 'X-Frame-Options', value: 'DENY' },
  { key: 'X-XSS-Protection', value: '1; mode=block' },
  { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' }
]

// 2. Server Actions 输入验证
// 3. 无 dangerouslySetInnerHTML (如需使用 → DOMPurify)
```

### 合规检查

```
□ 隐私政策页面可访问 (/privacy)
□ Cookie 同意弹窗 (如使用分析/广告 cookies)
□ 联系邮箱可访问
□ GDPR 合规 (欧盟用户): 数据删除权、数据导出权
□ 无障碍 (Accessibility): WCAG 2.1 AA 标准
```

### 代码质量

```bash
# [SHELL] ESLint
pnpm lint 2>&1 | tail -5

# [SHELL] TypeScript 严格检查
pnpm tsc --noEmit 2>&1 | tail -5

# [SHELL] Prettier 格式检查
pnpm prettier --check "src/**/*.{ts,tsx}"
```

---

## Phase 7: 测试 & 性能 & SEO (Day 2, 3:00-5:30)

### Vitest 单元测试

> **[WRITE]** `src/__tests__/task-actions.test.ts`:

```typescript
import { describe, it, expect, vi } from "vitest"
import { createTask, getTasks } from "@/actions/task-actions"
import { prisma } from "@/lib/db"

vi.mock("@/lib/db", () => ({
    prisma: { task: { create: vi.fn(), findMany: vi.fn() } }
}))

describe("createTask", () => {
  it("should reject empty title", async () => {
    const formData = new FormData()
    formData.set("title", "")
    await expect(createTask(formData)).rejects.toThrow("标题不能为空")
  })
})
```

### Playwright E2E

```typescript
// e2e/home.spec.ts
import { test, expect } from "@playwright/test"

test("add and complete task flow", async ({ page }) => {
  await page.goto("http://localhost:3000")
  await page.fill("input[name=title]", "Buy groceries")
  await page.click("button[type=submit]")
  await expect(page.locator("text=Buy groceries")).toBeVisible()
  await page.click("input[type=checkbox]")
  await expect(page.locator("text=Buy groceries")).toHaveClass(/line-through/)
})
```

### 性能优化

```tsx
// 图片优化
import Image from "next/image"  // 自动 WebP + 懒加载

// 字体优化
import { Inter } from "next/font/google"
const inter = Inter({ subsets: ["latin"], display: "swap" })

// 动态导入 (代码分割)
const HeavyComponent = dynamic(() => import("@/components/heavy"), {
  loading: () => <Skeleton />
})

// ISR (增量静态再生成)
export const revalidate = 60  // 60 秒重新验证
```

### SEO 元数据

```tsx
// app/layout.tsx
export const metadata: Metadata = {
  title: { default: "我的App", template: "%s | 我的App" },
  description: "高效管理日常任务",
  openGraph: { title: "我的App", description: "高效管理日常任务", type: "website" }
}

// app/page.tsx — 动态 sitemap
// app/sitemap.ts → 自动生成 sitemap.xml
// app/robots.ts → 自动生成 robots.txt
```

---

## Phase 8: 部署 (Day 2, 5:30-7:30)

### Vercel 部署 (推荐)

```bash
# [SHELL] 推送代码
git add -A && git commit -m "Ready for production"
git push origin main

# [DIALOG] 用户在 Vercel 操作:
# 1. vercel.com → Import Git Repository
# 2. 自动检测 Next.js → Framework Preset: Next.js
# 3. 环境变量: DATABASE_URL (Vercel Postgres 自动注入)
# 4. Deploy → 获得域名: xxx.vercel.app
```

### Vercel Postgres (数据库)

```bash
# [SHELL] Vercel CLI
npx vercel link          # 链接项目
npx vercel env pull      # 拉取环境变量
npx vercel postgres create  # 创建 PostgreSQL
# 自动注入: POSTGRES_URL, POSTGRES_PRISMA_URL

# 生产迁移:
npx prisma migrate deploy
```

### 自定义域名 & 监控

```bash
# [SHELL] 域名 + 分析
# Vercel → Settings → Domains → 添加 yourdomain.com
# Vercel → Analytics → Enable (免费层: 2500 events/月)
# Vercel → Speed Insights → Enable (Core Web Vitals)
```

### CLI 一键部署

```bash
npx vercel --prod          # 生产部署
npx vercel --prod --env DATABASE_URL=xxx  # 带环境变量
```

---

## Phase 9: 前后端联调 & E2E 验证 (Day 2, 7:30-8:00)

### 联调配置

```bash
# [SHELL] 启动后端 (终端1)
cd ../my-api-server && pnpm dev

# [SHELL] 启动前端 (终端2) 
cd ../my-web-app && pnpm dev

# .env.local 配置后端地址
NEXT_PUBLIC_API_URL=http://localhost:3000/api/v1
```

### API 客户端对接

```typescript
// [WRITE] src/lib/api.ts — 对接 Backend SOP 的后端
const API_URL = process.env.NEXT_PUBLIC_API_URL

export async function fetchAPI<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
  const token = typeof window !== "undefined" ? localStorage.getItem("token") : null
  const res = await fetch(`${API_URL}${endpoint}`, {
    ...options,
    headers: { "Content-Type": "application/json", ...(token ? { Authorization: `Bearer ${token}` } : {}), ...options.headers }
  })
  const json = await res.json()
  if (!json.success) throw new Error(json.error?.message || "Request failed")
  return json.data
}

// 替换 Server Actions 为后端 API 调用:
// createTask → fetchAPI("/tasks", { method: "POST", body: JSON.stringify(data) })
```

### E2E 验证清单

```
□ 首页加载 < 2s (Lighthouse Performance ≥ 90)
□ 创建任务 → 列表刷新 → 数据持久化
□ 筛选/搜索功能正常
□ 完成任务 → 删除线显示
□ 删除任务 → 列表更新
□ 响应式 (手机/平板/桌面 均正常)
□ SEO 元数据正确 (title/description/OG)
□ sitemap.xml 可访问
□ 后端 API 健康检查通过
```

---

## 📚 附录

### A. 命令速查

```bash
npm run dev              # 开发服务器 (localhost:3000)
npm run build            # 生产构建
npm run start            # 生产启动
npx prisma studio        # 数据库可视化管理
npx prisma migrate dev   # 数据库迁移
npx tsc --noEmit         # TypeScript 类型检查
npx eslint .             # 代码规范检查
```

### B. 技术栈对比 (Web vs iOS vs 鸿蒙 vs Android)

| 概念 | Web (Next.js) | iOS | 鸿蒙 | Android |
|------|-------------|-----|------|---------|
| 语言 | TypeScript | Swift | ArkTS | Kotlin |
| UI | React/JSX | SwiftUI | ArkUI | Compose |
| 路由 | App Router | NavigationStack | Navigation | NavHost |
| 状态 | useState/Server Actions | @State | @State | StateFlow |
| 存储 | Prisma/SQLite | SwiftData | RelationalStore | Room |
| 认证 | NextAuth.js | Sign in with Apple | Account Kit | Firebase Auth |
| 部署 | Vercel | App Store | AppGallery | Google Play |

### C. 常见错误速查

| 错误 | 原因 | 修复 |
|------|------|------|
| `Module not found: Can't resolve 'xxx'` | 依赖未安装 | `pnpm add xxx` |
| `Type 'xxx' is not assignable to type 'yyy'` | 类型不匹配 | 检查 TypeScript 类型 |
| `PrismaClientInitializationError` | 数据库未迁移 | `npx prisma migrate dev` |
| `Hydration failed` | 服务端/客户端 HTML 不一致 | 检查 `use client` 边界 |
| `revalidatePath not working` | 未在 Server Action | 确保文件顶部有 `"use server"` |
| `CORS error` (前后端联调) | 后端未配置前端 origin | Backend SOP: cors({ origin: [...] }) |
| `Build failed: window is not defined` | 服务端组件用了浏览器 API | 加 `"use client"` 或 `useEffect` |

### D. 文件依赖关系

```
Phase 0   项目骨架
Phase 1   SPECS.md
Phase 1.2 PRD.md
Phase 1.5 原型 (prototype/page.tsx)
Phase 2   ┌─ prisma/schema.prisma
         └─ src/lib/db.ts
Phase 3   └─ src/actions/task-actions.ts  ← 依赖 db
Phase 4   ┌─ src/components/task-card.tsx  ← 依赖 actions
         ├─ src/components/add-task-form.tsx
         ├─ src/components/filter-bar.tsx
         └─ src/app/page.tsx              ← 依赖 components + actions
Phase 5   └─ src/auth.ts                  ← 依赖 db
Phase 6-7 测试 + 性能
Phase 8   Vercel 部署
Phase 9   前后端联调 (对接 Backend SOP)
```

---

> **SOP 版本**: 1.0.0 | **最后更新**: 2026-06-08
> **技术栈**: Next.js 14+ + TypeScript + Tailwind CSS + Prisma + SQLite
> **关联文档**: 可接入 `../pipeline/` 完成软著申请
