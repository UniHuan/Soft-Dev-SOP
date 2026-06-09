# 🔌 Claude Code + Hono 两天全自动后端服务开发 SOP

> **适用对象**: Claude Code (AI Agent) 全自动执行
> **目标**: 从零到生产部署。可与任意前端联调，前后端配合后完整上线
> **技术栈**: Hono (API框架) + TypeScript + Prisma + PostgreSQL + Zod + JWT
> **部署**: Docker → Railway / Render / VPS
> **最低要求**: Node.js 20+, pnpm, Docker Desktop, 云服务账号

---

## 🧠 Claude Code 技能调用矩阵

| 技能标识 | 技能名称 | 说明 | Claude Code 工具 |
|---------|---------|------|-----------------|
| `[SHELL]` | Shell 执行 | npm/pnpm、docker、curl、git | `Bash` |
| `[WRITE]` | 文件写入 | .ts .prisma .yml .env 等 | `Edit` |
| `[READ]` | 文件读取 | 日志、配置 | `Read` |
| `[DIALOG]` | 用户交互 | 提问、确认 | 对话 |
| `[GENERATE]` | 代码生成 | TypeScript/Hono 代码 | `Edit` |
| `[REVIEW]` | 代码审查 | 审查代码质量 | `Read`→分析 |
| `[DEBUG]` | 调试分析 | 编译/运行时错误修复 | `Read`+`Bash`+`Edit` |
| `[VALIDATE]` | 验证检查 | API 测试、清单验证 | `Bash`+`Read` |
| `[GIT]` | 版本控制 | git add/commit/tag | `Bash` |

---

## 📋 总览时间线

```
Day 1 (8h)                              Day 2 (8h)
├─ [0.0h] 环境 & 项目初始化              ├─ [0.0h] Day 2 启动 & API 测试
├─ [0.5h] 需求确认 & API 设计            ├─ [1.0h] 认证授权 (JWT + 中间件)
├─ [1.5h] 数据库设计 (Prisma Schema)    ├─ [2.5h] 文件上传 & 存储
├─ [2.5h] 核心 CRUD API 实现            ├─ [3.5h] 日志 & 错误处理 & 限流
├─ [5.0h] 请求验证 (Zod) + 错误处理     ├─ [5.0h] Docker 容器化
├─ [6.5h] Swagger/OpenAPI 文档          ├─ [6.0h] 部署 (Railway/Render)
└─ [8.0h] 自测 & 前端联调准备           └─ [8.0h] 归档 & 监控配置
```

---

## ⚠️ 执行前提

```bash
# [SHELL] 检查环境
node -v | grep "v2[0-9]" || { echo "❌ 需要 Node.js 20+"; exit 1; }
pnpm -v 2>/dev/null || npm i -g pnpm
docker --version 2>/dev/null || echo "⚠️ Docker 未安装 (部署时需要)"
git --version || exit 1

# [DIALOG] 确认:
# 数据库: PostgreSQL (推荐) 还是 SQLite (快速原型)?
# 部署目标: Railway / Render / 自有 VPS?
```

---

## Phase 0: 项目初始化 (0:00-0:30)

> **[SHELL]** 创建 Hono 项目

```bash
PROJECT_NAME="my-api-server"

cat > /tmp/sop_backend.env << BACKENV
PROJECT_NAME="$PROJECT_NAME"
PROJECT_DIR="$(pwd)/$PROJECT_NAME"
API_PORT=3000
BACKENV

# [SHELL] 用 pnpm 创建项目
mkdir ${PROJECT_NAME} && cd ${PROJECT_NAME}
pnpm init
pnpm add hono @hono/zod-validator zod @prisma/client bcryptjs jsonwebtoken
pnpm add -D typescript @types/node @types/bcryptjs @types/jsonwebtoken prisma tsx vitest
pnpm add -D @hono/swagger-ui @scalar/hono-api-reference  # API 文档

# [SHELL] TypeScript 配置
cat > tsconfig.json << 'TSCEOF'
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["src/**/*"]
}
TSCEOF

# [SHELL] 初始化 Prisma
npx prisma init --datasource-provider postgresql

# 目录结构
mkdir -p src/{routes,middleware,services,lib,types}

# [SHELL] 首次编译
npx tsc --noEmit 2>&1 | tail -5
git init && git checkout -b main
git add -A && git commit -m "Initial Hono + Prisma project"
```

---

## Phase 1: API 设计 & 数据库 (0:30-1:30)

### Step 1.1 — API 设计文档

> **[WRITE]** `API_SPEC.md`:

```markdown
# API 规格书

## 基础信息
- Base URL: `http://localhost:3000/api/v1`
- 认证: Bearer JWT Token
- 格式: JSON

## 端点清单
| 方法 | 路径 | 认证 | 说明 |
|------|------|------|------|
| POST | /auth/register | ❌ | 用户注册 |
| POST | /auth/login | ❌ | 用户登录 → 返回 JWT |
| GET | /tasks | ✅ | 获取任务列表 (支持 ?status=active&search=xxx) |
| POST | /tasks | ✅ | 创建任务 |
| GET | /tasks/:id | ✅ | 获取任务详情 |
| PATCH | /tasks/:id | ✅ | 更新任务 |
| DELETE | /tasks/:id | ✅ | 删除任务 |
| GET | /health | ❌ | 健康检查 |

## 通用响应格式
```json
{ "success": true, "data": {...} }
{ "success": false, "error": { "code": "VALIDATION_ERROR", "message": "..." } }
```
```

### Step 1.2 — Prisma Schema

> **[WRITE]** `prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  password  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  tasks     Task[]
}

model Task {
  id          String    @id @default(cuid())
  title       String
  description String?
  priority    Int       @default(1)
  isCompleted Boolean   @default(false)
  dueDate     DateTime?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  userId      String
  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([userId, isCompleted])
}

model RefreshToken {
  id        String   @id @default(cuid())
  token     String   @unique
  userId    String
  expiresAt DateTime
  createdAt DateTime @default(now())
}
```

---

## Phase 2: 核心 CRUD API (1:30-5:00)

### Step 2.1 — 应用入口

> **[WRITE]** `src/index.ts`:

```typescript
import { Hono } from "hono"
import { cors } from "hono/cors"
import { logger } from "hono/logger"
import { authRoutes } from "./routes/auth"
import { taskRoutes } from "./routes/tasks"
import { healthRoute } from "./routes/health"

const app = new Hono().basePath("/api/v1")

// 全局中间件
app.use("*", cors({
  origin: ["http://localhost:5173", "http://localhost:3000"], // 前端地址
  credentials: true
}))
app.use("*", logger())

// 路由
app.route("/", healthRoute)
app.route("/auth", authRoutes)
app.route("/tasks", taskRoutes)

// 全局错误处理
app.onError((err, c) => {
  console.error(`[ERROR] ${err.message}`)
  return c.json({
    success: false,
    error: { code: "INTERNAL_ERROR", message: err.message }
  }, 500)
})

// 404
app.notFound((c) => c.json({
  success: false,
  error: { code: "NOT_FOUND", message: "Route not found" }
}, 404))

export default app
```

### Step 2.2 — 数据库 & 工具

> **[WRITE]** `src/lib/db.ts`:

```typescript
import { PrismaClient } from "@prisma/client"

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient }
export const prisma = globalForPrisma.prisma || new PrismaClient()
if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma
```

> **[WRITE]** `src/lib/jwt.ts`:

```typescript
import jwt from "jsonwebtoken"

const SECRET = process.env.JWT_SECRET || "dev-secret-change-in-production"

export function signToken(payload: { userId: string }): string {
  return jwt.sign(payload, SECRET, { expiresIn: "7d" })
}

export function verifyToken(token: string): { userId: string } {
  return jwt.verify(token, SECRET) as { userId: string }
}
```

> **[WRITE]** `src/lib/response.ts`:

```typescript
import type { Context } from "hono"

export function success<T>(c: Context, data: T, status = 200) {
  return c.json({ success: true, data }, status as any)
}

export function error(c: Context, code: string, message: string, status = 400) {
  return c.json({ success: false, error: { code, message } }, status as any)
}

export function paginated<T>(c: Context, data: T[], total: number, page: number, limit: number) {
  return c.json({
    success: true,
    data,
    pagination: { total, page, limit, totalPages: Math.ceil(total / limit) }
  })
}
```

### Step 2.3 — 任务 CRUD

> **[WRITE]** `src/routes/tasks.ts`:

```typescript
import { Hono } from "hono"
import { zValidator } from "@hono/zod-validator"
import { z } from "zod"
import { prisma } from "../lib/db"
import { authMiddleware } from "../middleware/auth"
import { success, error } from "../lib/response"

export const taskRoutes = new Hono()

// 所有任务路由需要认证
taskRoutes.use("*", authMiddleware)

// GET /tasks — 获取任务列表 (支持筛选+搜索+分页)
taskRoutes.get("/", async (c) => {
  const userId = c.get("userId") as string
  const status = c.req.query("status")          // active | completed
  const search = c.req.query("search") || ""
  const page = parseInt(c.req.query("page") || "1")
  const limit = parseInt(c.req.query("limit") || "20")

  const where: any = { userId }
  if (status === "active") where.isCompleted = false
  else if (status === "completed") where.isCompleted = true
  if (search) where.title = { contains: search }

  const [tasks, total] = await Promise.all([
    prisma.task.findMany({
      where,
      orderBy: { createdAt: "desc" },
      skip: (page - 1) * limit,
      take: limit
    }),
    prisma.task.count({ where })
  ])

  return c.json({ success: true, data: tasks, pagination: { total, page, limit, totalPages: Math.ceil(total / limit) } })
})

// POST /tasks — 创建任务
taskRoutes.post("/", zValidator("json", z.object({
  title: z.string().min(1).max(200),
  description: z.string().max(1000).optional(),
  priority: z.number().int().min(0).max(3).default(1),
  dueDate: z.string().datetime().optional()
})), async (c) => {
  const userId = c.get("userId") as string
  const body = c.req.valid("json")

  const task = await prisma.task.create({
    data: { ...body, userId, dueDate: body.dueDate ? new Date(body.dueDate) : null }
  })
  return success(c, task, 201)
})

// PATCH /tasks/:id — 更新任务
taskRoutes.patch("/:id", zValidator("json", z.object({
  title: z.string().min(1).max(200).optional(),
  description: z.string().max(1000).optional().nullable(),
  priority: z.number().int().min(0).max(3).optional(),
  isCompleted: z.boolean().optional(),
  dueDate: z.string().datetime().optional().nullable()
})), async (c) => {
  const userId = c.get("userId") as string
  const id = c.req.param("id")
  const body = c.req.valid("json")

  // 验证任务归属
  const existing = await prisma.task.findUnique({ where: { id } })
  if (!existing || existing.userId !== userId) {
    return error(c, "NOT_FOUND", "Task not found", 404)
  }

  const updated = await prisma.task.update({
    where: { id },
    data: { ...body, dueDate: body.dueDate !== undefined ? (body.dueDate ? new Date(body.dueDate) : null) : undefined }
  })
  return success(c, updated)
})

// DELETE /tasks/:id — 删除任务
taskRoutes.delete("/:id", async (c) => {
  const userId = c.get("userId") as string
  const id = c.req.param("id")

  const existing = await prisma.task.findUnique({ where: { id } })
  if (!existing || existing.userId !== userId) {
    return error(c, "NOT_FOUND", "Task not found", 404)
  }

  await prisma.task.delete({ where: { id } })
  return success(c, { deleted: true })
})
```

### Step 2.4 — 认证路由

> **[WRITE]** `src/routes/auth.ts`:

```typescript
import { Hono } from "hono"
import { zValidator } from "@hono/zod-validator"
import { z } from "zod"
import bcrypt from "bcryptjs"
import { prisma } from "../lib/db"
import { signToken } from "../lib/jwt"
import { success, error } from "../lib/response"
import { authMiddleware } from "../middleware/auth"

export const authRoutes = new Hono()

// POST /auth/register
authRoutes.post("/register", zValidator("json", z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100),
  name: z.string().min(1).max(50).optional()
})), async (c) => {
  const { email, password, name } = c.req.valid("json")

  const existing = await prisma.user.findUnique({ where: { email } })
  if (existing) return error(c, "EMAIL_EXISTS", "Email already registered", 409)

  const hashed = await bcrypt.hash(password, 10)
  const user = await prisma.user.create({
    data: { email, password: hashed, name },
    select: { id: true, email: true, name: true, createdAt: true }
  })
  const token = signToken({ userId: user.id })
  return success(c, { user, token }, 201)
})

// POST /auth/login
authRoutes.post("/login", zValidator("json", z.object({
  email: z.string().email(),
  password: z.string().min(1)
})), async (c) => {
  const { email, password } = c.req.valid("json")

  const user = await prisma.user.findUnique({ where: { email } })
  if (!user) return error(c, "INVALID_CREDENTIALS", "Invalid email or password", 401)

  const valid = await bcrypt.compare(password, user.password)
  if (!valid) return error(c, "INVALID_CREDENTIALS", "Invalid email or password", 401)

  const token = signToken({ userId: user.id })
  return success(c, {
    user: { id: user.id, email: user.email, name: user.name },
    token
  })
})

// GET /auth/me — 获取当前用户
authRoutes.get("/me", authMiddleware, async (c) => {
  const userId = c.get("userId") as string
  const user = await prisma.user.findUnique({
    where: { id: userId },
    select: { id: true, email: true, name: true, createdAt: true }
  })
  if (!user) return error(c, "NOT_FOUND", "User not found", 404)
  return success(c, user)
})
```

### Step 2.5 — 认证中间件 & 健康检查

> **[WRITE]** `src/middleware/auth.ts`:

```typescript
import type { Context, Next } from "hono"
import { verifyToken } from "../lib/jwt"

export async function authMiddleware(c: Context, next: Next) {
  const header = c.req.header("Authorization")
  if (!header?.startsWith("Bearer ")) {
    return c.json({ success: false, error: { code: "UNAUTHORIZED", message: "Missing token" } }, 401)
  }
  try {
    const payload = verifyToken(header.slice(7))
    c.set("userId", payload.userId)
    await next()
  } catch {
    return c.json({ success: false, error: { code: "UNAUTHORIZED", message: "Invalid token" } }, 401)
  }
}
```

> **[WRITE]** `src/routes/health.ts`:

```typescript
import { Hono } from "hono"
import { prisma } from "../lib/db"

export const healthRoute = new Hono()

healthRoute.get("/health", async (c) => {
  try {
    await prisma.$queryRaw`SELECT 1`
    return c.json({ status: "healthy", timestamp: new Date().toISOString(), uptime: process.uptime() })
  } catch {
    return c.json({ status: "unhealthy", database: "disconnected" }, 503)
  }
})
```

---

## Phase 3: 后端本地启动 & 前端联调 (5:00-8:00)

### Step 3.1 — 启动脚本

```json
// package.json scripts
{
  "dev": "tsx watch src/index.ts",
  "build": "tsc",
  "start": "node dist/index.js",
  "db:migrate": "prisma migrate dev",
  "db:studio": "prisma studio",
  "db:seed": "tsx prisma/seed.ts",
  "test": "vitest run",
  "test:watch": "vitest"
}
```

```bash
# [SHELL] 启动后端
source /tmp/sop_backend.env 2>/dev/null && cd ${PROJECT_DIR}

# 1. 数据库迁移
npx prisma migrate dev --name init

# 2. 启动开发服务器
pnpm dev &
sleep 2

# 3. 验证 API
echo "=== 健康检查 ==="
curl -s http://localhost:3000/api/v1/health | jq

echo ""
echo "=== 注册测试用户 ==="
curl -s -X POST http://localhost:3000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test123456","name":"Test User"}' | jq

echo ""
echo "=== 创建任务 (带 Token) ==="
# 保存 Token
TOKEN=$(curl -s -X POST http://localhost:3000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test123456"}' | jq -r '.data.token')

curl -s -X POST http://localhost:3000/api/v1/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"title":"第一个任务","priority":2}' | jq

echo ""
echo "✅ 后端启动成功! 前端请配置:"
echo "   API_BASE_URL=http://localhost:3000/api/v1"
```

### Step 3.2 — 前端联调配置

> **[DIALOG]** 指导用户在前端项目中配置 API 地址:

```typescript
// 前端项目 .env.local (Next.js) 或 .env (Vite)
NEXT_PUBLIC_API_URL=http://localhost:3000/api/v1
```

```typescript
// 前端 API 客户端封装示例
// src/lib/api.ts
const API_URL = process.env.NEXT_PUBLIC_API_URL || "http://localhost:3000/api/v1"

async function fetchAPI<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
  const token = localStorage.getItem("token")
  const res = await fetch(`${API_URL}${endpoint}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options.headers,
    },
  })
  const json = await res.json()
  if (!json.success) throw new Error(json.error?.message || "Request failed")
  return json.data
}

// 使用示例
export const api = {
  tasks: {
    list: (params?: string) => fetchAPI(`/tasks?${params}`),
    create: (data: any) => fetchAPI("/tasks", { method: "POST", body: JSON.stringify(data) }),
    update: (id: string, data: any) => fetchAPI(`/tasks/${id}`, { method: "PATCH", body: JSON.stringify(data) }),
    delete: (id: string) => fetchAPI(`/tasks/${id}`, { method: "DELETE" }),
  },
  auth: {
    login: (data: any) => fetchAPI("/auth/login", { method: "POST", body: JSON.stringify(data) }),
    register: (data: any) => fetchAPI("/auth/register", { method: "POST", body: JSON.stringify(data) }),
  }
}
```

### Step 3.3 — Swagger 文档

> **[WRITE]** `src/routes/docs.ts` — 添加 API 文档端点:

```typescript
import { Hono } from "hono"
import { swaggerUI } from "@hono/swagger-ui"

export const docsRoute = new Hono()

docsRoute.get("/docs", swaggerUI({ url: "/api/v1/openapi.json" }))
docsRoute.get("/openapi.json", (c) => {
  return c.json({
    openapi: "3.0.0",
    info: { title: "Task API", version: "1.0.0" },
    servers: [{ url: "http://localhost:3000/api/v1" }],
    paths: {
      "/health": { get: { summary: "Health check", responses: { "200": { description: "OK" } } } },
      "/auth/register": { post: { summary: "Register", /* ... */ } },
      "/auth/login": { post: { summary: "Login", /* ... */ } },
      "/tasks": {
        get: { summary: "List tasks", parameters: [/* ... */], security: [{ bearerAuth: [] }] },
        post: { summary: "Create task", security: [{ bearerAuth: [] }] }
      },
    }
  })
})
```

---

## Phase 4: 安全 & 合规审计 (Day 2, 4:00-5:00)

> **[VALIDATE]** 后端安全是系统最后一道防线，逐项检查不得跳过

### 安全检查

```bash
# [SHELL] 依赖漏洞扫描
pnpm audit --audit-level=high

# [SHELL] 环境变量检查
test -f .env.example && echo "✅ .env.example 存在" || echo "❌ 创建 .env.example!"
git check-ignore .env && echo "✅ .env 已 gitignored" || echo "❌ 立即添加!"

# [SHELL] Secret 扫描
grep -rn "sk-\|password\|secret\|api_key" src/ --include="*.ts" | grep -v "process\.env\|example\|test" | head -5
```

### 安全清单

```
□ 密码: bcrypt (cost ≥ 10)

□ JWT: 过期 ≤ 7天, 使用环境变量 JWT_SECRET (非硬编码)
□ CORS: 仅允许已知前端域名 (非 allowOrigin: "*")
□ Rate Limiting: 已配置限流 (100 req/min)
□ Helmet: 已添加安全 headers (XSS/CSRF/点击劫持防护)
□ SQL 注入: Prisma 参数化查询 (无需额外防护)
□ 输入验证: Zod schema 覆盖所有端点
□ HTTPS: 强制 (生产环境)
```

```typescript
// ✅ Rate Limiting (Hono)
import { rateLimiter } from "hono-rate-limiter"
app.use(rateLimiter({ windowMs: 60_000, max: 100, message: "Too many requests" }))

// ✅ Helmet-like headers
app.use("*", async (c, next) => {
  c.res.headers.set("X-Content-Type-Options", "nosniff")
  c.res.headers.set("X-Frame-Options", "DENY")
  await next()
})
```

### 合规检查

```
□ 日志不记录密码/Token/PII (个人身份信息)
□ 数据库备份策略: 每日自动备份
□ 数据保留策略: 用户可请求删除所有数据
□ 隐私政策: 声明数据收集/使用/存储方式
□ 生产环境: NODE_ENV=production
```

### 代码质量

```bash
# [SHELL] TypeScript 严格检查
pnpm tsc --noEmit

# [SHELL] ESLint
pnpm eslint src/ --ext .ts

# [SHELL] 测试覆盖率
pnpm vitest --coverage
# 目标: ≥ 70% 行覆盖率
```

---

## Phase 5: Docker 容器化 (Day 2, 5:00-6:00)

> **[WRITE]** `Dockerfile`:

```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm i -g pnpm && pnpm install --frozen-lockfile --prod

FROM base AS builder
RUN pnpm install --frozen-lockfile
COPY . .
RUN npx prisma generate && pnpm build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/package.json ./
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

> **[WRITE]** `docker-compose.yml`:

```yaml
version: "3.8"
services:
  api:
    build: .
    ports: ["3000:3000"]
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/mydb
      JWT_SECRET: ${JWT_SECRET:-change-me-in-production}
      NODE_ENV: production
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: mydb
    volumes: [pgdata:/var/lib/postgresql/data]
    ports: ["5432:5432"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

```bash
# [SHELL] Docker 本地测试
docker compose up -d
sleep 5
curl http://localhost:3000/api/v1/health

# 数据库迁移
docker compose exec api npx prisma migrate deploy
```

---

## Phase 5: 生产部署 (Day 2, 6:00-7:30)

### 部署方案 A: Railway (推荐, 最简单)

```bash
# [SHELL] Railway CLI 部署
npm i -g @railway/cli
railway login
railway init
railway up

# Railway 自动检测 Dockerfile → 构建 → 部署
# 添加 PostgreSQL 插件: railway add -d postgresql
# 环境变量自动注入 DATABASE_URL
```

### 部署方案 B: Render (免费层)

```yaml
# render.yaml
services:
  - type: web
    name: my-api
    env: docker
    plan: free
    envVars:
      - key: DATABASE_URL
        fromDatabase:
          name: my-db
          property: connectionString
      - key: JWT_SECRET
        generateValue: true

databases:
  - name: my-db
    plan: free
```

### 部署方案 C: VPS (Docker Compose)

```bash
# [SHELL] 在 VPS 上一键部署
ssh user@vps "mkdir -p ~/app"
scp docker-compose.yml user@vps:~/app/
scp -r . user@vps:~/app/
ssh user@vps "cd ~/app && docker compose up -d"
```

---

## Phase 6: 归档 & 监控 (Day 2, 7:30-8:00)

```bash
# [GIT]
git tag v1.0.0
git add -A && git commit -m "Release v1.0.0 - API server ready for production"

# 监控建议:
# 1. 接入 Sentry (错误追踪): npx @sentry/wizard -i nextjs
# 2. 接入 UptimeRobot (可用性监控, 免费)
# 3. 配置日志: pino (结构化日志) 替代 console.log
```

---

## 📚 附录

### A. 前后端联调检查清单

```
□ 后端 CORS 已配置前端地址
□ API Base URL 前端已配置 (开发环境: localhost:3000)
□ 注册/登录 API 可用 (curl 测试通过)
□ CRUD API 可用 (带 Token 测试通过)
□ 错误响应格式统一 ({ success, error })
□ Swagger 文档可访问 (http://localhost:3000/api/v1/docs)
□ Docker Compose 本地运行正常
□ 环境变量 .env.example 已创建 (不含密钥)
```

### B. 技术栈对比

| 概念 | Hono (后端) | Express | Fastify |
|------|-----------|---------|---------|
| 性能 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| TypeScript | 原生 | 需 @types | 原生 |
| 验证 | 内置 Zod | 需中间件 | 需插件 |
| 体积 | ~15KB | ~200KB | ~50KB |
| 生态 | 快速成长 | 最成熟 | 成熟 |
| 学习曲线 | 低 | 最低 | 中 |

### C. 与前端 Web SOP 配合

```
Web SOP (Next.js) ←→ Backend SOP (Hono)
     │                       │
     ├─ API 调用 → fetchAPI() → http://localhost:3000/api/v1/tasks
     ├─ 认证    → /auth/login → JWT Token → localStorage
     ├─ 部署    → Vercel      → Railway/Render
     └─ 数据库  → (无)        → PostgreSQL
```

---

> **SOP 版本**: 1.0.0 | **最后更新**: 2026-06-08
> **技术栈**: Hono + TypeScript + Prisma + PostgreSQL + JWT + Docker
> **前端配合**: 可与 `../web/Web_App_2Day_Development_SOP.md` 直接联调
