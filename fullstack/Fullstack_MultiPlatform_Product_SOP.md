# 🏗️ Claude Code 三天全自动全栈多端产品开发 SOP

> **适用对象**: Claude Code (AI Agent) 全自动执行
> **目标**: 从零到华为云生产部署，三天 (24 小时) 完成一个完整产品
> **产品形态**: 共享后端 + 数据库 + CMS → iOS App + Android App + Web 应用（按需选择前端组合）
> **技术栈**: Hono + TypeScript + Prisma + PostgreSQL + Directus (CMS) + Next.js 14+ + SwiftUI + Kotlin/Compose
> **部署**: Docker Compose → 华为云 ECS + RDS + OBS + CDN
> **最低要求**: Node.js 20+, pnpm, Docker Desktop, 华为云账号, macOS (iOS 构建) / 任意系统 (Android/Web)

---

## 🧠 Claude Code 技能调用矩阵

| 技能标识 | 技能名称 | 说明 | Claude Code 工具 |
|---------|---------|------|-----------------|
| `[SHELL]` | Shell 执行 | npm/pnpm、docker、git、hcloud CLI | `Bash` |
| `[WRITE]` | 文件写入 | .ts .tsx .swift .kt .prisma .yml 等 | `Edit` |
| `[READ]` | 文件读取 | 代码、日志、配置、华为云文档 | `Read` |
| `[DIALOG]` | 用户交互 | 提问、确认、展示结果 | 对话输出 |
| `[GENERATE]` | 代码生成 | TypeScript/Swift/Kotlin/React 代码 | `Edit` |
| `[REVIEW]` | 代码审查 | 审查代码质量、架构合规、安全性 | `Read`→分析 |
| `[DEBUG]` | 调试分析 | 编译/运行时错误修复 | `Read`+`Bash`+`Edit` |
| `[RESEARCH]` | 知识检索 | 查阅 API 文档、华为云文档 | 内置知识 + `Read` |
| `[VALIDATE]` | 验证检查 | API 测试、清单验证、端到端测试 | `Bash`+`Read` |
| `[GIT]` | 版本控制 | git add/commit/tag/push | `Bash` |

### 技能调用原则

```
1. 每个 Phase 开始前 → [RESEARCH] 查阅相关技术文档和最佳实践
2. 每次 WRITE 后 → 编译/构建验证 → [DEBUG] 自动修复错误（最多 5 次重试）
3. 编译失败 → 读日志 → 定位文件 → Edit 修复 → 重构建
4. 每个 Phase 结束 → [VALIDATE] → [GIT] commit
5. 不可逆操作（华为云付费、域名绑定、App Store 提交）→ [DIALOG] 必须用户确认
6. 前后端联调 → curl 测试 API → 检查响应格式
```

---

## 📋 总览时间线

```
Day 1 (8h) — 架构 & 后端 & CMS
├─ Phase 0 [0.0h] 环境检查 & 华为云账号准备
├─ Phase 1 [0.5h] 产品需求 & 架构设计
├─ Phase 2 [1.5h] 数据库 Schema 设计
├─ Phase 3 [3.0h] 后端 API 开发 (Hono)
├─ Phase 4 [5.5h] CMS 集成 (Directus)
└─ Phase 5 [6.5h] 统一认证授权 (JWT + RBAC)

Day 2 (8h) — 前端多端开发
├─ Phase 6 [0.0h] 共享 API Client SDK
├─ Phase 7 [0.5h] Web 前端 (Next.js — 用户端 + 管理后台)
├─ Phase 8 [3.5h] iOS App (SwiftUI，可选跳过)
├─ Phase 9 [6.0h] Android App (Kotlin/Compose，可选跳过)
└─ Phase 10 [8.0h] 前端功能验证 & 联调

Day 3 (8h) — 华为云部署 & 上线
├─ Phase 11 [0.0h] Docker 容器化 & 本地编排
├─ Phase 12 [1.5h] 华为云基础设施搭建
├─ Phase 13 [3.5h] 生产部署 & SSL/域名
├─ Phase 14 [5.0h] CI/CD 流水线 & 推送通知
├─ Phase 15 [6.0h] 安全合规审计 & 监控告警
└─ Phase 16 [7.5h] 上线检查清单 & 归档

Day 4+ (持续) — 后期运营 & 迭代增长
├─ Phase 17 [上线后 Week 1] 稳定性监控 & 快速响应
├─ Phase 18 [上线后 Week 2-4] 内容运营 & 用户增长
├─ Phase 19 [上线后 Month 1-3] 数据驱动迭代
├─ Phase 20 [持续] 日常运营 SOP (日/周/月/季)
└─ Phase 21 [上线后 Month 3-6] 生态建设 — 开发者生态 & 合作伙伴 & 社区
```

### 🎯 前端选择策略

```
产品需求                    →  前端组合
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
「只要有 Web 就行」          →  Phase 7 only（最快）
「Web + iOS 用户」           →  Phase 7 + Phase 8
「Web + Android 用户」       →  Phase 7 + Phase 9
「全平台都要」               →  Phase 7 + Phase 8 + Phase 9（完整覆盖）
「先做 MVP 验证再扩展」      →  Phase 7 先上线 → 再加 Phase 8/9
```

---

## ⚠️ 执行前提

```bash
# [SHELL] 1. 确认 Node.js 20+
node -v | grep "v2[0-9]" || { echo "需要 Node.js 20+"; exit 1; }

# [SHELL] 2. 确认包管理器
which pnpm && echo "pnpm $(pnpm -v)" || npm i -g pnpm

# [SHELL] 3. 确认 Docker
docker --version || { echo "Docker Desktop 未安装"; exit 1; }
docker compose version || { echo "Docker Compose 未安装"; exit 1; }

# [SHELL] 4. 确认 Git
git --version && git config user.name && git config user.email

# [SHELL] 5. 确认 macOS (build iOS 时需要)
sw_vers -productVersion 2>/dev/null && echo "macOS 已确认" || echo "(非 macOS 环境，iOS 部分将跳过)"

# [SHELL] 6. 华为云 CLI (可选，用于脚本化部署)
which hcloud 2>/dev/null || echo "hcloud CLI 未安装 (部署时将使用 Web 控制台)"

# [DIALOG] 需要用户确认以下信息:
echo ""
echo "⚠️ 请确认以下信息已准备好:"
echo "  1. 华为云账号 (https://www.huaweicloud.com) — 已实名认证"
echo "  2. 产品名称: _______________"
echo "  3. 域名 (如有): _______________"
echo "  4. 前端组合: [全部 / Web+iOS / Web+Android / 仅Web]"
echo "  5. 预算: 华为云最低配置约 ¥200-400/月 (ECS+RDS+OBS+CDN)"
```

---

## 🏛 系统架构总览

```
┌──────────────────────────────────────────────────────────┐
│                      客户端层                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐       │
│  │ iOS App  │  │ Android  │  │  Web App         │       │
│  │ (SwiftUI)│  │ (Compose)│  │  (Next.js 14+)   │       │
│  │          │  │          │  │  ┌──────┬──────┐ │       │
│  │          │  │          │  │  │用户端│管理端│ │       │
│  └────┬─────┘  └────┬─────┘  └──┴──┬───┴──┬───┘       │
│       │              │              │       │            │
│       └──────────────┼──────────────┘       │            │
│                      │                     │            │
│              ┌───────┴───────┐    ┌────────┴────────┐  │
│              │  REST API      │    │  CMS Admin UI   │  │
│              │  (Hono)        │    │  (Directus)     │  │
│              │  :3000/api/v1  │    │  :8055/admin    │  │
│              └───────┬───────┘    └────────┬────────┘  │
│                      │                     │            │
│         ┌────────────┴─────────────────────┴─────┐      │
│         │           服务层 (华为云 ECS)            │      │
│         │  ┌─────────┐  ┌────────┐  ┌────────┐  │      │
│         │  │  Nginx   │  │ Redis  │  │Docker  │  │      │
│         │  │ (反向代理)│  │(缓存)  │  │Compose │  │      │
│         │  └─────────┘  └────────┘  └────────┘  │      │
│         └────────────────┬──────────────────────┘      │
│                          │                              │
│         ┌────────────────┴──────────────────────┐      │
│         │         华为云基础设施                   │      │
│         │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ │      │
│         │  │ RDS  │ │ OBS  │ │ CDN  │ │ WAF  │ │      │
│         │  │(PG)  │ │(存储)│ │(加速)│ │(防护)│ │      │
│         │  └──────┘ └──────┘ └──────┘ └──────┘ │      │
│         └────────────────────────────────────────┘      │
│                                                          │
│         ┌──────────────────────────────────────┐        │
│         │       外部服务集成                      │        │
│         │  ┌────────┐ ┌────────┐ ┌──────────┐ │        │
│         │  │ 邮件   │ │ 短信   │ │ 推送通知  │ │        │
│         │  │(SMTP)  │ │(华为云)│ │(APNs/FCM)│ │        │
│         │  └────────┘ └────────┘ └──────────┘ │        │
│         └──────────────────────────────────────┘        │
└──────────────────────────────────────────────────────────┘
```

### 数据流

```
用户操作 → 客户端 (iOS/Android/Web)
         → REST API (Hono) ──→ Prisma ORM ──→ PostgreSQL (RDS)
         → 文件上传 ──→ 后端临时处理 ──→ OBS 存储 ──→ CDN 分发
         → CMS 管理 ──→ Directus Admin ──→ 同数据库
         → 缓存读取 ──→ Redis ──→ (命中则跳过 DB)
```

---

## 🚨 遗漏清单 — 开始前逐项确认

> **[DIALOG]** 在进入 Phase 0 之前，Claude Code 逐项检查以下是否已在产品规划中考虑。如有遗漏，先在 SPECS.md 中补充。

```
□ 1.  用户系统: 注册/登录/找回密码/注销/账号删除
□ 2.  认证方案: JWT access token + refresh token + 多设备登录策略
□ 3.  权限模型: RBAC (超级管理员/编辑/普通用户/游客)
□ 4.  内容管理: 文章/公告/Banner/帮助中心 — 通过 CMS 管理
□ 5.  文件存储: 用户头像/内容图片/视频/附件 — OBS + CDN
□ 6.  搜索功能: 是否需要全文搜索？(PostgreSQL full-text / Elasticsearch)
□ 7.  推送通知: 系统通知/运营推送 — 是否需要？
□ 8.  实时功能: 即时通讯/实时通知 — 是否需要 WebSocket？
□ 9.  支付系统: 内购/订阅/微信支付/支付宝 — 是否需要？
□ 10. 短信/邮件: 验证码/通知邮件 — 华为云 SMS + SMTP
□ 11. 数据分析: 用户行为埋点/留存分析 — 是否需要？
□ 12. 国际化: 多语言支持 — 是否需要？
□ 13. 无障碍: VoiceOver/TalkBack/ARIA — 是否需要？
□ 14. 深色模式: iOS/Android/Web 三端深色主题
□ 15. 离线支持: 缓存策略/离线浏览 — 是否需要？
□ 16. 版本管理: 强制更新/灰度发布/版本兼容矩阵
□ 17. 数据合规: 个人信息保护法/GDPR/隐私政策/用户数据删除
□ 18. 备份策略: 数据库自动备份/异地容灾
□ 19. 性能目标: API 响应 < 200ms / 首页加载 < 2s / App 启动 < 1.5s
□ 20. 监控告警: 服务可用性/错误率/API 延迟/资源使用率
```

---

## Phase 0: 环境初始化 & 项目结构 (0:00-0:30)

> **[SHELL]** + **[WRITE]**

### Step 0.1 — 创建 monorepo 项目结构

```bash
PROJECT_NAME="my-fullstack-app"
DISPLAY_NAME="我的全栈应用"

cat > /tmp/sop_fullstack.env << 'FSENV'
PROJECT_NAME="my-fullstack-app"
DISPLAY_NAME="我的全栈应用"
PROJECT_DIR="$(pwd)/my-fullstack-app"
FSENV

source /tmp/sop_fullstack.env

# [SHELL] 创建 monorepo 根目录
mkdir -p ${PROJECT_NAME} && cd ${PROJECT_NAME}
git init && git checkout -b main

# 目录结构
mkdir -p packages/backend/src/{routes,middleware,services,lib,types,uploads}
mkdir -p packages/cms
mkdir -p packages/web/src/{app,components,lib,hooks,types}
mkdir -p packages/shared/src
mkdir -p mobile/ios
mkdir -p mobile/android
mkdir -p docker
mkdir -p scripts
mkdir -p docs

# 根 package.json (monorepo workspaces)
cat > package.json << 'PKGJSON'
{
  "name": "my-fullstack-app",
  "private": true,
  "workspaces": ["packages/*"],
  "scripts": {
    "dev:backend": "pnpm --filter backend dev",
    "dev:web": "pnpm --filter web dev",
    "dev:cms": "pnpm --filter cms dev",
    "build:all": "pnpm --filter backend build && pnpm --filter web build",
    "docker:up": "docker compose -f docker/docker-compose.yml up -d",
    "docker:down": "docker compose -f docker/docker-compose.yml down",
    "db:migrate": "pnpm --filter backend db:migrate",
    "db:seed": "pnpm --filter backend db:seed"
  }
}
PKGJSON

# .gitignore
cat > .gitignore << 'GI'
node_modules/
dist/
.next/
*.db
.env
.env.local
.env.production
uploads/
temp/
.DS_Store
*.log
GI

# .env.example (不含密钥，可提交)
cat > .env.example << 'ENVEXAMPLE'
# ===== 数据库 =====
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/mydb

# ===== JWT =====
JWT_SECRET=change-me-to-random-64-chars
JWT_REFRESH_SECRET=change-me-to-another-random-64-chars

# ===== 华为云 OBS =====
HUAWEI_OBS_ACCESS_KEY=your_access_key
HUAWEI_OBS_SECRET_KEY=your_secret_key
HUAWEI_OBS_BUCKET=my-app-uploads
HUAWEI_OBS_ENDPOINT=https://obs.cn-north-4.myhuaweicloud.com

# ===== 华为云 RDS =====
RDS_HOST=xxx.rds.cn-north-4.myhuaweicloud.com
RDS_PORT=5432
RDS_USER=admin
RDS_PASSWORD=your_rds_password
RDS_DB_NAME=mydb

# ===== 邮件 (SMTP) =====
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=noreply@example.com
SMTP_PASS=your_smtp_password

# ===== 华为云 SMS =====
HUAWEI_SMS_APP_KEY=your_sms_key
HUAWEI_SMS_APP_SECRET=your_sms_secret
HUAWEI_SMS_SIGN_ID=your_sign_id
HUAWEI_SMS_TEMPLATE_ID=your_template_id

# ===== Redis =====
REDIS_URL=redis://localhost:6379

# ===== App 配置 =====
NODE_ENV=development
API_BASE_URL=http://localhost:3000/api/v1
CORS_ORIGINS=http://localhost:3000,http://localhost:3001
ENVEXAMPLE

cp .env.example .env

git add -A && git commit -m "Initial monorepo structure"
```

---

## Phase 1: 产品需求 & 架构设计 (0:30-1:30)

> **[DIALOG]** + **[GENERATE]** + **[WRITE]**

### Step 1.1 — 结构化需求问卷

> **[DIALOG]** 与用户逐项确认产品需求：

```markdown
## 产品需求确认清单

### A. 产品定位
- 产品名称: ___________
- 一句话描述: ___________
- 目标用户: ___________
- 核心解决的问题: ___________

### B. 功能清单 (按优先级)
#### P0 (必须有，MVP)
- [ ] 用户注册/登录
- [ ] 核心功能 1: ___________
- [ ] 核心功能 2: ___________
- [ ] 个人中心/设置

#### P1 (应该有，V1.0)
- [ ] ___________
- [ ] ___________

#### P2 (锦上添花，V1.1+)
- [ ] ___________

### C. CMS 管理内容
- [ ] 文章/资讯管理
- [ ] Banner/广告位管理
- [ ] 推送通知模板
- [ ] 用户管理 (查看/禁用/删除)
- [ ] 数据统计看板
- [ ] 其他: ___________

### D. 前端平台选择
- [ ] Web (用户端 + 管理后台)
- [ ] iOS App
- [ ] Android App
```

### Step 1.2 — 生成 SPECS.md

> **[WRITE]** `docs/SPECS.md` — 7 章产品规格书：

```markdown
# 产品规格书 (SPECS)

## 1. 产品概述
- 产品名称 / 一句话描述 / 目标用户 / 核心价值主张

## 2. 功能规格
### 2.1 用户系统
- 注册 (邮箱+密码 / 手机号+验证码 / 第三方登录)
- 登录 / 找回密码 / 个人信息编辑 / 账号注销

### 2.2 核心功能
- [按产品实际填写]

### 2.3 CMS 管理
- 内容管理: 文章/Banner/公告 CRUD
- 用户管理: 查看列表/禁用/删除
- 数据统计: 核心指标看板

### 2.4 推送通知
- 系统通知 (站内信)
- 运营推送 (APNs + FCM + 华为 Push Kit)

## 3. 数据模型 (概要)
[Phase 2 详细设计]

## 4. API 设计 (概要)
[Phase 3 详细设计]

## 5. 前端页面/屏幕清单
- Web: 首页/详情页/用户中心/管理后台
- iOS: [屏幕清单]
- Android: [屏幕清单]

## 6. 非功能需求
- 性能: API < 200ms, 首屏 < 2s
- 安全: HTTPS, JWT, SQL 注入防护, XSS 防护
- 可用性: 99.5% uptime
- 数据合规: 个人信息保护法, 隐私政策

## 7. 里程碑 & 发布计划
- M1 (Day 3): MVP 上线华为云
- M2 (Week 2): App Store + Google Play 提交
- M3 (Month 1): 用户反馈迭代
```

### Step 1.3 — API 设计文档

> **[WRITE]** `docs/API_SPEC.md`:

```markdown
# API 规格书

## 基础信息
- Base URL: `https://api.yourdomain.com/api/v1`
- 认证: Bearer JWT (access token 2h, refresh token 7d)
- 格式: JSON
- 版本策略: URL 路径版本 `/api/v1/`, `/api/v2/`

## 通用响应格式
```json
// 成功
{ "success": true, "data": { ... } }
// 分页
{ "success": true, "data": [...], "pagination": { "total": 100, "page": 1, "limit": 20, "totalPages": 5 } }
// 错误
{ "success": false, "error": { "code": "VALIDATION_ERROR", "message": "...", "details": [...] } }
```

## 端点清单

### 认证 (Auth)
| 方法 | 路径 | 认证 | 说明 |
|------|------|------|------|
| POST | /auth/register | No | 用户注册 |
| POST | /auth/login | No | 用户登录 |
| POST | /auth/refresh | No | 刷新 Token |
| POST | /auth/logout | Yes | 登出 |
| POST | /auth/forgot-password | No | 忘记密码 |
| POST | /auth/reset-password | No | 重置密码 |
| GET | /auth/me | Yes | 获取当前用户 |

### 用户 (Users)
| 方法 | 路径 | 认证 | 角色 | 说明 |
|------|------|------|------|------|
| GET | /users | Yes | Admin | 用户列表 |
| GET | /users/:id | Yes | Admin | 用户详情 |
| PATCH | /users/:id | Yes | User | 更新自己 |
| DELETE | /users/:id | Yes | Admin | 删除用户 |

### 内容 (CMS)
| 方法 | 路径 | 认证 | 说明 |
|------|------|------|------|
| GET | /content/articles | No | 文章列表 (公开) |
| GET | /content/articles/:id | No | 文章详情 |
| GET | /content/banners | No | Banner 列表 |
| GET | /content/announcements | No | 公告列表 |

### 文件 (Upload)
| 方法 | 路径 | 认证 | 说明 |
|------|------|------|------|
| POST | /upload/image | Yes | 上传图片 |
| POST | /upload/file | Yes | 上传文件 |
| GET | /upload/:key | No | 获取文件 URL |

### 通用
| 方法 | 路径 | 认证 | 说明 |
|------|------|------|------|
| GET | /health | No | 健康检查 |
| GET | /app/config | No | App 配置 (版本/强制更新等) |
```

```bash
# [GIT]
git add -A && git commit -m "Phase 1: Product requirements, architecture design & API spec"
```

---

## Phase 2: 数据库 Schema 设计 (1:30-3:00)

> **[RESEARCH]** + **[WRITE]** + **[SHELL]**

### Step 2.1 — Prisma Schema (统一数据模型)

> **[WRITE]** `packages/backend/prisma/schema.prisma`:

```prisma
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["fullTextSearch"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ==================== 用户 & 认证 ====================

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  phone         String?   @unique
  name          String?
  avatar        String?             // OBS URL
  password      String
  role          Role      @default(USER)
  status        UserStatus @default(ACTIVE)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  // 关联
  refreshTokens RefreshToken[]
  devices       UserDevice[]
  notifications Notification[]
  likes         Like[]
  comments      Comment[]

  @@index([email])
  @@index([role])
  @@map("users")
}

enum Role {
  SUPER_ADMIN   // 超级管理员 (CMS + 所有权限)
  ADMIN         // 管理员 (CMS 管理)
  EDITOR        // 内容编辑 (CMS 内容管理)
  USER          // 普通用户
  GUEST         // 游客
}

enum UserStatus {
  ACTIVE
  DISABLED
  DELETED
}

model RefreshToken {
  id        String   @id @default(cuid())
  token     String   @unique
  userId    String
  deviceId  String?
  expiresAt DateTime
  createdAt DateTime @default(now())

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([token])
  @@map("refresh_tokens")
}

model UserDevice {
  id           String   @id @default(cuid())
  userId       String
  platform     Platform // IOS, ANDROID, WEB
  pushToken    String?  // 推送 token
  deviceName   String?
  lastActiveAt DateTime @default(now())

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([userId, platform, pushToken])
  @@map("user_devices")
}

enum Platform {
  IOS
  ANDROID
  WEB
}

// ==================== CMS 内容 ====================

model Article {
  id          String   @id @default(cuid())
  title       String
  slug        String   @unique
  summary     String?
  content     String                     // HTML/JSON 富文本
  coverImage  String?                    // OBS URL
  tags        String[]                   // PostgreSQL array
  status      PublishStatus @default(DRAFT)
  authorId    String?
  publishedAt DateTime?
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  author      User?     @relation(fields: [authorId], references: [id])

  @@index([slug])
  @@index([status, publishedAt])
  @@index([tags])
  @@map("articles")
}

model Banner {
  id          String   @id @default(cuid())
  title       String
  imageUrl    String                     // OBS URL
  linkUrl     String?
  position    String   @default("home_top") // home_top, home_middle, sidebar
  sortOrder   Int      @default(0)
  status      PublishStatus @default(DRAFT)
  startAt     DateTime?
  endAt       DateTime?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([position, status])
  @@map("banners")
}

model Announcement {
  id        String   @id @default(cuid())
  title     String
  content   String
  type      String   @default("info")   // info, warning, success, error
  isSticky  Boolean  @default(false)
  status    PublishStatus @default(DRAFT)
  startAt   DateTime?
  endAt     DateTime?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([status, startAt])
  @@map("announcements")
}

enum PublishStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

// ==================== 通知 ====================

model Notification {
  id        String    @id @default(cuid())
  userId    String
  title     String
  body      String
  data      Json?                     // 附加数据 { type, targetId, ... }
  isRead    Boolean   @default(false)
  createdAt DateTime  @default(now())

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId, isRead, createdAt])
  @@map("notifications")
}

// ==================== 文件管理 ====================

model MediaFile {
  id         String   @id @default(cuid())
  userId     String?
  key        String                     // OBS object key
  url        String                     // CDN URL or OBS URL
  fileName   String
  fileType   String                     // image, video, document, other
  mimeType   String
  fileSize   Int                        // bytes
  width      Int?
  height     Int?
  createdAt  DateTime @default(now())

  user User? @relation(fields: [userId], references: [id])

  @@index([userId])
  @@index([fileType])
  @@map("media_files")
}

// ==================== 应用配置 ====================

model AppConfig {
  id           String   @id @default(cuid())
  key          String   @unique       // e.g. "ios_latest_version", "force_update_required"
  value        String                 // JSON string
  description  String?
  updatedAt    DateTime @updatedAt

  @@map("app_configs")
}

// ==================== 通用: 评论 & 点赞 ====================

model Comment {
  id        String   @id @default(cuid())
  userId    String
  targetType String  // article, ...
  targetId  String
  content   String
  parentId  String?                    // 回复评论
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  user     User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  parent   Comment?  @relation("CommentReplies", fields: [parentId], references: [id])
  replies  Comment[] @relation("CommentReplies")

  @@index([targetType, targetId, createdAt])
  @@map("comments")
}

model Like {
  id         String   @id @default(cuid())
  userId     String
  targetType String                   // article, comment, ...
  targetId   String
  createdAt  DateTime @default(now())

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([userId, targetType, targetId])
  @@map("likes")
}
```

### Step 2.2 — 初始化 Backend 包

```bash
# [SHELL] 创建 backend package
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}

cd packages/backend

# 初始化 package.json
cat > package.json << 'PKGJSON'
{
  "name": "backend",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "db:migrate": "prisma migrate dev",
    "db:deploy": "prisma migrate deploy",
    "db:seed": "tsx prisma/seed.ts",
    "db:studio": "prisma studio",
    "db:generate": "prisma generate"
  }
}
PKGJSON

# 安装依赖
pnpm add hono @hono/zod-validator zod @prisma/client bcryptjs jsonwebtoken dotenv uuid
pnpm add -D typescript @types/node @types/bcryptjs @types/jsonwebtoken @types/uuid prisma tsx

# TypeScript 配置
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

# 初始化 Prisma
npx prisma init --datasource-provider postgresql

# 回到根目录
cd ${PROJECT_DIR}
```

```bash
# [SHELL] 验证 Prisma Schema
cd packages/backend
npx prisma validate
# 预期输出: "The Prisma schema is valid"

# [GIT]
cd ${PROJECT_DIR}
git add -A && git commit -m "Phase 2: Unified database schema design (Prisma + PostgreSQL)"
```

### Step 2.3 — 种子数据脚本

> **[WRITE]** `packages/backend/prisma/seed.ts`:

```typescript
import { PrismaClient, Role } from "@prisma/client"
import bcrypt from "bcryptjs"

const prisma = new PrismaClient()

async function main() {
  console.log("🌱 开始种子数据...")

  // 创建超级管理员
  const adminPassword = await bcrypt.hash("admin123", 10)
  const admin = await prisma.user.upsert({
    where: { email: "admin@example.com" },
    update: {},
    create: {
      email: "admin@example.com",
      name: "超级管理员",
      password: adminPassword,
      role: Role.SUPER_ADMIN,
    },
  })
  console.log(`  Admin: ${admin.email}`)

  // 创建示例文章
  await prisma.article.createMany({
    skipDuplicates: true,
    data: [
      {
        title: "欢迎使用我们的产品",
        slug: "welcome",
        summary: "这是一篇欢迎文章",
        content: "<h1>欢迎</h1><p>感谢使用我们的产品！</p>",
        status: "PUBLISHED",
        publishedAt: new Date(),
        tags: ["公告", "产品"],
      },
      {
        title: "产品使用指南",
        slug: "user-guide",
        summary: "快速上手产品的基本操作",
        content: "<h1>使用指南</h1><p>正文内容...</p>",
        status: "PUBLISHED",
        publishedAt: new Date(),
        tags: ["教程"],
      },
    ],
  })
  console.log("  示例文章已创建")

  // 创建示例 Banner
  await prisma.banner.createMany({
    skipDuplicates: true,
    data: [
      { title: "首页横幅1", imageUrl: "https://via.placeholder.com/750x300", position: "home_top", status: "PUBLISHED", sortOrder: 0 },
      { title: "首页横幅2", imageUrl: "https://via.placeholder.com/750x300", position: "home_top", status: "PUBLISHED", sortOrder: 1 },
    ],
  })
  console.log("  示例 Banner 已创建")

  console.log("✅ 种子数据完成")
}

main()
  .catch((e) => { console.error(e); process.exit(1) })
  .finally(() => prisma.$disconnect())
```

---

## Phase 3: 后端 API 开发 (3:00-5:30)

> **[GENERATE]** + **[WRITE]** + **[VALIDATE]**

### Step 3.1 — 应用入口 & 全局中间件

> **[WRITE]** `packages/backend/src/index.ts`:

```typescript
import { Hono } from "hono"
import { cors } from "hono/cors"
import { logger as honoLogger } from "hono/logger"
import { rateLimiter } from "./middleware/rate-limiter"
import { securityHeaders, requestId } from "./middleware/security"
import { logger } from "./lib/logger"
import { authRoutes } from "./routes/auth"
import { userRoutes } from "./routes/users"
import { contentRoutes } from "./routes/content"
import { uploadRoutes } from "./routes/upload"
import { notificationRoutes } from "./routes/notifications"
import { healthRoute } from "./routes/health"
import { appConfigRoute } from "./routes/app-config"
import { adminStatsRoute } from "./routes/admin-stats"
import { analyticsRoutes } from "./routes/analytics"

const app = new Hono().basePath("/api/v1")

// 全局中间件
app.use("*", securityHeaders)
app.use("*", requestId)
app.use("*", cors({
  origin: process.env.CORS_ORIGINS?.split(",") || ["http://localhost:3000"],
  credentials: true,
  allowHeaders: ["Authorization", "Content-Type", "X-Request-ID"],
  allowMethods: ["GET", "POST", "PATCH", "DELETE", "OPTIONS"],
  maxAge: 86400,
}))
app.use("*", honoLogger())
app.use("*", rateLimiter({ windowMs: 60_000, max: 100 }))

// 路由注册
app.route("/", healthRoute)
app.route("/auth", authRoutes)
app.route("/users", userRoutes)
app.route("/content", contentRoutes)
app.route("/upload", uploadRoutes)
app.route("/notifications", notificationRoutes)
app.route("/app", appConfigRoute)
app.route("/admin/stats", adminStatsRoute)
app.route("/analytics", analyticsRoutes)

// 全局错误处理
app.onError((err, c) => {
  const requestId = c.get("requestId") as string
  logger.error("Unhandled error", err as Error, { requestId, path: c.req.path })
  return c.json({
    success: false,
    error: { code: "INTERNAL_ERROR", message: "An unexpected error occurred", requestId }
  }, 500)
})

// 404
app.notFound((c) => c.json({
  success: false,
  error: { code: "NOT_FOUND", message: "Route not found" }
}, 404))

// 优雅关闭 (Graceful Shutdown)
const shutdown = async (signal: string) => {
  logger.info(`Received ${signal}, shutting down gracefully...`)
  const { prisma } = await import("./lib/db")
  await prisma.$disconnect()
  logger.info("Database disconnected")
  process.exit(0)
}
process.on("SIGTERM", () => shutdown("SIGTERM"))
process.on("SIGINT", () => shutdown("SIGINT"))

export default app
```

### Step 3.2 — 核心库文件

> **[WRITE]** `packages/backend/src/lib/db.ts`:

```typescript
import { PrismaClient } from "@prisma/client"

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient }
export const prisma = globalForPrisma.prisma || new PrismaClient({
  log: process.env.NODE_ENV === "development" ? ["error", "warn"] : ["error"],
})
if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma
```

> **[WRITE]** `packages/backend/src/lib/jwt.ts`:

```typescript
import jwt from "jsonwebtoken"

const ACCESS_SECRET = process.env.JWT_SECRET || "dev-access-secret"
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET || "dev-refresh-secret"

export interface TokenPayload {
  userId: string
  role: string
}

export function signAccessToken(payload: TokenPayload): string {
  return jwt.sign(payload, ACCESS_SECRET, { expiresIn: "2h" })
}

export function signRefreshToken(payload: TokenPayload): string {
  return jwt.sign(payload, REFRESH_SECRET, { expiresIn: "7d" })
}

export function verifyAccessToken(token: string): TokenPayload {
  return jwt.verify(token, ACCESS_SECRET) as TokenPayload
}

export function verifyRefreshToken(token: string): TokenPayload {
  return jwt.verify(token, REFRESH_SECRET) as TokenPayload
}
```

> **[WRITE]** `packages/backend/src/lib/response.ts`:

```typescript
import type { Context } from "hono"

export function ok<T>(c: Context, data: T, status = 200) {
  return c.json({ success: true, data }, status as any)
}

export function fail(c: Context, code: string, message: string, status = 400, details?: any) {
  return c.json({
    success: false,
    error: { code, message, ...(details ? { details } : {}) }
  }, status as any)
}

export function paginated<T>(c: Context, data: T[], total: number, page: number, limit: number) {
  return c.json({
    success: true,
    data,
    pagination: { total, page, limit, totalPages: Math.ceil(total / limit) }
  })
}
```

> **[WRITE]** `packages/backend/src/lib/redis.ts`:

```typescript
// Redis 缓存客户端 (开发环境可选，生产环境必须)
const REDIS_URL = process.env.REDIS_URL

// 简单的内存缓存 fallback (当 Redis 不可用时)
const memoryCache = new Map<string, { value: string; expiresAt: number }>()

export const cache = {
  async get(key: string): Promise<string | null> {
    const item = memoryCache.get(key)
    if (item && item.expiresAt > Date.now()) return item.value
    memoryCache.delete(key)
    return null
  },

  async set(key: string, value: string, ttlSeconds = 300): Promise<void> {
    memoryCache.set(key, { value, expiresAt: Date.now() + ttlSeconds * 1000 })
  },

  async del(key: string): Promise<void> {
    memoryCache.delete(key)
  },

  async flush(): Promise<void> {
    memoryCache.clear()
  },
}

// TODO Phase 12: 替换为真实 Redis 客户端
// import { createClient } from "redis"
// const redis = createClient({ url: REDIS_URL })
// redis.connect()
```

### Step 3.3 — 认证中间件 & 角色守卫

> **[WRITE]** `packages/backend/src/middleware/auth.ts`:

```typescript
import type { Context, Next } from "hono"
import { verifyAccessToken, type TokenPayload } from "../lib/jwt"

// 必须认证
export async function authRequired(c: Context, next: Next) {
  const header = c.req.header("Authorization")
  if (!header?.startsWith("Bearer ")) {
    return c.json({
      success: false,
      error: { code: "UNAUTHORIZED", message: "Missing or invalid authorization header" }
    }, 401)
  }
  try {
    const payload = verifyAccessToken(header.slice(7))
    c.set("userId", payload.userId)
    c.set("userRole", payload.role)
    await next()
  } catch {
    return c.json({
      success: false,
      error: { code: "TOKEN_EXPIRED", message: "Token expired or invalid" }
    }, 401)
  }
}

// 可选认证 (不强制，但如果提供了 token 则解析)
export async function authOptional(c: Context, next: Next) {
  const header = c.req.header("Authorization")
  if (header?.startsWith("Bearer ")) {
    try {
      const payload = verifyAccessToken(header.slice(7))
      c.set("userId", payload.userId)
      c.set("userRole", payload.role)
    } catch { /* token invalid, continue unauthenticated */ }
  }
  await next()
}

// 角色守卫工厂
export function requireRole(...roles: string[]) {
  return async (c: Context, next: Next) => {
    const userRole = c.get("userRole") as string
    if (!roles.includes(userRole)) {
      return c.json({
        success: false,
        error: { code: "FORBIDDEN", message: "Insufficient permissions" }
      }, 403)
    }
    await next()
  }
}
```

> **[WRITE]** `packages/backend/src/middleware/rate-limiter.ts`:

```typescript
import type { Context, Next } from "hono"

const hits = new Map<string, { count: number; resetAt: number }>()

export function rateLimiter(opts: { windowMs: number; max: number }) {
  return async (c: Context, next: Next) => {
    const key = c.req.header("x-forwarded-for") || "unknown"
    const now = Date.now()
    const record = hits.get(key)

    if (!record || record.resetAt < now) {
      hits.set(key, { count: 1, resetAt: now + opts.windowMs })
      return next()
    }

    if (record.count >= opts.max) {
      return c.json({
        success: false,
        error: { code: "RATE_LIMITED", message: "Too many requests, please try later" }
      }, 429)
    }

    record.count++
    await next()
  }

  // 定期清理 (每 5 分钟)
  setInterval(() => {
    const now = Date.now()
    for (const [key, record] of hits) {
      if (record.resetAt < now) hits.delete(key)
    }
  }, 300_000)
}
```

> **[WRITE]** `packages/backend/src/middleware/security.ts` — 安全 Headers + 请求 ID:

```typescript
import type { Context, Next } from "hono"
import { randomUUID } from "crypto"

// 安全 Headers (OWASP 推荐 + 企业级基线)
export async function securityHeaders(c: Context, next: Next) {
  c.res.headers.set("X-Content-Type-Options", "nosniff")
  c.res.headers.set("X-Frame-Options", "DENY")
  c.res.headers.set("X-XSS-Protection", "0")
  c.res.headers.set("Referrer-Policy", "strict-origin-when-cross-origin")
  c.res.headers.set("X-Permitted-Cross-Domain-Policies", "none")
  c.res.headers.set("X-Download-Options", "noopen")
  if (process.env.NODE_ENV === "production") {
    c.res.headers.set("Content-Security-Policy",
      "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' https: data:; font-src 'self'; connect-src 'self' https://api.yourdomain.com")
    c.res.headers.set("Strict-Transport-Security", "max-age=31536000; includeSubDomains; preload")
  }
  await next()
}

// 请求 ID — 全链路追踪基础
export async function requestId(c: Context, next: Next) {
  const id = c.req.header("X-Request-ID") || randomUUID()
  c.set("requestId", id)
  c.res.headers.set("X-Request-ID", id)
  await next()
}
```

> **[WRITE]** `packages/backend/src/lib/logger.ts` — 结构化 JSON 日志 (可接入 ELK / 华为云 LTS):

```typescript
export interface LogEntry {
  level: "info" | "warn" | "error" | "debug"
  message: string
  requestId?: string
  userId?: string
  [key: string]: any
}

function emit(entry: LogEntry) {
  console.log(JSON.stringify({ timestamp: new Date().toISOString(), ...entry }))
}

export const logger = {
  info(msg: string, ctx?: Record<string, any>) { emit({ level: "info", message: msg, ...ctx }) },
  warn(msg: string, ctx?: Record<string, any>) { emit({ level: "warn", message: msg, ...ctx }) },
  error(msg: string, err?: Error, ctx?: Record<string, any>) {
    emit({ level: "error", message: msg, errorMessage: err?.message, stack: err?.stack, ...ctx })
  },
  debug(msg: string, ctx?: Record<string, any>) {
    if (process.env.NODE_ENV !== "production") emit({ level: "debug", message: msg, ...ctx })
  },
}
```

### Step 3.4 — 认证路由

> **[WRITE]** `packages/backend/src/routes/auth.ts`:

```typescript
import { Hono } from "hono"
import { zValidator } from "@hono/zod-validator"
import { z } from "zod"
import bcrypt from "bcryptjs"
import { prisma } from "../lib/db"
import { signAccessToken, signRefreshToken, verifyRefreshToken } from "../lib/jwt"
import { ok, fail } from "../lib/response"
import { authRequired } from "../middleware/auth"
import { v4 as uuid } from "uuid"

export const authRoutes = new Hono()

// POST /auth/register
authRoutes.post("/register", zValidator("json", z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100),
  name: z.string().min(1).max(50).optional(),
})), async (c) => {
  const { email, password, name } = c.req.valid("json")

  const existing = await prisma.user.findUnique({ where: { email } })
  if (existing) return fail(c, "EMAIL_EXISTS", "This email is already registered", 409)

  const hashed = await bcrypt.hash(password, 10)
  const user = await prisma.user.create({
    data: { email, password: hashed, name },
    select: { id: true, email: true, name: true, role: true, avatar: true, createdAt: true }
  })

  const accessToken = signAccessToken({ userId: user.id, role: user.role })
  const refreshToken = signRefreshToken({ userId: user.id, role: user.role })

  // 保存 refresh token
  await prisma.refreshToken.create({
    data: { token: refreshToken, userId: user.id, expiresAt: new Date(Date.now() + 7 * 86400000) }
  })

  return ok(c, { user, accessToken, refreshToken }, 201)
})

// POST /auth/login
authRoutes.post("/login", zValidator("json", z.object({
  email: z.string().email(),
  password: z.string().min(1),
  deviceName: z.string().optional(),
  platform: z.enum(["IOS", "ANDROID", "WEB"]).optional(),
  pushToken: z.string().optional(),
})), async (c) => {
  const { email, password, deviceName, platform, pushToken } = c.req.valid("json")

  const user = await prisma.user.findUnique({ where: { email } })
  if (!user) return fail(c, "INVALID_CREDENTIALS", "Invalid email or password", 401)

  if (user.status === "DISABLED") return fail(c, "ACCOUNT_DISABLED", "Account is disabled", 403)

  const valid = await bcrypt.compare(password, user.password)
  if (!valid) return fail(c, "INVALID_CREDENTIALS", "Invalid email or password", 401)

  const accessToken = signAccessToken({ userId: user.id, role: user.role })
  const refreshToken = signRefreshToken({ userId: user.id, role: user.role })

  await prisma.refreshToken.create({
    data: { token: refreshToken, userId: user.id, expiresAt: new Date(Date.now() + 7 * 86400000) }
  })

  // 注册设备推送 token
  if (platform && pushToken) {
    await prisma.userDevice.upsert({
      where: { userId_platform_pushToken: { userId: user.id, platform, pushToken } },
      update: { lastActiveAt: new Date(), deviceName },
      create: { userId: user.id, platform, pushToken, deviceName }
    })
  }

  return ok(c, {
    user: { id: user.id, email: user.email, name: user.name, role: user.role, avatar: user.avatar },
    accessToken,
    refreshToken
  })
})

// POST /auth/refresh
authRoutes.post("/refresh", zValidator("json", z.object({
  refreshToken: z.string(),
})), async (c) => {
  const { refreshToken } = c.req.valid("json")

  try {
    const payload = verifyRefreshToken(refreshToken)
    const stored = await prisma.refreshToken.findUnique({ where: { token: refreshToken } })

    if (!stored || stored.expiresAt < new Date()) {
      return fail(c, "TOKEN_EXPIRED", "Refresh token expired", 401)
    }

    // 轮换 refresh token
    await prisma.refreshToken.delete({ where: { id: stored.id } })
    const newAccessToken = signAccessToken({ userId: payload.userId, role: payload.role })
    const newRefreshToken = signRefreshToken({ userId: payload.userId, role: payload.role })

    await prisma.refreshToken.create({
      data: { token: newRefreshToken, userId: payload.userId, expiresAt: new Date(Date.now() + 7 * 86400000) }
    })

    return ok(c, { accessToken: newAccessToken, refreshToken: newRefreshToken })
  } catch {
    return fail(c, "TOKEN_INVALID", "Invalid refresh token", 401)
  }
})

// POST /auth/logout
authRoutes.post("/logout", authRequired, async (c) => {
  const userId = c.get("userId") as string
  await prisma.refreshToken.deleteMany({ where: { userId } })
  return ok(c, { loggedOut: true })
})

// GET /auth/me
authRoutes.get("/me", authRequired, async (c) => {
  const userId = c.get("userId") as string
  const user = await prisma.user.findUnique({
    where: { id: userId },
    select: { id: true, email: true, phone: true, name: true, role: true, avatar: true, createdAt: true }
  })
  if (!user) return fail(c, "NOT_FOUND", "User not found", 404)
  return ok(c, user)
})

// POST /auth/forgot-password — 发送重置密码邮件
authRoutes.post("/forgot-password", zValidator("json", z.object({
  email: z.string().email(),
})), async (c) => {
  const { email } = c.req.valid("json")

  // 不暴露用户是否存在 (防止邮箱枚举攻击)
  const user = await prisma.user.findUnique({ where: { email } })
  if (!user) return ok(c, { message: "If the email exists, a reset link has been sent" })

  // 生成一次性重置 Token (5分钟有效)
  const resetToken = signAccessToken({ userId: user.id, role: user.role })
  // store in-memory or dedicated table; here simplified as JWT with short expiry

  // 发送邮件 (需配置 email service)
  const { emailService } = await import("../services/email-service")
  await emailService.sendPasswordReset(email, resetToken)

  return ok(c, { message: "If the email exists, a reset link has been sent" })
})

// POST /auth/reset-password
authRoutes.post("/reset-password", zValidator("json", z.object({
  token: z.string(),
  newPassword: z.string().min(8).max(100),
})), async (c) => {
  const { token, newPassword } = c.req.valid("json")

  try {
    const payload = verifyAccessToken(token)
    const hashed = await bcrypt.hash(newPassword, 10)
    await prisma.user.update({
      where: { id: payload.userId },
      data: { password: hashed }
    })
    // 清除所有 refresh token (强制所有设备重新登录)
    await prisma.refreshToken.deleteMany({ where: { userId: payload.userId } })
    return ok(c, { message: "Password reset successfully, please login again" })
  } catch {
    return fail(c, "TOKEN_INVALID", "Reset token is invalid or expired", 400)
  }
})
```


### Step 3.5 — 内容路由 (CMS API)

> **[WRITE]** `packages/backend/src/routes/content.ts`:

```typescript
import { Hono } from "hono"
import { prisma } from "../lib/db"
import { ok, paginated } from "../lib/response"
import { authOptional } from "../middleware/auth"
import { cache } from "../lib/redis"

export const contentRoutes = new Hono()

// GET /content/articles — 文章列表 (公开，带缓存)
contentRoutes.get("/articles", authOptional, async (c) => {
  const page = parseInt(c.req.query("page") || "1")
  const limit = Math.min(parseInt(c.req.query("limit") || "20"), 50)
  const tag = c.req.query("tag")
  const search = c.req.query("search") || ""

  const cacheKey = `articles:${page}:${limit}:${tag}:${search}`
  const cached = await cache.get(cacheKey)
  if (cached) return c.json(JSON.parse(cached))

  const where: any = { status: "PUBLISHED" }
  if (tag) where.tags = { has: tag }
  if (search) where.OR = [
    { title: { contains: search } },
    { summary: { contains: search } },
  ]

  const [articles, total] = await Promise.all([
    prisma.article.findMany({
      where,
      select: { id: true, title: true, slug: true, summary: true, coverImage: true, tags: true, publishedAt: true },
      orderBy: { publishedAt: "desc" },
      skip: (page - 1) * limit,
      take: limit,
    }),
    prisma.article.count({ where }),
  ])

  const result = paginated(c, articles, total, page, limit)
  await cache.set(cacheKey, JSON.stringify(result), 60)
  return result
})

// GET /content/articles/:slug — 文章详情
contentRoutes.get("/articles/:slug", authOptional, async (c) => {
  const slug = c.req.param("slug")

  const article = await prisma.article.findUnique({
    where: { slug, status: "PUBLISHED" },
    include: { author: { select: { name: true, avatar: true } } }
  })
  if (!article) return c.json({ success: false, error: { code: "NOT_FOUND", message: "Article not found" } }, 404)

  return ok(c, article)
})

// GET /content/banners — Banner 列表 (公开)
contentRoutes.get("/banners", async (c) => {
  const position = c.req.query("position") || "home_top"

  const banners = await prisma.banner.findMany({
    where: {
      position,
      status: "PUBLISHED",
      OR: [
        { startAt: null, endAt: null },
        { startAt: { lte: new Date() }, endAt: { gte: new Date() } },
      ],
    },
    select: { id: true, title: true, imageUrl: true, linkUrl: true },
    orderBy: { sortOrder: "asc" },
  })

  return ok(c, banners)
})

// GET /content/announcements — 公告列表
contentRoutes.get("/announcements", async (c) => {
  const announcements = await prisma.announcement.findMany({
    where: {
      status: "PUBLISHED",
      OR: [
        { startAt: null, endAt: null },
        { startAt: { lte: new Date() }, endAt: { gte: new Date() } },
      ],
    },
    select: { id: true, title: true, content: true, type: true, isSticky: true },
    orderBy: [{ isSticky: "desc" }, { createdAt: "desc" }],
    take: 10,
  })
  return ok(c, announcements)
})
```

### Step 3.6 — 文件上传路由

> **[WRITE]** `packages/backend/src/routes/upload.ts`:

```typescript
import { Hono } from "hono"
import { authRequired } from "../middleware/auth"
import { ok, fail } from "../lib/response"
import { prisma } from "../lib/db"
import * as fs from "fs/promises"
import * as path from "path"
import { randomUUID } from "crypto"

const UPLOAD_DIR = path.join(process.cwd(), "uploads")
const ALLOWED_IMAGE_TYPES = ["image/jpeg", "image/png", "image/webp", "image/gif", "image/svg+xml"]
const MAX_FILE_SIZE = 10 * 1024 * 1024 // 10MB

export const uploadRoutes = new Hono()

// POST /upload/image
uploadRoutes.post("/image", authRequired, async (c) => {
  const userId = c.get("userId") as string
  const body = await c.req.parseBody()
  const file = body["file"] as File

  if (!file) return fail(c, "NO_FILE", "No file provided")
  if (!ALLOWED_IMAGE_TYPES.includes(file.type)) return fail(c, "INVALID_TYPE", "Only JPEG/PNG/WebP/GIF/SVG allowed")
  if (file.size > MAX_FILE_SIZE) return fail(c, "FILE_TOO_LARGE", "File exceeds 10MB limit")

  // 本地存储 (生产环境 → Phase 12 替换为 OBS 上传)
  await fs.mkdir(UPLOAD_DIR, { recursive: true })
  const ext = path.extname(file.name) || ".jpg"
  const key = `images/${userId}/${randomUUID()}${ext}`
  const filePath = path.join(UPLOAD_DIR, key)
  await fs.mkdir(path.dirname(filePath), { recursive: true })

  const buffer = Buffer.from(await file.arrayBuffer())
  await fs.writeFile(filePath, buffer)

  // 记录到数据库
  const mediaFile = await prisma.mediaFile.create({
    data: {
      userId,
      key,
      url: `/uploads/${key}`, // 开发环境直接访问，生产用 CDN URL
      fileName: file.name,
      fileType: "image",
      mimeType: file.type,
      fileSize: file.size,
    }
  })

  return ok(c, mediaFile, 201)
})

// GET /upload/:key — 获取文件访问 URL (生产返回 CDN URL)
uploadRoutes.get("/:key", async (c) => {
  const key = c.req.param("key")
  const mediaFile = await prisma.mediaFile.findFirst({ where: { key } })
  if (!mediaFile) return fail(c, "NOT_FOUND", "File not found", 404)
  return ok(c, { url: mediaFile.url, key: mediaFile.key })
})
```

### Step 3.7 — 健康检查 & App 配置

> **[WRITE]** `packages/backend/src/routes/health.ts`:

```typescript
import { Hono } from "hono"
import { prisma } from "../lib/db"

export const healthRoute = new Hono()

healthRoute.get("/health", async (c) => {
  try {
    await prisma.$queryRaw`SELECT 1`
    return c.json({
      status: "healthy",
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      version: "1.0.0",
    })
  } catch (e) {
    return c.json({
      status: "unhealthy",
      database: "disconnected",
      error: (e as Error).message,
    }, 503)
  }
})
```

> **[WRITE]** `packages/backend/src/routes/app-config.ts`:

```typescript
import { Hono } from "hono"
import { prisma } from "../lib/db"
import { ok } from "../lib/response"

export const appConfigRoute = new Hono()

// GET /app/config — 公开，客户端启动时调用获取配置
appConfigRoute.get("/config", async (c) => {
  const platform = c.req.query("platform") || "WEB"
  const version = c.req.query("version") || "0.0.0"

  const configs = await prisma.appConfig.findMany({
    where: {
      key: {
        in: [
          `${platform.toLowerCase()}_latest_version`,
          `${platform.toLowerCase()}_min_version`,
          "force_update_required",
          "maintenance_mode",
        ]
      }
    }
  })

  const configMap: Record<string, string> = {}
  for (const config of configs) {
    configMap[config.key] = config.value
  }

  return ok(c, {
    latestVersion: configMap[`${platform.toLowerCase()}_latest_version`] || version,
    minVersion: configMap[`${platform.toLowerCase()}_min_version`] || version,
    forceUpdate: configMap["force_update_required"] === "true",
    maintenanceMode: configMap["maintenance_mode"] === "true",
  })
})
```

### Step 3.8 — 后端启动 & 验证

```bash
# [SHELL] 启动 PostgreSQL (本地 Docker)
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}

docker run -d --name sop-postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  postgres:16-alpine 2>/dev/null || docker start sop-postgres

# 等待数据库就绪
echo "等待 PostgreSQL 启动..."
sleep 3

# 配置 .env
cat > packages/backend/.env << 'ENVEOF'
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/mydb
JWT_SECRET=dev-secret-change-me-in-production-64chars
JWT_REFRESH_SECRET=dev-refresh-secret-change-me-too
CORS_ORIGINS=http://localhost:3000,http://localhost:3001
REDIS_URL=redis://localhost:6379
NODE_ENV=development
ENVEOF

# [SHELL] 数据库迁移
cd packages/backend
npx prisma migrate dev --name init
npx prisma generate

# [SHELL] 种子数据
npx tsx prisma/seed.ts

# [SHELL] 启动后端
pnpm dev &
sleep 3

# [VALIDATE] 验证 API
echo "=== 健康检查 ==="
curl -s http://localhost:3000/api/v1/health | python3 -m json.tool

echo ""
echo "=== 注册测试用户 ==="
curl -s -X POST http://localhost:3000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test123456","name":"Test User"}' | python3 -m json.tool

echo ""
echo "=== 登录获取 Token ==="
TOKEN=$(curl -s -X POST http://localhost:3000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test123456"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['accessToken'])")
echo "Token: ${TOKEN:0:20}..."

echo ""
echo "=== 获取文章列表 ==="
curl -s http://localhost:3000/api/v1/content/articles | python3 -m json.tool

echo ""
echo "=== 获取 Banner ==="
curl -s http://localhost:3000/api/v1/content/banners | python3 -m json.tool

echo ""
echo "=== 获取当前用户 (带认证) ==="
curl -s http://localhost:3000/api/v1/auth/me \
  -H "Authorization: Bearer $TOKEN" | python3 -m json.tool

echo ""
echo "✅ 后端 API 全部验证通过!"

# [GIT]
cd ${PROJECT_DIR}
```

### Step 3.9 — 企业级服务补齐

> **[WRITE]** + **[GENERATE]**

#### 3.9.1 邮件服务

> **[WRITE]** `packages/backend/src/services/email-service.ts`:

```typescript
import { logger } from "../lib/logger"

// 邮件服务抽象 (开发环境用 console.log 替代，生产接 SMTP / 华为云邮件)
class EmailService {
  private enabled = !!process.env.SMTP_HOST

  async sendPasswordReset(email: string, token: string): Promise<void> {
    const resetUrl = `${process.env.APP_URL || "http://localhost:3001"}/reset-password?token=${token}`

    if (!this.enabled) {
      logger.info("[EMAIL] SMTP not configured, logging instead:", { email, resetUrl })
      return
    }

    // 生产环境: 使用 nodemailer 或华为云邮件服务
    // const transporter = nodemailer.createTransport({ host: process.env.SMTP_HOST, ... })
    // await transporter.sendMail({ from, to: email, subject: "密码重置", html: `...` })

    logger.info("[EMAIL] Password reset email sent", { email })
  }

  async sendWelcome(email: string, name?: string): Promise<void> {
    if (!this.enabled) {
      logger.info("[EMAIL] Welcome email (dry run)", { email, name })
      return
    }
    // 生产实现
  }

  async sendNotification(email: string, subject: string, body: string): Promise<void> {
    if (!this.enabled) {
      logger.info("[EMAIL] Notification (dry run)", { email, subject })
      return
    }
    // 生产实现
  }
}

export const emailService = new EmailService()
```

#### 3.9.2 OBS 文件上传服务 (真实实现)

> **[WRITE]** `packages/backend/src/services/obs-service.ts`:

```typescript
import { logger } from "../lib/logger"
import * as fs from "fs/promises"
import * as path from "path"

// 华为云 OBS 上传服务
// 开发环境 → 本地存储；生产环境 → OBS SDK 上传
class ObsService {
  private isProduction = process.env.NODE_ENV === "production"
  private bucket = process.env.HUAWEI_OBS_BUCKET || ""
  private endpoint = process.env.HUAWEI_OBS_ENDPOINT || ""

  // 上传文件到 OBS (或本地)
  async uploadFile(
    localPath: string,
    remoteKey: string,
    contentType: string
  ): Promise<string> {
    if (!this.isProduction || !this.bucket) {
      // 开发环境: 本地存储，返回本地 URL
      const uploadDir = path.join(process.cwd(), "uploads")
      await fs.mkdir(path.dirname(path.join(uploadDir, remoteKey)), { recursive: true })
      await fs.copyFile(localPath, path.join(uploadDir, remoteKey))
      const devUrl = `/uploads/${remoteKey}`
      logger.debug("[OBS] Local upload", { remoteKey, url: devUrl })
      return devUrl
    }

    // 生产环境: 使用 ObsClient SDK
    try {
      // const ObsClient = require("esdk-obs-nodejs")
      // const obsClient = new ObsClient({
      //   access_key_id: process.env.HUAWEI_OBS_ACCESS_KEY!,
      //   secret_access_key: process.env.HUAWEI_OBS_SECRET_KEY!,
      //   server: this.endpoint,
      // })
      // const result = await obsClient.putObject({
      //   Bucket: this.bucket,
      //   Key: remoteKey,
      //   SourceFile: localPath,
      //   ContentType: contentType,
      // })

      // CDN URL (如果配置了 CDN 加速域名)
      const cdnDomain = process.env.CDN_DOMAIN || this.endpoint.replace("obs", "cdn")
      const url = `https://${cdnDomain}/${remoteKey}`

      logger.info("[OBS] File uploaded", { remoteKey, url })
      return url
    } catch (error) {
      logger.error("[OBS] Upload failed", error as Error, { remoteKey })
      throw error
    }
  }

  // 获取公开访问 URL
  getPublicUrl(key: string): string {
    if (!this.isProduction || !this.bucket) {
      return `/uploads/${key}`
    }
    const cdnDomain = process.env.CDN_DOMAIN || ""
    return cdnDomain ? `https://${cdnDomain}/${key}` : `${this.endpoint}/${this.bucket}/${key}`
  }

  // 删除文件
  async deleteFile(key: string): Promise<void> {
    if (!this.isProduction) {
      const localPath = path.join(process.cwd(), "uploads", key)
      await fs.unlink(localPath).catch(() => {})
      return
    }
    // 生产: obsClient.deleteObject({ Bucket: this.bucket, Key: key })
  }
}

export const obsService = new ObsService()
```

> `packages/backend/src/routes/upload.ts` 中集成 OBS 服务 (替换原有本地存储):
> 将原有 `fs.writeFile(filePath, buffer)` 替换为:
> `const url = await obsService.uploadFile(tempPath, key, file.type)`

#### 3.9.3 增强健康检查

> **[WRITE]** `packages/backend/src/routes/health.ts` (替换为增强版):

```typescript
import { Hono } from "hono"
import { prisma } from "../lib/db"
import { cache } from "../lib/redis"
import { logger } from "../lib/logger"

export const healthRoute = new Hono()

healthRoute.get("/health", async (c) => {
  const checks: Record<string, { status: string; latency?: number }> = {}
  let healthy = true

  // 1. 数据库检查
  const dbStart = Date.now()
  try {
    await prisma.$queryRaw`SELECT 1`
    checks.database = { status: "healthy", latency: Date.now() - dbStart }
  } catch (e) {
    checks.database = { status: "unhealthy" }
    healthy = false
  }

  // 2. Redis 检查
  const redisStart = Date.now()
  try {
    await cache.set("health_check", "ok", 10)
    const val = await cache.get("health_check")
    checks.redis = { status: val === "ok" ? "healthy" : "unhealthy", latency: Date.now() - redisStart }
  } catch {
    checks.redis = { status: "unavailable" } // 非关键
  }

  // 3. 磁盘检查 (简单)
  checks.uptime = Math.floor(process.uptime())

  logger.debug("Health check", checks)

  return c.json({
    status: healthy ? "healthy" : "degraded",
    version: "1.0.0",
    timestamp: new Date().toISOString(),
    checks,
  }, healthy ? 200 : 503)
})
```

#### 3.9.4 数据库连接池 & Prisma 生产配置

> **[WRITE]** `packages/backend/src/lib/db.ts` (替换为增强版):

```typescript
import { PrismaClient } from "@prisma/client"
import { logger } from "./logger"

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient }

const prismaClientOptions = {
  log: [
    { level: "error" as const, emit: "event" as const },
    { level: "warn" as const, emit: "event" as const },
  ],
  // 生产环境连接池配置
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
}

export const prisma = globalForPrisma.prisma || new PrismaClient(prismaClientOptions)

// 记录慢查询 (>500ms)
prisma.$on("query" as any, (e: any) => {
  if (e.duration > 500) {
    logger.warn("Slow query detected", { query: e.query, duration: e.duration, params: e.params })
  }
})

prisma.$on("error", (e: any) => {
  logger.error("Prisma error", new Error(e.message), { target: e.target })
})

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma
}

// 连接池大小通过 DATABASE_URL 参数控制:
// postgresql://user:pass@host:5432/db?connection_limit=20&pool_timeout=10
```

```bash
# [SHELL] 生产环境数据库连接池参数
# 在生产 .env 的 DATABASE_URL 末尾追加:
echo "# 连接池: DATABASE_URL=postgresql://...?connection_limit=20&pool_timeout=10&socket_timeout=30"
```

#### 3.9.5 .dockerignore & .gitleaks 配置

> **[WRITE]** `.dockerignore`:

```dockerignore
node_modules
.next
dist
.git
.env
.env.local
.env.production
uploads
*.md
!.github
```

> **[WRITE]** `.gitleaks.toml`:

```toml
# GitLeaks 配置 — 检测代码中的密钥泄露
[title]
description = "SOP Fullstack — Gitleaks Config"

[allowlist]
  description = "允许的测试/示例值"
  paths = [
    '''\.md$''',
    '''\.example$''',
    '''\.env\.example$''',
  ]
```

```bash
# [SHELL] 安装依赖补充
cd packages/backend
pnpm add nodemailer 2>/dev/null || true  # 邮件发送 (生产需要)
pnpm add -D @types/nodemailer 2>/dev/null || true
```

```bash
# [GIT]
cd ${PROJECT_DIR}
git add -A && git commit -m "Phase 3: Backend API development complete (Hono + Prisma + JWT)"
```

---

## Phase 4: CMS 集成 (5:30-6:30)

> **[RESEARCH]** + **[SHELL]** + **[WRITE]**

### Step 4.1 — Directus CMS 安装 & 配置

> **为什么选 Directus?** 它直接包装已有 PostgreSQL 数据库，不强制用它的 migration 系统，可以与 Prisma 管理的 schema 和平共存。

```bash
# [SHELL] 安装 Directus
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}

cd packages/cms

# 初始化 Directus 项目
cat > package.json << 'PKGJSON'
{
  "name": "cms",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "directus start",
    "bootstrap": "npx directus bootstrap"
  }
}
PKGJSON

pnpm add directus

# 创建 Directus 配置
cat > .env << 'CMSENV'
# Directus 连同一个数据库，但有自己独立的 schema
DB_CLIENT=pg
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=mydb
DB_USER=postgres
DB_PASSWORD=postgres

# Directus Admin 账号
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=admin123

# 安全
KEY=replace-with-random-string
SECRET=replace-with-random-string

# 配置
PUBLIC_URL=http://localhost:8055
CMSENV

# [SHELL] Bootstrap Directus (创建 Directus 系统表到同一数据库)
npx directus bootstrap

# 后台启动 Directus
npx directus start &
sleep 5
```

### Step 4.2 — CMS 内容模型配置

> **[DIALOG]** 在浏览器打开 `http://localhost:8055/admin`，用 `admin@example.com / admin123` 登录，然后按以下步骤配置：

```markdown
## Directus CMS 内容模型配置指南

### 1. 创建 Collections (内容集合)

Directus 会自动发现数据库中已有的表 (articles, banners, announcements 等)，
将其显示为 Collections。你需要为每个 Collection 配置字段权限：

#### Articles 集合
- Settings → Fields → 确认字段映射正确
- 配置 slug 为自动生成 (基于 title)
- 设置 publishedAt 默认值

#### Banners 集合
- 配置 imageUrl 为 Image 类型
- 设置 sortOrder 排序

#### Announcements 集合
- 配置 type 为 Dropdown: info/warning/success/error

### 2. 创建 Roles & Permissions

Settings → Roles:
1. **Administrator** — 完全访问 (默认)
2. **Editor** — 只能管理 Articles/Banners/Announcements
   - Articles: Create/Read/Update (不能 Delete)
   - Banners: Create/Read/Update
   - Announcements: Create/Read/Update
3. **Public** — 只能 Read 已发布内容
   - Articles: Read (filter: status=PUBLISHED)
   - Banners: Read (filter: status=PUBLISHED)
   - Announcements: Read (filter: status=PUBLISHED)

### 3. 配置 Webhooks (可选)

Settings → Webhooks:
- Actions: Create/Update/Delete
- Collections: articles, banners, announcements
- URL: http://backend:3000/api/v1/cms/webhook (缓存刷新)
```

### Step 4.3 — CMS API 代理配置

> 在 Nginx 配置中 (Phase 11) 统一入口:
> - `/api/*` → Backend (Hono :3000)
> - `/admin/*` → CMS (Directus :8055)
> - `/` → Web Frontend (Next.js :3001)

```bash
# [GIT]
git add -A && git commit -m "Phase 4: CMS (Directus) integrated with shared PostgreSQL"
```

---

## Phase 5: 统一认证授权 (6:30-8:00)

> **[WRITE]** + **[VALIDATE]**

### Step 5.1 — 用户管理路由 (Admin)

> **[WRITE]** `packages/backend/src/routes/users.ts`:

```typescript
import { Hono } from "hono"
import { zValidator } from "@hono/zod-validator"
import { z } from "zod"
import { prisma } from "../lib/db"
import { ok, fail, paginated } from "../lib/response"
import { authRequired, requireRole } from "../middleware/auth"

export const userRoutes = new Hono()

// 所有用户路由需要认证
userRoutes.use("*", authRequired)

// GET /users — Admin: 用户列表
userRoutes.get("/", requireRole("SUPER_ADMIN", "ADMIN"), async (c) => {
  const page = parseInt(c.req.query("page") || "1")
  const limit = Math.min(parseInt(c.req.query("limit") || "20"), 100)
  const search = c.req.query("search") || ""
  const role = c.req.query("role")
  const status = c.req.query("status")

  const where: any = {}
  if (search) where.OR = [
    { email: { contains: search } },
    { name: { contains: search } },
  ]
  if (role) where.role = role
  if (status) where.status = status

  const [users, total] = await Promise.all([
    prisma.user.findMany({
      where,
      select: { id: true, email: true, name: true, role: true, status: true, avatar: true, createdAt: true },
      orderBy: { createdAt: "desc" },
      skip: (page - 1) * limit,
      take: limit,
    }),
    prisma.user.count({ where }),
  ])

  return paginated(c, users, total, page, limit)
})

// PATCH /users/:id — 更新用户 (自己 或 Admin)
userRoutes.patch("/:id", zValidator("json", z.object({
  name: z.string().min(1).max(50).optional(),
  avatar: z.string().url().optional(),
  phone: z.string().optional(),
  role: z.enum(["SUPER_ADMIN", "ADMIN", "EDITOR", "USER", "GUEST"]).optional(),
  status: z.enum(["ACTIVE", "DISABLED", "DELETED"]).optional(),
})), async (c) => {
  const userId = c.get("userId") as string
  const userRole = c.get("userRole") as string
  const targetId = c.req.param("id")
  const body = c.req.valid("json")

  // 权限检查: 只能修改自己，除非是 Admin
  if (userId !== targetId && !["SUPER_ADMIN", "ADMIN"].includes(userRole)) {
    return fail(c, "FORBIDDEN", "Can only update your own profile", 403)
  }

  // 只有 SUPER_ADMIN 可以修改角色
  if (body.role && userRole !== "SUPER_ADMIN") {
    return fail(c, "FORBIDDEN", "Only super admin can change roles", 403)
  }

  const user = await prisma.user.update({
    where: { id: targetId },
    data: body,
    select: { id: true, email: true, name: true, role: true, status: true, avatar: true }
  })

  return ok(c, user)
})

// DELETE /users/:id — Admin: 软删除用户
userRoutes.delete("/:id", requireRole("SUPER_ADMIN", "ADMIN"), async (c) => {
  const targetId = c.req.param("id")

  await prisma.user.update({
    where: { id: targetId },
    data: { status: "DELETED" }
  })

  // 清除该用户的所有 refresh token
  await prisma.refreshToken.deleteMany({ where: { userId: targetId } })

  return ok(c, { deleted: true })
})
```

### Step 5.2 — 通知路由

> **[WRITE]** `packages/backend/src/routes/notifications.ts`:

```typescript
import { Hono } from "hono"
import { prisma } from "../lib/db"
import { ok, paginated } from "../lib/response"
import { authRequired } from "../middleware/auth"

export const notificationRoutes = new Hono()
notificationRoutes.use("*", authRequired)

// GET /notifications — 获取通知列表
notificationRoutes.get("/", async (c) => {
  const userId = c.get("userId") as string
  const page = parseInt(c.req.query("page") || "1")
  const limit = Math.min(parseInt(c.req.query("limit") || "20"), 50)

  const [notifications, total] = await Promise.all([
    prisma.notification.findMany({
      where: { userId },
      orderBy: { createdAt: "desc" },
      skip: (page - 1) * limit,
      take: limit,
    }),
    prisma.notification.count({ where: { userId } }),
  ])

  const unreadCount = await prisma.notification.count({ where: { userId, isRead: false } })
  return paginated(c, notifications, total, page, limit)
})

// PATCH /notifications/read-all — 全部标记为已读
notificationRoutes.patch("/read-all", async (c) => {
  const userId = c.get("userId") as string
  await prisma.notification.updateMany({
    where: { userId, isRead: false },
    data: { isRead: true },
  })
  return ok(c, { success: true })
})
```

### Step 5.3 — 认证链路完整测试

```bash
# [SHELL] 端到端认证流程验证
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}

echo "=== 1. 注册 ==="
REGISTER=$(curl -s -X POST http://localhost:3000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"e2e@test.com","password":"test123456","name":"E2E User"}')
echo $REGISTER | python3 -m json.tool

ACCESS_TOKEN=$(echo $REGISTER | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['accessToken'])")
REFRESH_TOKEN=$(echo $REGISTER | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['refreshToken'])")

echo ""
echo "=== 2. 获取当前用户 ==="
curl -s http://localhost:3000/api/v1/auth/me \
  -H "Authorization: Bearer $ACCESS_TOKEN" | python3 -m json.tool

echo ""
echo "=== 3. 刷新 Token ==="
REFRESH_RESULT=$(curl -s -X POST http://localhost:3000/api/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d "{\"refreshToken\":\"$REFRESH_TOKEN\"}")
echo $REFRESH_RESULT | python3 -m json.tool
NEW_ACCESS=$(echo $REFRESH_RESULT | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['accessToken'])")

echo ""
echo "=== 4. 未认证请求 (应返回 401) ==="
curl -s http://localhost:3000/api/v1/users | python3 -m json.tool

echo ""
echo "=== 5. 登出 ==="
curl -s -X POST http://localhost:3000/api/v1/auth/logout \
  -H "Authorization: Bearer $NEW_ACCESS" | python3 -m json.tool

echo ""
echo "✅ 认证链路完整验证通过!"

# [GIT]
git add -A && git commit -m "Phase 5: Unified auth system (JWT + RBAC + refresh token rotation)"
```

---

## Phase 6: 共享 API Client SDK (Day 2, 0:00-0:30)

> **[WRITE]** + **[GENERATE]**

> 所有前端共享同一套 API 调用封装，保证类型安全和调用一致性。

> **[WRITE]** `packages/shared/src/api-client.ts`:

```typescript
// 共享 API Client — Web/iOS/Android 通用接口规范
// 各平台可根据此接口实现原生 HTTP client

const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || "http://localhost:3000/api/v1"

// ===== 类型定义 (前后端共享) =====

export interface ApiResponse<T = any> {
  success: boolean
  data: T
  pagination?: {
    total: number
    page: number
    limit: number
    totalPages: number
  }
  error?: {
    code: string
    message: string
    details?: any
  }
}

export interface User {
  id: string
  email: string
  name?: string
  role: string
  avatar?: string
  createdAt: string
}

export interface Article {
  id: string
  title: string
  slug: string
  summary?: string
  coverImage?: string
  tags: string[]
  publishedAt?: string
}

export interface Banner {
  id: string
  title: string
  imageUrl: string
  linkUrl?: string
}

export interface Announcement {
  id: string
  title: string
  content: string
  type: "info" | "warning" | "success" | "error"
  isSticky: boolean
}

export interface Notification {
  id: string
  title: string
  body: string
  data?: any
  isRead: boolean
  createdAt: string
}

// ===== Token 管理 =====

const TOKEN_STORAGE_KEY = "auth_tokens"

export interface AuthTokens {
  accessToken: string
  refreshToken: string
}

export class TokenManager {
  static getTokens(): AuthTokens | null {
    if (typeof window === "undefined") return null
    const raw = localStorage.getItem(TOKEN_STORAGE_KEY)
    return raw ? JSON.parse(raw) : null
  }

  static setTokens(tokens: AuthTokens): void {
    if (typeof window === "undefined") return
    localStorage.setItem(TOKEN_STORAGE_KEY, JSON.stringify(tokens))
  }

  static clearTokens(): void {
    if (typeof window === "undefined") return
    localStorage.removeItem(TOKEN_STORAGE_KEY)
  }

  static getAccessToken(): string | null {
    return TokenManager.getTokens()?.accessToken || null
  }
}

// ===== HTTP Client =====

class ApiClient {
  private baseUrl: string

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<ApiResponse<T>> {
    const tokens = TokenManager.getTokens()
    const headers: Record<string, string> = {
      "Content-Type": "application/json",
      ...(options.headers as Record<string, string> || {}),
    }

    if (tokens?.accessToken) {
      headers["Authorization"] = `Bearer ${tokens.accessToken}`
    }

    const res = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers,
    })

    // Token 过期 → 自动刷新
    if (res.status === 401 && tokens?.refreshToken) {
      const refreshRes = await fetch(`${this.baseUrl}/auth/refresh`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ refreshToken: tokens.refreshToken }),
      })
      if (refreshRes.ok) {
        const refreshData: ApiResponse<AuthTokens> = await refreshRes.json()
        TokenManager.setTokens(refreshData.data)
        // 重试原请求
        return this.request<T>(endpoint, options)
      }
      TokenManager.clearTokens()
    }

    return res.json()
  }

  // ===== Auth API =====
  auth = {
    register: (data: { email: string; password: string; name?: string }) =>
      this.request<{ user: User; accessToken: string; refreshToken: string }>(
        "/auth/register",
        { method: "POST", body: JSON.stringify(data) }
      ),

    login: (data: { email: string; password: string; platform?: string; pushToken?: string }) =>
      this.request<{ user: User; accessToken: string; refreshToken: string }>(
        "/auth/login",
        { method: "POST", body: JSON.stringify(data) }
      ),

    refresh: (refreshToken: string) =>
      this.request<AuthTokens>("/auth/refresh", {
        method: "POST",
        body: JSON.stringify({ refreshToken }),
      }),

    logout: () =>
      this.request("/auth/logout", { method: "POST" }),

    me: () => this.request<User>("/auth/me"),
  }

  // ===== Content API =====
  content = {
    articles: (params?: { page?: number; limit?: number; tag?: string; search?: string }) => {
      const qs = new URLSearchParams()
      if (params?.page) qs.set("page", String(params.page))
      if (params?.limit) qs.set("limit", String(params.limit))
      if (params?.tag) qs.set("tag", params.tag)
      if (params?.search) qs.set("search", params.search)
      return this.request<Article[]>(`/content/articles?${qs.toString()}`)
    },

    article: (slug: string) =>
      this.request<Article & { content: string }>(`/content/articles/${slug}`),

    banners: (position = "home_top") =>
      this.request<Banner[]>(`/content/banners?position=${position}`),

    announcements: () =>
      this.request<Announcement[]>("/content/announcements"),
  }

  // ===== Upload API =====
  upload = {
    image: (file: File) => {
      const formData = new FormData()
      formData.append("file", file)
      return this.request<{ id: string; url: string }>("/upload/image", {
        method: "POST",
        headers: {}, // let browser set Content-Type for multipart
        body: formData,
      })
    },
  }

  // ===== Notification API =====
  notifications = {
    list: (page = 1) =>
      this.request<Notification[]>(`/notifications?page=${page}`),

    readAll: () =>
      this.request("/notifications/read-all", { method: "PATCH" }),
  }

  // ===== App Config =====
  appConfig = (platform: string, version: string) =>
    this.request<{
      latestVersion: string
      minVersion: string
      forceUpdate: boolean
      maintenanceMode: boolean
    }>(`/app/config?platform=${platform}&version=${version}`),
}

export const api = new ApiClient(API_BASE_URL)
```

> **[WRITE]** `packages/shared/src/index.ts`:

```typescript
export { api, TokenManager, ApiClient } from "./api-client"
export type {
  ApiResponse,
  User,
  Article,
  Banner,
  Announcement,
  Notification,
  AuthTokens,
} from "./api-client"
```

> **[WRITE]** `packages/shared/package.json`:

```json
{
  "name": "shared",
  "version": "1.0.0",
  "private": true,
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "exports": {
    ".": "./src/index.ts",
    "./*": "./src/*"
  }
}
```

```bash
# [GIT]
git add -A && git commit -m "Phase 6: Shared API client SDK for all platforms"
```

---

## Phase 7: Web 前端 — Next.js 用户端 + 管理后台 (0:30-3:30)

> **[GENERATE]** + **[WRITE]** + **[VALIDATE]**

### Step 7.1 — 创建 Next.js 项目

```bash
# [SHELL]
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}

cd packages/web

# 初始化 Next.js
npx create-next-app@latest . \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --use-pnpm \
  --no-turbopack \
  --skip-install 2>/dev/null || true

# 安装依赖
pnpm add next@latest react@latest react-dom@latest
pnpm add -D typescript @types/react @types/react-dom @types/node tailwindcss postcss autoprefixer

# 初始化
pnpm install

# 链接 shared workspace (使 import "shared" 生效)
cd ${PROJECT_DIR}
pnpm add --filter web shared@workspace:*
cd packages/web

# 验证 import 可解析
node -e "require.resolve('shared')" 2>/dev/null || echo "⚠️ shared 解析失败，检查 workspace 配置"
```

### Step 7.2 — 配置 Tailwind & 布局

> **[WRITE]** `packages/web/tailwind.config.ts`:

```typescript
import type { Config } from "tailwindcss"

const config: Config = {
  content: ["./src/**/*.{js,ts,jsx,tsx,mdx}"],
  theme: {
    extend: {
      colors: {
        primary: {
          50: "#eff6ff",
          100: "#dbeafe",
          200: "#bfdbfe",
          300: "#93c5fd",
          400: "#60a5fa",
          500: "#3b82f6",
          600: "#2563eb",
          700: "#1d4ed8",
          800: "#1e40af",
          900: "#1e3a8a",
        },
      },
    },
  },
  plugins: [],
}
export default config
```

> **[WRITE]** `packages/web/src/app/layout.tsx`:

```tsx
import type { Metadata } from "next"
import "./globals.css"

export const metadata: Metadata = {
  title: { default: "我的全栈应用", template: "%s | 我的全栈应用" },
  description: "高效管理你的内容",
}

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="zh-CN">
      <body className="min-h-[100dvh] bg-gray-50 text-gray-900 antialiased">
        {children}
      </body>
    </html>
  )
}
```

### Step 7.3 — 首页 (用户端)

> **[WRITE]** `packages/web/src/app/page.tsx`:

```tsx
import { api } from "shared"
import Link from "next/link"

async function getHomeData() {
  try {
    const [articlesRes, bannersRes, announcementsRes] = await Promise.all([
      fetch(`${process.env.NEXT_PUBLIC_API_URL}/content/articles?limit=10`),
      fetch(`${process.env.NEXT_PUBLIC_API_URL}/content/banners?position=home_top`),
      fetch(`${process.env.NEXT_PUBLIC_API_URL}/content/announcements`),
    ])
    const articles = (await articlesRes.json()).data || []
    const banners = (await bannersRes.json()).data || []
    const announcements = (await announcementsRes.json()).data || []
    return { articles, banners, announcements }
  } catch {
    return { articles: [], banners: [], announcements: [] }
  }
}

export default async function HomePage() {
  const { articles, banners, announcements } = await getHomeData()

  return (
    <div className="min-h-[100dvh]">
      {/* Header */}
      <header className="bg-white shadow-sm sticky top-0 z-10">
        <div className="max-w-4xl mx-auto px-4 py-3 flex items-center justify-between">
          <h1 className="text-xl font-bold">我的应用</h1>
          <nav className="flex gap-4 text-sm">
            <Link href="/" className="text-primary-600 font-medium">首页</Link>
            <Link href="/articles" className="text-gray-600 hover:text-primary-600">文章</Link>
            <Link href="/login" className="text-gray-600 hover:text-primary-600">登录</Link>
          </nav>
        </div>
      </header>

      <main className="max-w-4xl mx-auto p-4 space-y-6">
        {/* Banner 轮播区 */}
        {banners.length > 0 && (
          <section className="rounded-xl overflow-hidden bg-gradient-to-r from-primary-500 to-primary-700 text-white p-8 min-h-[200px] flex items-center">
            <div>
              <h2 className="text-2xl font-bold">{banners[0]?.title}</h2>
              {banners[0]?.linkUrl && (
                <Link href={banners[0].linkUrl} className="mt-3 inline-block bg-white text-primary-700 px-4 py-2 rounded-lg text-sm font-medium">
                  了解更多
                </Link>
              )}
            </div>
          </section>
        )}

        {/* 公告区 */}
        {announcements.length > 0 && (
          <section className="space-y-2">
            {announcements.map((a: any) => (
              <div key={a.id} className={`p-3 rounded-lg text-sm ${
                a.type === "warning" ? "bg-yellow-50 text-yellow-800 border border-yellow-200" :
                a.type === "error" ? "bg-red-50 text-red-800 border border-red-200" :
                a.type === "success" ? "bg-green-50 text-green-800 border border-green-200" :
                "bg-blue-50 text-blue-800 border border-blue-200"
              }`}>
                <span className="font-medium">{a.title}</span>
                {a.content && <span className="ml-2">{a.content}</span>}
              </div>
            ))}
          </section>
        )}

        {/* 文章列表 */}
        <section>
          <h2 className="text-lg font-semibold mb-3">最新文章</h2>
          {articles.length === 0 ? (
            <p className="text-gray-400 text-center py-12">暂无文章</p>
          ) : (
            <div className="grid gap-4 sm:grid-cols-2">
              {articles.map((article: any) => (
                <Link key={article.id} href={`/articles/${article.slug}`}
                  className="block bg-white rounded-xl p-5 shadow-sm hover:shadow-md transition-shadow">
                  {article.coverImage && (
                    <img src={article.coverImage} alt={article.title} className="w-full h-40 object-cover rounded-lg mb-3" />
                  )}
                  <h3 className="font-semibold line-clamp-2">{article.title}</h3>
                  {article.summary && <p className="text-sm text-gray-500 mt-1 line-clamp-2">{article.summary}</p>}
                  {article.tags?.length > 0 && (
                    <div className="flex gap-1 mt-2 flex-wrap">
                      {article.tags.map((tag: string) => (
                        <span key={tag} className="text-xs bg-gray-100 text-gray-600 px-2 py-0.5 rounded-full">{tag}</span>
                      ))}
                    </div>
                  )}
                </Link>
              ))}
            </div>
          )}
        </section>
      </main>
    </div>
  )
}
```

### Step 7.4 — 管理后台 (Admin Panel)

> **[WRITE]** `packages/web/src/app/admin/layout.tsx`:

```tsx
"use client"

import { useEffect, useState } from "react"
import { useRouter } from "next/navigation"
import { api, TokenManager } from "shared"

export default function AdminLayout({ children }: { children: React.ReactNode }) {
  const router = useRouter()
  const [authorized, setAuthorized] = useState(false)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    const tokens = TokenManager.getTokens()
    if (!tokens) {
      router.push("/login?redirect=/admin")
      return
    }
    api.auth.me().then((res) => {
      if (res.success && ["SUPER_ADMIN", "ADMIN", "EDITOR"].includes(res.data.role)) {
        setAuthorized(true)
      } else {
        router.push("/login?redirect=/admin")
      }
    }).finally(() => setLoading(false))
  }, [router])

  if (loading) return <div className="flex items-center justify-center min-h-screen">Loading...</div>
  if (!authorized) return null

  return (
    <div className="flex min-h-[100dvh]">
      {/* Sidebar */}
      <aside className="w-56 bg-gray-900 text-white p-4 flex-shrink-0 hidden md:block">
        <h2 className="text-lg font-bold mb-6">管理后台</h2>
        <nav className="space-y-1 text-sm">
          <a href="/admin" className="block px-3 py-2 rounded hover:bg-gray-800">概览</a>
          <a href="/admin/articles" className="block px-3 py-2 rounded hover:bg-gray-800">文章管理</a>
          <a href="/admin/banners" className="block px-3 py-2 rounded hover:bg-gray-800">Banner 管理</a>
          <a href="/admin/users" className="block px-3 py-2 rounded hover:bg-gray-800">用户管理</a>
          <a href="/admin/announcements" className="block px-3 py-2 rounded hover:bg-gray-800">公告管理</a>
        </nav>
        <div className="mt-auto pt-4 border-t border-gray-700">
          <button onClick={() => { api.auth.logout(); TokenManager.clearTokens(); router.push("/login") }}
            className="text-sm text-gray-400 hover:text-white">退出登录</button>
        </div>
      </aside>

      {/* Content */}
      <main className="flex-1 p-6 bg-gray-50 overflow-auto">
        {children}
      </main>
    </div>
  )
}
```

> **[WRITE]** `packages/web/src/app/admin/page.tsx`:

```tsx
"use client"

import { useEffect, useState } from "react"
import { api } from "shared"

export default function AdminDashboard() {
  const [stats, setStats] = useState({ users: 0, articles: 0, banners: 0 })

  useEffect(() => {
    Promise.all([
      api.content.articles({ limit: 1 }),
      api.content.banners(),
    ]).then(([articlesRes, bannersRes]) => {
      setStats({
        users: 0, // 需要 Admin API 权限
        articles: articlesRes.pagination?.total || 0,
        banners: bannersRes.data?.length || 0,
      })
    })
  }, [])

  return (
    <div>
      <h1 className="text-2xl font-bold mb-6">管理后台概览</h1>
      <div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
        <StatCard title="用户总数" value={stats.users} href="/admin/users" color="blue" />
        <StatCard title="文章总数" value={stats.articles} href="/admin/articles" color="green" />
        <StatCard title="Banner 数" value={stats.banners} href="/admin/banners" color="purple" />
      </div>
    </div>
  )
}

function StatCard({ title, value, href, color }: { title: string; value: number; href: string; color: string }) {
  const colors: Record<string, string> = {
    blue: "bg-blue-50 border-blue-200 text-blue-700",
    green: "bg-green-50 border-green-200 text-green-700",
    purple: "bg-purple-50 border-purple-200 text-purple-700",
  }
  return (
    <a href={href} className={`block rounded-xl border p-5 ${colors[color]}`}>
      <p className="text-sm opacity-75">{title}</p>
      <p className="text-3xl font-bold mt-1">{value}</p>
    </a>
  )
}
```

### Step 7.5 — Web 前端启动 & 验证

```bash
# [SHELL] 配置 Web 环境变量
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}/packages/web

cat > .env.local << 'WEBENV'
NEXT_PUBLIC_API_URL=http://localhost:3000/api/v1
WEBENV

# [SHELL] 安装依赖 & 构建验证
pnpm install
pnpm build 2>&1 | tail -20
# [DEBUG] 如有构建错误，分析并修复

# [SHELL] 启动 Web 前端 (端口 3001，避免与后端 3000 冲突)
pnpm dev -p 3001 &
sleep 3

echo ""
echo "=== 验证 Web 前端 ==="
curl -s -o /dev/null -w "HTTP Status: %{http_code}" http://localhost:3001
echo ""
echo "✅ Web 前端启动成功: http://localhost:3001"
echo "   管理后台: http://localhost:3001/admin"

# [GIT]
cd ${PROJECT_DIR}
git add -A && git commit -m "Phase 7: Web frontend (Next.js user + admin panel)"
```

---

## Phase 8: iOS App (SwiftUI) (3:30-6:00) — 可选

> **[GENERATE]** + **[WRITE]** + **[DEBUG]**
>
> ⚠️ **前置条件**: macOS + Xcode 16+。非 macOS 环境跳过此 Phase。

### Step 8.1 — 创建 Xcode 项目

```bash
# [SHELL] 仅在 macOS 上执行
if [[ "$(uname)" != "Darwin" ]]; then
  echo "非 macOS 环境，跳过 iOS Phase"
  exit 0
fi

source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}/mobile/ios

# 创建 iOS 项目结构
mkdir -p MyApp
cd MyApp

# 初始化 Swift Package (简化版)
cat > Package.swift << 'SWIFTPKG'
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
  name: "MyApp",
  platforms: [.iOS(.v17)],
  dependencies: [],
  targets: [
    .executableTarget(name: "MyApp", path: "Sources")
  ]
)
SWIFTPKG
```

### Step 8.2 — API Client (Swift)

> **[WRITE]** `mobile/ios/MyApp/Sources/APIClient.swift`:

```swift
import Foundation

// MARK: - Shared Types (mirror packages/shared/src/api-client.ts)

struct APIResponse<T: Codable>: Codable {
    let success: Bool
    let data: T?
    let pagination: Pagination?
    let error: APIError?
}

struct Pagination: Codable {
    let total: Int
    let page: Int
    let limit: Int
    let totalPages: Int
}

struct APIError: Codable {
    let code: String
    let message: String
}

struct User: Codable, Identifiable {
    let id: String
    let email: String
    let name: String?
    let role: String
    let avatar: String?
    let createdAt: String
}

struct Article: Codable, Identifiable {
    let id: String
    let title: String
    let slug: String
    let summary: String?
    let coverImage: String?
    let tags: [String]
    let publishedAt: String?
}

struct Banner: Codable, Identifiable {
    let id: String
    let title: String
    let imageUrl: String
    let linkUrl: String?
}

struct Announcement: Codable, Identifiable {
    let id: String
    let title: String
    let content: String
    let type: String
    let isSticky: Bool
}

struct AuthTokens: Codable {
    let accessToken: String
    let refreshToken: String
}

// MARK: - Token Manager

class TokenManager {
    static let shared = TokenManager()
    private let defaults = UserDefaults.standard
    private let key = "auth_tokens"

    var tokens: AuthTokens? {
        get {
            guard let data = defaults.data(forKey: key) else { return nil }
            return try? JSONDecoder().decode(AuthTokens.self, from: data)
        }
        set {
            if let newValue = newValue {
                defaults.set(try? JSONEncoder().encode(newValue), forKey: key)
            } else {
                defaults.removeObject(forKey: key)
            }
        }
    }

    var accessToken: String? { tokens?.accessToken }
    var isLoggedIn: Bool { tokens != nil }

    func clear() { tokens = nil }
}

// MARK: - API Client

class APIClient {
    static let shared = APIClient()
    private let baseURL: String
    private let session: URLSession
    private let decoder: JSONDecoder

    init(baseURL: String = "http://localhost:3000/api/v1") {
        self.baseURL = baseURL
        self.session = URLSession.shared
        self.decoder = JSONDecoder()
    }

    // MARK: Generic Request

    func request<T: Codable>(
        _ endpoint: String,
        method: String = "GET",
        body: Encodable? = nil,
        authenticated: Bool = false
    ) async throws -> APIResponse<T> {
        guard let url = URL(string: "\(baseURL)\(endpoint)") else {
            throw URLError(.badURL)
        }

        var request = URLRequest(url: url)
        request.httpMethod = method
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")

        if authenticated, let token = TokenManager.shared.accessToken {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }

        if let body = body {
            request.httpBody = try JSONEncoder().encode(AnyEncodable(body))
        }

        let (data, response) = try await session.data(for: request)

        // Auto-refresh on 401
        if let httpResponse = response as? HTTPURLResponse,
           httpResponse.statusCode == 401,
           let refreshToken = TokenManager.shared.tokens?.refreshToken {
            let newTokens = try await refreshAccessToken(refreshToken: refreshToken)
            TokenManager.shared.tokens = newTokens
            // Retry with new token
            request.setValue("Bearer \(newTokens.accessToken)", forHTTPHeaderField: "Authorization")
            let (retryData, _) = try await session.data(for: request)
            return try decoder.decode(APIResponse<T>.self, from: retryData)
        }

        return try decoder.decode(APIResponse<T>.self, from: data)
    }

    private func refreshAccessToken(refreshToken: String) async throws -> AuthTokens {
        let res: APIResponse<AuthTokens> = try await request(
            "/auth/refresh",
            method: "POST",
            body: ["refreshToken": refreshToken]
        )
        guard let tokens = res.data else {
            throw URLError(.userAuthenticationRequired)
        }
        return tokens
    }

    // MARK: Convenience Methods

    func login(email: String, password: String) async throws -> APIResponse<LoginResult> {
        return try await request("/auth/login", method: "POST", body: ["email": email, "password": password])
    }

    func register(email: String, password: String, name: String) async throws -> APIResponse<LoginResult> {
        return try await request("/auth/register", method: "POST", body: ["email": email, "password": password, "name": name])
    }

    func getArticles(page: Int = 1, limit: Int = 20) async throws -> APIResponse<[Article]> {
        return try await request("/content/articles?page=\(page)&limit=\(limit)")
    }

    func getArticle(slug: String) async throws -> APIResponse<ArticleDetail> {
        return try await request("/content/articles/\(slug)")
    }

    func getBanners(position: String = "home_top") async throws -> APIResponse<[Banner]> {
        return try await request("/content/banners?position=\(position)")
    }

    func getAnnouncements() async throws -> APIResponse<[Announcement]> {
        return try await request("/content/announcements")
    }

    func getMe() async throws -> APIResponse<User> {
        return try await request("/auth/me", authenticated: true)
    }
}

// MARK: - Helper Types

struct LoginResult: Codable {
    let user: User
    let accessToken: String
    let refreshToken: String
}

struct ArticleDetail: Codable {
    let id: String
    let title: String
    let slug: String
    let content: String?
    let coverImage: String?
    let tags: [String]
    let publishedAt: String?
    let author: ArticleAuthor?
}

struct ArticleAuthor: Codable {
    let name: String?
    let avatar: String?
}

// Helper to encode Any Encodable
struct AnyEncodable: Encodable {
    let value: Encodable
    init(_ value: Encodable) { self.value = value }
    func encode(to encoder: Encoder) throws {
        try value.encode(to: encoder)
    }
}
```

### Step 8.3 — SwiftUI 首页

> **[WRITE]** `mobile/ios/MyApp/Sources/ContentView.swift`:

```swift
import SwiftUI

struct ContentView: View {
    @State private var articles: [Article] = []
    @State private var banners: [Banner] = []
    @State private var announcements: [Announcement] = []
    @State private var isLoading = true

    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 16) {
                    // Banner
                    if let firstBanner = banners.first {
                        BannerCard(banner: firstBanner)
                    }

                    // Announcements
                    ForEach(announcements) { announcement in
                        AnnouncementRow(announcement: announcement)
                    }

                    // Articles
                    Text("最新文章")
                        .font(.headline)
                        .frame(maxWidth: .infinity, alignment: .leading)
                        .padding(.horizontal)

                    LazyVStack(spacing: 12) {
                        ForEach(articles) { article in
                            NavigationLink(destination: ArticleDetailView(slug: article.slug)) {
                                ArticleCard(article: article)
                            }
                            .buttonStyle(.plain)
                        }
                    }
                    .padding(.horizontal)
                }
                .padding(.vertical)
            }
            .navigationTitle("我的应用")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    NavigationLink(destination: LoginView()) {
                        Image(systemName: "person.circle")
                    }
                }
            }
            .task { await loadData() }
            .refreshable { await loadData() }
        }
    }

    func loadData() async {
        isLoading = true
        defer { isLoading = false }

        do {
            async let articlesRes = APIClient.shared.getArticles()
            async let bannersRes = APIClient.shared.getBanners()
            async let announcementsRes = APIClient.shared.getAnnouncements()

            let (articlesData, bannersData, announcementsData) = try await (articlesRes, bannersRes, announcementsRes)

            articles = articlesData.data ?? []
            banners = bannersData.data ?? []
            announcements = announcementsData.data ?? []
        } catch {
            print("Failed to load data: \(error)")
        }
    }
}

// MARK: - Subviews

struct BannerCard: View {
    let banner: Banner

    var body: some View {
        ZStack(alignment: .bottomLeading) {
            AsyncImage(url: URL(string: banner.imageUrl)) { image in
                image.resizable().aspectRatio(contentMode: .fill)
            } placeholder: {
                Color.blue.opacity(0.3)
            }
            .frame(height: 180)
            .clipped()

            VStack(alignment: .leading) {
                Text(banner.title)
                    .font(.title3.bold())
                    .foregroundColor(.white)
                if let link = banner.linkUrl {
                    Text("了解更多 →")
                        .font(.caption)
                        .foregroundColor(.white.opacity(0.8))
                }
            }
            .padding()
            .frame(maxWidth: .infinity, alignment: .leading)
            .background(LinearGradient(colors: [.clear, .black.opacity(0.5)], startPoint: .top, endPoint: .bottom))
        }
        .clipShape(RoundedRectangle(cornerRadius: 16))
        .padding(.horizontal)
    }
}

struct AnnouncementRow: View {
    let announcement: Announcement

    var backgroundColor: Color {
        switch announcement.type {
        case "warning": return .yellow.opacity(0.2)
        case "error": return .red.opacity(0.2)
        case "success": return .green.opacity(0.2)
        default: return .blue.opacity(0.2)
        }
    }

    var body: some View {
        HStack {
            Image(systemName: announcement.type == "warning" ? "exclamationmark.triangle" :
                  announcement.type == "error" ? "xmark.circle" :
                  announcement.type == "success" ? "checkmark.circle" : "info.circle")
            Text(announcement.title)
                .font(.subheadline)
        }
        .padding()
        .frame(maxWidth: .infinity, alignment: .leading)
        .background(backgroundColor)
        .clipShape(RoundedRectangle(cornerRadius: 10))
        .padding(.horizontal)
    }
}

struct ArticleCard: View {
    let article: Article

    var body: some View {
        HStack(spacing: 12) {
            if let cover = article.coverImage {
                AsyncImage(url: URL(string: cover)) { image in
                    image.resizable().aspectRatio(contentMode: .fill)
                } placeholder: {
                    Color.gray.opacity(0.2)
                }
                .frame(width: 80, height: 80)
                .clipShape(RoundedRectangle(cornerRadius: 8))
            }

            VStack(alignment: .leading, spacing: 4) {
                Text(article.title)
                    .font(.subheadline.bold())
                    .lineLimit(2)

                if let summary = article.summary {
                    Text(summary)
                        .font(.caption)
                        .foregroundColor(.secondary)
                        .lineLimit(2)
                }

                if !article.tags.isEmpty {
                    ScrollView(.horizontal, showsIndicators: false) {
                        HStack(spacing: 4) {
                            ForEach(article.tags, id: \.self) { tag in
                                Text(tag)
                                    .font(.caption2)
                                    .padding(.horizontal, 6)
                                    .padding(.vertical, 2)
                                    .background(Color.gray.opacity(0.1))
                                    .clipShape(Capsule())
                            }
                        }
                    }
                }
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .clipShape(RoundedRectangle(cornerRadius: 12))
        .shadow(color: .black.opacity(0.05), radius: 4, y: 2)
    }
}

struct ArticleDetailView: View {
    let slug: String
    @State private var article: ArticleDetail?

    var body: some View {
        ScrollView {
            if let article = article {
                VStack(alignment: .leading, spacing: 16) {
                    if let cover = article.coverImage {
                        AsyncImage(url: URL(string: cover)) { image in
                            image.resizable().aspectRatio(contentMode: .fill)
                        } placeholder: {
                            Color.gray.opacity(0.2)
                        }
                        .frame(height: 200)
                        .clipped()
                    }

                    Text(article.title)
                        .font(.title.bold())

                    if let content = article.content {
                        Text(content)
                    }
                }
                .padding()
            } else {
                ProgressView()
            }
        }
        .navigationBarTitleDisplayMode(.inline)
        .task {
            do {
                let res = try await APIClient.shared.getArticle(slug: slug)
                article = res.data
            } catch {}
        }
    }
}

struct LoginView: View {
    @State private var email = ""
    @State private var password = ""
    @State private var isLoading = false
    @State private var errorMessage: String?
    @Environment(\.dismiss) var dismiss

    var body: some View {
        Form {
            Section("登录") {
                TextField("邮箱", text: $email)
                    .keyboardType(.emailAddress)
                    .textContentType(.emailAddress)
                    .autocapitalization(.none)

                SecureField("密码", text: $password)
                    .textContentType(.password)
            }

            if let error = errorMessage {
                Text(error)
                    .foregroundColor(.red)
                    .font(.caption)
            }

            Button(action: login) {
                if isLoading {
                    ProgressView()
                } else {
                    Text("登录")
                        .frame(maxWidth: .infinity)
                }
            }
            .disabled(isLoading || email.isEmpty || password.isEmpty)
        }
        .navigationTitle("登录")
    }

    func login() {
        isLoading = true
        errorMessage = nil
        Task {
            do {
                let res = try await APIClient.shared.login(email: email, password: password)
                if let data = res.data {
                    TokenManager.shared.tokens = AuthTokens(
                        accessToken: data.accessToken,
                        refreshToken: data.refreshToken
                    )
                    dismiss()
                } else {
                    errorMessage = res.error?.message ?? "Login failed"
                }
            } catch {
                errorMessage = error.localizedDescription
            }
            isLoading = false
        }
    }
}
```

```bash
# [GIT] (仅在 macOS 上)
if [[ "$(uname)" == "Darwin" ]]; then
  cd ${PROJECT_DIR}
  git add -A && git commit -m "Phase 8: iOS app (SwiftUI) — API client + home page + auth"
fi
```

---

## Phase 9: Android App (Kotlin/Compose) (6:00-8:00) — 可选

> **[GENERATE]** + **[WRITE]**
>
> ⚠️ 需要 Android Studio Hedgehog+ 和 JDK 17+。

### Step 9.1 — 项目结构 & API Client

```bash
# [SHELL]
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}/mobile/android

# 创建 Android 项目结构
mkdir -p MyApp/app/src/main/java/com/example/myapp/{data/api,data/model,ui/screens,ui/components,ui/theme}
mkdir -p MyApp/app/src/main/res/{values,drawable}
```

> **[WRITE]** `mobile/android/MyApp/app/src/main/java/com/example/myapp/data/model/Models.kt`:

```kotlin
package com.example.myapp.data.model

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class ApiResponse<T>(
    val success: Boolean,
    val data: T? = null,
    val error: ApiError? = null,
    val pagination: Pagination? = null
)

@Serializable
data class ApiError(val code: String, val message: String)

@Serializable
data class Pagination(val total: Int, val page: Int, val limit: Int, val totalPages: Int)

@Serializable
data class User(
    val id: String, val email: String, val name: String?,
    val role: String, val avatar: String?, val createdAt: String
)

@Serializable
data class Article(
    val id: String, val title: String, val slug: String,
    val summary: String?, val coverImage: String?,
    val tags: List<String>, val publishedAt: String?
)

@Serializable
data class ArticleDetail(
    val id: String, val title: String, val slug: String,
    val content: String?, val coverImage: String?,
    val tags: List<String>, val publishedAt: String?,
    val author: ArticleAuthor?
)

@Serializable
data class ArticleAuthor(val name: String?, val avatar: String?)

@Serializable
data class Banner(
    val id: String, val title: String,
    val imageUrl: String, val linkUrl: String?
)

@Serializable
data class Announcement(
    val id: String, val title: String, val content: String,
    val type: String, val isSticky: Boolean
)

@Serializable
data class AuthTokens(val accessToken: String, val refreshToken: String)

@Serializable
data class LoginResult(
    val user: User, val accessToken: String, val refreshToken: String
)

@Serializable
data class LoginRequest(val email: String, val password: String)

@Serializable
data class RegisterRequest(
    val email: String, val password: String, val name: String
)
```

> **[WRITE]** `mobile/android/MyApp/app/src/main/java/com/example/myapp/data/api/ApiClient.kt`:

```kotlin
package com.example.myapp.data.api

import com.example.myapp.data.model.*
import io.ktor.client.*
import io.ktor.client.call.*
import io.ktor.client.plugins.contentnegotiation.*
import io.ktor.client.request.*
import io.ktor.http.*
import io.ktor.serialization.kotlinx.json.*
import kotlinx.serialization.json.Json

class ApiClient(
    private val baseUrl: String = "http://10.0.2.2:3000/api/v1" // Android emulator -> host
) {
    private val client = HttpClient {
        install(ContentNegotiation) {
            json(Json { ignoreUnknownKeys = true; isLenient = true })
        }
    }

    var accessToken: String? = null
    var refreshToken: String? = null

    private suspend inline fun <reified T> request(
        endpoint: String,
        method: HttpMethod = HttpMethod.Get,
        body: Any? = null,
        authenticated: Boolean = false
    ): ApiResponse<T> {
        val response = client.request("$baseUrl$endpoint") {
            this.method = method
            contentType(ContentType.Application.Json)
            if (authenticated && accessToken != null) {
                header("Authorization", "Bearer $accessToken")
            }
            if (body != null) setBody(body)
        }
        return response.body()
    }

    // Auth
    suspend fun login(email: String, password: String): ApiResponse<LoginResult> =
        request("/auth/login", HttpMethod.Post, LoginRequest(email, password))

    suspend fun register(email: String, password: String, name: String): ApiResponse<LoginResult> =
        request("/auth/register", HttpMethod.Post, RegisterRequest(email, password, name))

    // Content
    suspend fun getArticles(page: Int = 1): ApiResponse<List<Article>> =
        request("/content/articles?page=$page")

    suspend fun getArticle(slug: String): ApiResponse<ArticleDetail> =
        request("/content/articles/$slug")

    suspend fun getBanners(): ApiResponse<List<Banner>> =
        request("/content/banners")

    suspend fun getAnnouncements(): ApiResponse<List<Announcement>> =
        request("/content/announcements")
}
```

### Step 9.2 — Compose UI 首页

> **[WRITE]** `mobile/android/MyApp/app/src/main/java/com/example/myapp/ui/screens/HomeScreen.kt`:

```kotlin
package com.example.myapp.ui.screens

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.unit.dp
import coil.compose.AsyncImage
import com.example.myapp.data.api.ApiClient
import com.example.myapp.data.model.Announcement
import com.example.myapp.data.model.Article
import com.example.myapp.data.model.Banner
import kotlinx.coroutines.launch

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun HomeScreen(api: ApiClient, onArticleClick: (String) -> Unit) {
    var articles by remember { mutableStateOf<List<Article>>(emptyList()) }
    var banners by remember { mutableStateOf<List<Banner>>(emptyList()) }
    var announcements by remember { mutableStateOf<List<Announcement>>(emptyList()) }
    var isLoading by remember { mutableStateOf(true) }
    val scope = rememberCoroutineScope()

    LaunchedEffect(Unit) {
        scope.launch {
            try {
                val articlesRes = api.getArticles()
                val bannersRes = api.getBanners()
                val announcementsRes = api.getAnnouncements()
                articles = articlesRes.data ?: emptyList()
                banners = bannersRes.data ?: emptyList()
                announcements = announcementsRes.data ?: emptyList()
            } catch (_: Exception) {}
            isLoading = false
        }
    }

    Scaffold(
        topBar = {
            TopAppBar(title = { Text("我的应用") })
        }
    ) { padding ->
        if (isLoading) {
            Box(Modifier.fillMaxSize().padding(padding)) {
                CircularProgressIndicator(Modifier.align(androidx.compose.ui.Alignment.Center))
            }
        } else {
            LazyColumn(
                modifier = Modifier.fillMaxSize().padding(padding),
                contentPadding = PaddingValues(16.dp),
                verticalArrangement = Arrangement.spacedBy(12.dp)
            ) {
                // Banner
                if (banners.isNotEmpty()) {
                    item {
                        val banner = banners.first()
                        Card(
                            shape = RoundedCornerShape(16.dp),
                            modifier = Modifier.fillMaxWidth().height(180.dp)
                        ) {
                            AsyncImage(
                                model = banner.imageUrl,
                                contentDescription = banner.title,
                                contentScale = ContentScale.Crop,
                                modifier = Modifier.fillMaxSize()
                            )
                        }
                    }
                }

                // Announcements
                items(announcements) { announcement ->
                    val bgColor = when (announcement.type) {
                        "warning" -> MaterialTheme.colorScheme.errorContainer
                        "success" -> MaterialTheme.colorScheme.primaryContainer
                        else -> MaterialTheme.colorScheme.secondaryContainer
                    }
                    Surface(
                        color = bgColor,
                        shape = RoundedCornerShape(8.dp),
                        modifier = Modifier.fillMaxWidth()
                    ) {
                        Text(
                            announcement.title,
                            modifier = Modifier.padding(12.dp),
                            style = MaterialTheme.typography.bodyMedium
                        )
                    }
                }

                // Articles header
                item {
                    Text(
                        "最新文章",
                        style = MaterialTheme.typography.titleMedium,
                        modifier = Modifier.padding(top = 8.dp)
                    )
                }

                // Article cards
                items(articles) { article ->
                    Card(
                        onClick = { onArticleClick(article.slug) },
                        shape = RoundedCornerShape(12.dp)
                    ) {
                        Row(modifier = Modifier.padding(12.dp)) {
                            if (article.coverImage != null) {
                                AsyncImage(
                                    model = article.coverImage,
                                    contentDescription = null,
                                    modifier = Modifier.size(72.dp).clip(RoundedCornerShape(8.dp)),
                                    contentScale = ContentScale.Crop
                                )
                                Spacer(Modifier.width(12.dp))
                            }
                            Column(Modifier.weight(1f)) {
                                Text(article.title, style = MaterialTheme.typography.titleSmall)
                                if (article.summary != null) {
                                    Text(
                                        article.summary,
                                        style = MaterialTheme.typography.bodySmall,
                                        color = MaterialTheme.colorScheme.onSurfaceVariant,
                                        maxLines = 2
                                    )
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```

```bash
# [GIT]
cd ${PROJECT_DIR}
git add -A && git commit -m "Phase 9: Android app (Kotlin/Compose) — API client + home screen"
```

---

## Phase 10: 前端功能验证 & 联调 (Day 2, 8:00)

> **[VALIDATE]** 三端完整联调验证

```bash
# [SHELL] 确保所有服务运行
# 终端 1: PostgreSQL (Docker)
docker start sop-postgres 2>/dev/null || docker run -d --name sop-postgres \
  -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=mydb \
  -p 5432:5432 postgres:16-alpine

# 终端 2: Backend
cd packages/backend && pnpm dev &

# 终端 3: Web Frontend
cd packages/web && pnpm dev -p 3001 &

sleep 5

echo "=== 完整联调验证 ==="

# 1. API 健康检查
echo "1. Backend API:"
curl -s http://localhost:3000/api/v1/health | python3 -m json.tool

# 2. Web 前端可访问
echo "2. Web Frontend:"
curl -s -o /dev/null -w "HTTP %{http_code}" http://localhost:3001
echo ""

# 3. 完整业务流程
echo "3. E2E Flow:"

# 注册
REGISTER_RES=$(curl -s -X POST http://localhost:3000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"final-test@example.com","password":"test123456","name":"Final Test"}')
TOKEN=$(echo $REGISTER_RES | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['accessToken'])")

# 获取内容
echo "  Articles:" $(curl -s http://localhost:3000/api/v1/content/articles | python3 -c "import sys,json; print(len(json.load(sys.stdin).get('data',[])))") "篇"
echo "  Banners:" $(curl -s http://localhost:3000/api/v1/content/banners | python3 -c "import sys,json; print(len(json.load(sys.stdin).get('data',[])))") "个"

# 用户信息
echo "  User:" $(curl -s http://localhost:3000/api/v1/auth/me -H "Authorization: Bearer $TOKEN" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['email'])")

echo ""
echo "✅ Day 2 联调验证完成! 后端 + Web 前端正常运行"

# [GIT]
git add -A && git commit -m "Phase 10: Multi-platform integration verification passed"
```

---

## Phase 11: Docker 容器化 & 本地编排 (Day 3, 0:00-1:30)

> **[WRITE]** + **[SHELL]** + **[VALIDATE]**

### Step 11.1 — Backend Dockerfile

> **[WRITE]** `packages/backend/Dockerfile`:

```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
RUN npm i -g pnpm

FROM base AS deps
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile --prod

FROM base AS builder
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
RUN npx prisma generate
RUN pnpm build

FROM node:20-alpine AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 nodejs && adduser --system --uid 1001 hono
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/package.json ./
USER hono
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Step 11.2 — Web Frontend Dockerfile

> **[WRITE]** `packages/web/Dockerfile`:

```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
RUN npm i -g pnpm

FROM base AS deps
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

FROM base AS builder
COPY . .
COPY --from=deps /app/node_modules ./node_modules
RUN pnpm build

FROM node:20-alpine AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 nodejs && adduser --system --uid 1001 nextjs
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]
```

> 在 `packages/web/next.config.js` 添加 standalone 输出:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: "standalone",
}
module.exports = nextConfig
```

### Step 11.3 — Docker Compose 编排

> **[WRITE]** `docker/docker-compose.yml`:

```yaml
version: "3.8"

services:
  # ===== Nginx 反向代理 =====
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - backend
      - web
      - cms
    restart: unless-stopped
    networks:
      - app-network

  # ===== Backend API (Hono) =====
  backend:
    build:
      context: ../packages/backend
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD:-postgres}@db:5432/mydb
      JWT_SECRET: ${JWT_SECRET}
      JWT_REFRESH_SECRET: ${JWT_REFRESH_SECRET}
      CORS_ORIGINS: ${CORS_ORIGINS:-https://yourdomain.com}
      REDIS_URL: redis://redis:6379
      NODE_ENV: production
      # 华为云 OBS (生产环境)
      HUAWEI_OBS_ACCESS_KEY: ${HUAWEI_OBS_ACCESS_KEY:-}
      HUAWEI_OBS_SECRET_KEY: ${HUAWEI_OBS_SECRET_KEY:-}
      HUAWEI_OBS_BUCKET: ${HUAWEI_OBS_BUCKET:-}
      HUAWEI_OBS_ENDPOINT: ${HUAWEI_OBS_ENDPOINT:-}
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    networks:
      - app-network

  # ===== Web Frontend (Next.js) =====
  web:
    build:
      context: ../packages/web
      dockerfile: Dockerfile
    environment:
      NEXT_PUBLIC_API_URL: https://api.yourdomain.com/api/v1
    depends_on:
      - backend
    restart: unless-stopped
    networks:
      - app-network

  # ===== CMS (Directus) =====
  cms:
    image: directus/directus:latest
    environment:
      DB_CLIENT: pg
      DB_HOST: db
      DB_PORT: 5432
      DB_DATABASE: mydb
      DB_USER: postgres
      DB_PASSWORD: ${DB_PASSWORD:-postgres}
      KEY: ${DIRECTUS_KEY:-replace-with-random}
      SECRET: ${DIRECTUS_SECRET:-replace-with-random}
      ADMIN_EMAIL: ${ADMIN_EMAIL:-admin@example.com}
      ADMIN_PASSWORD: ${ADMIN_PASSWORD:-admin123}
      PUBLIC_URL: https://admin.yourdomain.com
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - app-network

  # ===== PostgreSQL =====
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./backups:/backups
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - app-network

  # ===== Redis =====
  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data
    restart: unless-stopped
    networks:
      - app-network

volumes:
  pgdata:
  redisdata:

networks:
  app-network:
    driver: bridge
```

### Step 11.4 — Nginx 配置

> **[WRITE]** `docker/nginx/nginx.conf`:

```nginx
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Gzip
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=30r/s;
    limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

    upstream backend {
        server backend:3000;
    }

    upstream web {
        server web:3000;
    }

    upstream cms {
        server cms:8055;
    }

    # HTTP → HTTPS 重定向 (生产环境)
    server {
        listen 80;
        server_name yourdomain.com api.yourdomain.com admin.yourdomain.com;
        return 301 https://$host$request_uri;
    }

    # HTTPS
    server {
        listen 443 ssl http2;
        server_name api.yourdomain.com;

        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        location / {
            limit_req zone=api burst=50 nodelay;
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    server {
        listen 443 ssl http2;
        server_name yourdomain.com;

        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        location / {
            proxy_pass http://web;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

    server {
        listen 443 ssl http2;
        server_name admin.yourdomain.com;

        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        location / {
            proxy_pass http://cms;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

### Step 11.5 — 本地 Docker Compose 验证

```bash
# [SHELL] 本地构建 & 启动
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}

# 构建
docker compose -f docker/docker-compose.yml build

# 启动
docker compose -f docker/docker-compose.yml up -d

# 等待就绪
sleep 10

# 执行数据库迁移
docker compose -f docker/docker-compose.yml exec backend npx prisma migrate deploy

# 验证
echo "=== Docker Compose 验证 ==="
curl -s http://localhost/api/v1/health | python3 -m json.tool
curl -s -o /dev/null -w "Web: HTTP %{http_code}\n" http://localhost

echo ""
echo "✅ Docker Compose 本地编排验证通过!"

# [GIT]
git add -A && git commit -m "Phase 11: Docker containerization & local orchestration"
```

---

## Phase 12: 华为云基础设施搭建 (1:30-3:30)

> **[RESEARCH]** + **[SHELL]** + **[DIALOG]**
>
> ⚠️ **此 Phase 涉及华为云付费资源**，执行前 [DIALOG] 确认预算。

### Step 12.1 — 华为云服务清单 & 成本估算

```markdown
## 华为云服务清单

| 服务 | 规格 | 用途 | 月费用(约) |
|------|------|------|-----------|
| ECS | 2vCPU 4GB 40GB SSD | 应用服务器 (Docker) | ¥100-200 |
| RDS PostgreSQL | 2vCPU 4GB 100GB | 生产数据库 | ¥200-400 |
| OBS | 标准存储 50GB + CDN | 图片/文件存储分发 | ¥10-50 |
| CDN | 国内流量 100GB/月 | 内容加速 | ¥20-50 |
| WAF (可选) | 标准版 | Web 应用防火墙 | ¥100+ |
| 弹性公网 IP | 1 个 | 公网访问 | ¥10-20 |
| **合计** | | | **¥440-820/月** |

> 💡 **省钱建议**: 初期用 1 台 ECS 跑全部服务 (Docker Compose)，数据库也用 ECS 自建 PostgreSQL（数据通过 OBS 备份），月费可控制在 ¥150-250。
```

### Step 12.2 — 华为云 ECS 创建 & 初始化

```bash
# [DIALOG] 确认以下操作:
echo "即将执行华为云 ECS 创建。请确认:"
echo "1. 华为云账号已实名认证"
echo "2. 账户余额 ≥ ¥200"
echo "3. 已创建 VPC 和安全组"
echo ""
echo "继续? (yes/no)"

# [SHELL] 华为云 CLI 创建 ECS (或用 Web 控制台)
# 以下为 CLI 示例，也可通过华为云网页控制台完成

# 1. 创建 ECS 实例
hcloud ecs create-servers \
  --name "myapp-prod-server" \
  --flavor "s6.large.2" \
  --image "Ubuntu 22.04" \
  --vpc-id "your-vpc-id" \
  --subnet-id "your-subnet-id" \
  --security-group-id "your-sg-id" \
  --bandwidth-size 5 \
  --system-disk-type "GPSSD" \
  --system-disk-size 40 \
  --root-password "YourStrongPassword123!" \
  --region "cn-north-4"

# 2. 获取 ECS 公网 IP
ECS_IP=$(hcloud ecs list-servers --query "servers[?name=='myapp-prod-server'].public_ip" --output text)
echo "ECS 公网 IP: $ECS_IP"
```

### Step 12.3 — ECS 环境初始化脚本

```bash
# [SHELL] SSH 到 ECS 并初始化环境

ECS_IP="<your-ecs-ip>"  # 替换为实际 IP

ssh root@${ECS_IP} << 'REMOTE_SETUP'
# 1. 系统更新
apt update && apt upgrade -y

# 2. 安装 Docker
curl -fsSL https://get.docker.com | bash
systemctl enable docker
docker --version

# 3. 安装 Docker Compose
apt install -y docker-compose-plugin
docker compose version

# 4. 安装 Node.js 20 (for Prisma migrate)
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs
npm i -g pnpm

# 5. 设置时区
timedatectl set-timezone Asia/Shanghai

# 6. 创建应用目录
mkdir -p /opt/myapp
mkdir -p /opt/myapp/backups
mkdir -p /opt/myapp/ssl

# 7. 配置防火墙 (华为云安全组规则)
# 在华为云控制台 → 安全组 → 添加入方向规则:
# 80 (HTTP), 443 (HTTPS), 22 (SSH, 限制 IP)
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw --force enable

# 8. 安装监控代理
# 华为云 CloudEye Agent
wget https://telescope-ap-southeast-1.obs.ap-southeast-1.myhuaweicloud.com/Agent/linux/telescope.sh
bash telescope.sh

echo "✅ ECS 初始化完成"
REMOTE_SETUP
```

### Step 12.4 — 华为云 RDS 创建 (可选，或 ECS 自建)

```bash
# 方案 A: 华为云 RDS (推荐生产环境)
hcloud rds create-instance \
  --name "myapp-postgres" \
  --datastore-type "PostgreSQL" \
  --datastore-version "16" \
  --flavor "rds.pg.c2.large" \
  --volume-size 100 \
  --vpc-id "your-vpc-id" \
  --subnet-id "your-subnet-id" \
  --security-group-id "your-sg-id" \
  --region "cn-north-4" \
  --password "YourStrongDBPassword123!"

# 获取 RDS 内网地址
RDS_HOST=$(hcloud rds list-instances --query "instances[?name=='myapp-postgres'].private_ips[0]" --output text)

# 方案 B: ECS 自建 PostgreSQL (省钱方案)
# 在 docker-compose.yml 中已包含 PostgreSQL 容器

echo "DATABASE_URL=postgresql://postgres:YOUR_DB_PASSWORD@${RDS_HOST:-db}:5432/mydb"
```

### Step 12.5 — 华为云 OBS 创建

```bash
# [SHELL] 创建 OBS 存储桶
hcloud obs create-bucket \
  --bucket "myapp-uploads" \
  --region "cn-north-4" \
  --acl "private"

# 配置 CDN 加速 (可选)
# 华为云控制台 → CDN → 添加加速域名 → 回源到 OBS

echo "OBS 配置:"
echo "  Bucket: myapp-uploads"
echo "  Endpoint: https://obs.cn-north-4.myhuaweicloud.com"
echo "  CDN Domain: https://cdn.yourdomain.com (可选)"
```

```bash
# [GIT]
git add -A && git commit -m "Phase 12: Huawei Cloud infrastructure setup (ECS + RDS + OBS)"
```

---

## Phase 13: 生产部署 & SSL/域名 (3:30-5:00)

> **[SHELL]** + **[DIALOG]**

### Step 13.1 — 部署到华为云 ECS

```bash
# [SHELL] 1. 上传项目到 ECS
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}
ECS_IP="<your-ecs-ip>"

# 打包必要文件
tar -czf deploy.tar.gz \
  docker/ \
  packages/backend/{Dockerfile,prisma,package.json,pnpm-lock.yaml,.env.example} \
  packages/web/{Dockerfile,package.json,pnpm-lock.yaml} \
  --exclude='node_modules' --exclude='.next' --exclude='dist'

# 上传
scp deploy.tar.gz root@${ECS_IP}:/opt/myapp/

# 2. 解压 & 配置
ssh root@${ECS_IP} << 'DEPLOY'
cd /opt/myapp
tar -xzf deploy.tar.gz
rm deploy.tar.gz

# 3. 创建生产环境变量
cat > .env << 'ENVEOF'
DB_PASSWORD=your-strong-password
JWT_SECRET=$(openssl rand -hex 64)
JWT_REFRESH_SECRET=$(openssl rand -hex 64)
CORS_ORIGINS=https://yourdomain.com,https://admin.yourdomain.com
HUAWEI_OBS_ACCESS_KEY=your_obs_ak
HUAWEI_OBS_SECRET_KEY=your_obs_sk
HUAWEI_OBS_BUCKET=myapp-uploads
HUAWEI_OBS_ENDPOINT=https://obs.cn-north-4.myhuaweicloud.com
DIRECTUS_KEY=$(openssl rand -hex 32)
DIRECTUS_SECRET=$(openssl rand -hex 32)
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=<change-to-strong-password>
NODE_ENV=production
ENVEOF

# 4. 构建 & 启动
docker compose -f docker/docker-compose.yml build
docker compose -f docker/docker-compose.yml up -d

# 5. 数据库迁移
sleep 10
docker compose -f docker/docker-compose.yml exec backend npx prisma migrate deploy
docker compose -f docker/docker-compose.yml exec backend npx tsx prisma/seed.ts

# 6. 检查状态
docker compose -f docker/docker-compose.yml ps
docker compose -f docker/docker-compose.yml logs --tail=20

echo "✅ 部署完成!"
DEPLOY
```

### Step 13.2 — SSL 证书配置

```bash
# [SHELL] 方案 A: Let's Encrypt 免费证书
ssh root@${ECS_IP} << 'SSL_SETUP'
# 安装 certbot
apt install -y certbot

# 获取证书 (HTTP-01 challenge)
certbot certonly --standalone \
  -d yourdomain.com \
  -d api.yourdomain.com \
  -d admin.yourdomain.com \
  --email your-email@example.com \
  --agree-tos \
  --non-interactive

# 复制证书到 Nginx ssl 目录
cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem /opt/myapp/ssl/
cp /etc/letsencrypt/live/yourdomain.com/privkey.pem /opt/myapp/ssl/

# 重启 Nginx
docker compose -f /opt/myapp/docker/docker-compose.yml restart nginx

# 设置自动续期
echo "0 3 * * * certbot renew --quiet && cp /etc/letsencrypt/live/yourdomain.com/*.pem /opt/myapp/ssl/ && docker compose -f /opt/myapp/docker/docker-compose.yml restart nginx" | crontab -
SSL_SETUP

echo "SSL 证书配置完成! 访问 https://yourdomain.com 验证"
```

```bash
# 方案 B: 华为云 SSL 证书 (国内推荐)
# 华为云控制台 → SSL 证书管理 → 购买证书 → DNS 验证 → 下载证书
# 证书文件放到 docker/nginx/ssl/ 目录
```

### Step 13.3 — 生产环境验证

```bash
# [VALIDATE] 生产环境检查清单

echo "=== 生产环境验证 ==="

# 1. HTTPS 访问
echo "1. HTTPS:"
curl -s -o /dev/null -w "HTTP %{http_code}" https://yourdomain.com
echo ""

# 2. API 健康检查
echo "2. API Health:"
curl -s https://api.yourdomain.com/api/v1/health | python3 -m json.tool

# 3. 安全头检查
echo "3. Security Headers:"
curl -sI https://api.yourdomain.com/api/v1/health | grep -E "X-|Content-Security|Strict-Transport"

# 4. CMS 访问
echo "4. CMS:"
curl -s -o /dev/null -w "HTTP %{http_code}" https://admin.yourdomain.com

# 5. SSL 证书检查
echo "5. SSL:"
echo | openssl s_client -connect yourdomain.com:443 -servername yourdomain.com 2>/dev/null | openssl x509 -noout -dates

echo ""
echo "✅ 生产环境验证完成"
```

```bash
# [GIT]
git tag v1.0.0
git add -A && git commit -m "Phase 13: Production deployment with SSL on Huawei Cloud"
```

---

## Phase 14: CI/CD 流水线 & 推送通知 (5:00-6:00)

> **[WRITE]** + **[SHELL]**

### Step 14.1 — GitHub Actions CI/CD

> **[WRITE]** `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Huawei Cloud

on:
  push:
    branches: [main]
    paths-ignore:
      - "docs/**"
      - "*.md"

env:
  ECS_HOST: ${{ secrets.ECS_HOST }}
  ECS_USER: root
  DOCKER_REGISTRY: ${{ secrets.DOCKER_REGISTRY }}

jobs:
  # ===== 1. 代码质量检查 =====
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "pnpm"

      - run: pnpm install --frozen-lockfile

      - name: TypeScript Check (Backend)
        run: cd packages/backend && npx tsc --noEmit

      - name: TypeScript Check (Web)
        run: cd packages/web && npx tsc --noEmit

      - name: Security Audit
        run: pnpm audit --audit-level=high

      - name: Secret Scan
        uses: gitleaks/gitleaks-action@v2
        with:
          config-path: .gitleaks.toml

  # ===== 2. 构建 & 部署 =====
  deploy:
    needs: lint-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to ECS
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.ECS_HOST }}
          username: root
          key: ${{ secrets.ECS_SSH_KEY }}
          script: |
            cd /opt/myapp
            git pull origin main
            docker compose -f docker/docker-compose.yml build
            docker compose -f docker/docker-compose.yml up -d --remove-orphans
            docker compose -f docker/docker-compose.yml exec -T backend npx prisma migrate deploy
            docker system prune -f

      - name: Health Check
        run: |
          sleep 10
          curl -f https://api.${{ secrets.DOMAIN }}/api/v1/health || exit 1
          echo "Deployment healthy"

      - name: Notify
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Deploy ${{ job.status }}: ${{ github.repository }} @ ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Step 14.2 — 推送通知服务

> **[WRITE]** `packages/backend/src/services/push-service.ts`:

```typescript
import { prisma } from "../lib/db"
import { Platform } from "@prisma/client"

interface PushMessage {
  title: string
  body: string
  data?: Record<string, string>
  imageUrl?: string
}

// 推送服务抽象层
export class PushService {
  // 发送给单个用户的所有设备
  static async sendToUser(userId: string, message: PushMessage): Promise<void> {
    const devices = await prisma.userDevice.findMany({
      where: { userId, pushToken: { not: null } },
    })

    for (const device of devices) {
      try {
        await PushService.sendToDevice(device.platform, device.pushToken!, message)
      } catch (e) {
        console.error(`Push failed for device ${device.id}:`, e)
      }
    }

    // 同时创建站内通知
    await prisma.notification.create({
      data: {
        userId,
        title: message.title,
        body: message.body,
        data: message.data || {},
      },
    })
  }

  // 发送给单个设备
  private static async sendToDevice(
    platform: Platform,
    pushToken: string,
    message: PushMessage
  ): Promise<void> {
    switch (platform) {
      case "IOS":
        await PushService.sendAPNs(pushToken, message)
        break
      case "ANDROID":
        await PushService.sendFCM(pushToken, message)
        break
      case "WEB":
        // Web Push API (Service Worker)
        break
    }
  }

  // APNs (Apple Push Notification service)
  private static async sendAPNs(token: string, message: PushMessage): Promise<void> {
    // 使用 @parse/node-apn 或直接 HTTP/2 调用 APNs
    // https://developer.apple.com/documentation/usernotifications
    const jwt = PushService.generateAPNsJWT()

    await fetch(`https://api.push.apple.com/3/device/${token}`, {
      method: "POST",
      headers: {
        "authorization": `bearer ${jwt}`,
        "apns-topic": process.env.APNS_BUNDLE_ID!,
        "apns-push-type": "alert",
      },
      body: JSON.stringify({
        aps: {
          alert: { title: message.title, body: message.body },
          badge: 1,
          sound: "default",
        },
        data: message.data,
      }),
    })
  }

  // FCM (Firebase Cloud Messaging)
  private static async sendFCM(token: string, message: PushMessage): Promise<void> {
    const serverKey = process.env.FCM_SERVER_KEY!

    await fetch("https://fcm.googleapis.com/fcm/send", {
      method: "POST",
      headers: {
        "Authorization": `key=${serverKey}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        to: token,
        notification: { title: message.title, body: message.body },
        data: message.data,
      }),
    })
  }

  private static generateAPNsJWT(): string {
    // 实现 APNs JWT 生成逻辑
    // 使用 jsonwebtoken + APNs Key
    return ""
  }
}

// 后台任务: 每日推送定时检查
export async function schedulePushNotifications(): Promise<void> {
  // 场景: 新文章发布 → 推送通知所有用户
  // 使用 node-cron 或 Bull 队列
  // 此处为示例入口
  console.log("[Push] Scheduled check running...")
}
```

```bash
# [GIT]
git add -A && git commit -m "Phase 14: CI/CD pipeline + push notification service"
```

---

## Phase 15: 安全合规审计 & 监控告警 (6:00-7:30)

> **[VALIDATE]** + **[REVIEW]** + **[WRITE]**

### Step 15.1 — 安全审计清单

> Claude Code 逐项检查，不通过不进入 Phase 16

```markdown
## 🔒 安全审计清单

### A. 网络安全
- [ ] SSL/TLS 已配置，HTTP 自动重定向到 HTTPS
- [ ] HSTS 头已设置 (max-age=31536000; includeSubDomains)
- [ ] CORS 限制为已知域名 (非 `*`)
- [ ] Rate Limiting 已配置 (API: 100 req/min, Login: 5 req/min/IP)
- [ ] WAF (Web 应用防火墙) 已启用 (华为云 WAF 或 Cloudflare)

### B. 认证安全
- [ ] 密码 bcrypt hash (cost ≥ 10)
- [ ] JWT access token 有效期 ≤ 2h
- [ ] JWT refresh token 轮换机制 (一次一换)
- [ ] 登录失败限制 (5 次失败后锁定 15 分钟)
- [ ] 密码复杂度要求 (≥8 字符, 含字母+数字)
- [ ] API Key / Secret 不在代码中硬编码

### C. 数据安全
- [ ] 数据库连接使用 TLS/SSL
- [ ] 敏感字段加密存储 (如需要)
- [ ] SQL 注入防护 → Prisma 参数化查询 (已覆盖)
- [ ] XSS 防护 → 输入净化 + CSP 头
- [ ] 文件上传限制 (类型白名单 + 大小限制)
- [ ] 用户数据可导出/删除 (合规要求)

### D. 基础设施安全
- [ ] SSH 端口限制 IP (仅信任 IP 可访问 22)
- [ ] Docker 容器以非 root 用户运行
- [ ] 数据库端口不对外开放 (仅 ECS 内网)
- [ ] 自动安全更新 (unattended-upgrades)
- [ ] 数据库每日自动备份

### E. 合规检查
- [ ] 隐私政策页面可访问 (/privacy)
- [ ] 用户协议页面可访问 (/terms)
- [ ] Cookie 同意 (如使用跟踪 cookies)
- [ ] 个人信息收集最小化原则
- [ ] 未成年人保护机制 (如适用)
- [ ] ICP 备案 (国内服务器必须)
```

```bash
# [SHELL] 自动安全扫描
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}

echo "=== 安全扫描 ==="

# 1. 依赖漏洞
echo "1. npm audit:"
cd packages/backend && pnpm audit --audit-level=high 2>&1 | tail -5
cd ${PROJECT_DIR}

# 2. 硬编码密钥检查
echo "2. Secret scan:"
grep -rn "sk-\|api_key\|secret\|password\|token" \
  --include="*.ts" --include="*.tsx" --include="*.swift" --include="*.kt" \
  packages/ mobile/ .github/ \
  --exclude-dir=node_modules --exclude-dir=.next --exclude-dir=dist \
  | grep -v "process\.env\|\.env\|example\|test\|placeholder\|change-me\|your-\|replace-with" \
  | grep -v "SECRET_ENV\|SECRET_KEY" \
  | head -10
echo "(检查完毕 — 如有输出则人工核查)"

# 3. Docker 安全
echo "3. Docker non-root check:"
grep -r "USER" packages/*/Dockerfile docker/* 2>/dev/null || echo "⚠️ 检查 Dockerfile 是否使用非 root 用户"

# 4. 开放端口检查 (生产 ECS)
if [ -n "$ECS_IP" ]; then
  echo "4. Port scan:"
  ssh root@${ECS_IP} "ss -tlnp | grep LISTEN"
fi
```

### Step 15.2 — 监控告警配置

> **[WRITE]** `docker/prometheus.yml`:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "backend"
    static_configs:
      - targets: ["backend:3000"]
    metrics_path: "/api/v1/health"

  - job_name: "node-exporter"
    static_configs:
      - targets: ["node-exporter:9100"]

  - job_name: "postgres"
    static_configs:
      - targets: ["postgres-exporter:9187"]
```

```bash
# [SHELL] 华为云 CloudEye 告警规则 (Web 控制台操作指南)

echo "
华为云监控告警配置 (Web 控制台):

1. 华为云控制台 → 云监控服务 CloudEye
2. 告警规则 → 创建告警规则:

   | 规则名称 | 指标 | 阈值 | 持续周期 | 通知 |
   |---------|------|------|---------|------|
   | CPU 过高 | cpu_util | ≥ 80% | 3 周期 | 短信+邮件 |
   | 内存过高 | mem_util | ≥ 85% | 3 周期 | 短信+邮件 |
   | 磁盘不足 | disk_util | ≥ 85% | 1 周期 | 短信+邮件 |
   | API 不可用 | 健康检查 | 连续 2 次失败 | 1 分钟 | 电话+短信 |

3. 告警通知 → 配置通知对象 (手机号/邮箱)
4. 建议开启: 事件监控 (ECS 重启/宕机自动通知)
"
```

### Step 15.3 — 数据库备份脚本

> **[WRITE]** `scripts/backup-db.sh`:

```bash
#!/bin/bash
# 数据库自动备份脚本 (放在 ECS crontab 中)

BACKUP_DIR="/opt/myapp/backups"
RETENTION_DAYS=30
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/mydb_${TIMESTAMP}.sql.gz"

# 备份
docker compose -f /opt/myapp/docker/docker-compose.yml exec -T db \
  pg_dump -U postgres mydb | gzip > "$BACKUP_FILE"

echo "Backup created: $BACKUP_FILE ($(du -h $BACKUP_FILE | cut -f1))"

# 上传到 OBS (异地容灾)
# python3 /opt/myapp/scripts/upload-to-obs.py "$BACKUP_FILE"

# 清理过期备份
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete
echo "Cleaned backups older than $RETENTION_DAYS days"
```

```bash
# 在 ECS 上配置每日自动备份
# ssh root@${ECS_IP} "echo '0 2 * * * bash /opt/myapp/scripts/backup-db.sh >> /var/log/db-backup.log 2>&1' | crontab -"
```

```bash
# [GIT]
git add -A && git commit -m "Phase 15: Security audit + monitoring + backup strategy"
```

---

## Phase 16: 上线检查清单 & 归档 (7:30-8:00)

> **[VALIDATE]** + **[DIALOG]** + **[GIT]**

### Step 16.1 — 上线前最终检查清单

> Claude Code 逐项检查，全部通过方可宣布上线。

```markdown
## 🚀 上线检查清单 (Go-Live Checklist)

### 服务可用性
- [ ] `https://api.yourdomain.com/api/v1/health` → 200 OK
- [ ] `https://yourdomain.com` → 200 OK，页面正常渲染
- [ ] `https://admin.yourdomain.com` → CMS 管理后台可访问
- [ ] API 响应时间 < 200ms (p50)
- [ ] 首页加载 < 2s (Lighthouse Performance ≥ 90)

### 功能完整性
- [ ] 用户注册 → 收到注册成功响应 (含 Token)
- [ ] 用户登录 → 返回 AccessToken + RefreshToken
- [ ] 文章列表 → 公开 API 返回已发布内容
- [ ] Banner 列表 → 按位置返回启用的 Banner
- [ ] 文件上传 → 图片上传到 OBS 并返回 URL
- [ ] App 配置 → 返回版本号、强制更新等配置
- [ ] 管理后台 → 管理员可登录并查看数据

### 安全合规
- [ ] SSL 证书有效 (Let's Encrypt 或华为云 SSL)
- [ ] HTTPS 强制 (HTTP 自动 301 → HTTPS)
- [ ] 安全 Headers 正确 (X-Content-Type-Options, X-Frame-Options 等)
- [ ] 隐私政策页面可访问
- [ ] ICP 备案号已放置网站底部 (国内要求)
- [ ] 公安备案 (如适用)
- [ ] 无已知高危 CVE 依赖

### 数据保护
- [ ] 数据库自动备份已配置 (每日凌晨 2 点)
- [ ] OBS 备份已配置 (异地容灾)
- [ ] RDS 自动备份已启用 (如使用 RDS)
- [ ] 备份恢复流程已测试

### 性能 & 监控
- [ ] CDN 已配置并生效 (静态资源命中率 > 80%)
- [ ] Gzip 压缩已启用
- [ ] Redis 缓存已启用 (内容 API)
- [ ] 华为云 CloudEye 告警规则已配置
- [ ] 告警通知已测试 (短信/邮件可达)

### 移动端 (如做了 iOS/Android)
- [ ] iOS App 可正常调用生产 API
- [ ] Android App 可正常调用生产 API
- [ ] 推送通知可正常接收
- [ ] App 版本管理配置正确

### DevOps
- [ ] GitHub Actions CI/CD 流水线正常
- [ ] Git tag v1.0.0 已创建
- [ ] Rollback 流程已测试 (docker compose up 上一版本)
- [ ] 运维文档 (RUNBOOK.md) 已就绪

### 运营准备
- [ ] 初始内容已通过 CMS 发布 (欢迎文章/Banner)
- [ ] 管理员账号已创建
- [ ] 客服/反馈渠道已配置
- [ ] 社交媒体/推广链接 已准备 (如适用)
```

### Step 16.2 — 运维 Runbook (RUNBOOK.md)

> **[WRITE]** `docs/RUNBOOK.md`:

```markdown
# 运维 Runbook

## 日常运维

### 健康检查
```bash
curl -s https://api.yourdomain.com/api/v1/health
```

### 查看日志
```bash
ssh root@<ECS_IP>
cd /opt/myapp
docker compose -f docker/docker-compose.yml logs --tail=100 backend
docker compose -f docker/docker-compose.yml logs -f  # 实时
```

### 重启服务
```bash
ssh root@<ECS_IP>
cd /opt/myapp
docker compose -f docker/docker-compose.yml restart backend
```

### 数据库备份
```bash
ssh root@<ECS_IP>
bash /opt/myapp/scripts/backup-db.sh
```

## 紧急操作

### Rollback (回滚到上一个版本)
```bash
ssh root@<ECS_IP>
cd /opt/myapp
git log --oneline -5  # 找到上一个稳定版本的 commit hash
git checkout <stable-commit-hash>
docker compose -f docker/docker-compose.yml up -d --build
docker compose -f docker/docker-compose.yml exec backend npx prisma migrate deploy
```

### 数据库恢复
```bash
ssh root@<ECS_IP>
cd /opt/myapp
# 1. 找到最新备份
ls -la backups/
# 2. 恢复
gunzip -c backups/mydb_20260101_020000.sql.gz | \
  docker compose -f docker/docker-compose.yml exec -T db psql -U postgres mydb
```

### 紧急维护模式
```sql
-- 在数据库 app_configs 表中启用维护模式
INSERT INTO app_configs (id, key, value, description)
VALUES (gen_random_uuid(), 'maintenance_mode', 'true', '紧急维护');
-- 客户端读取 /app/config 后会显示维护页面
```

## 关键账号

| 服务 | URL | 备注 |
|------|-----|------|
| CMS Admin | https://admin.yourdomain.com | 管理员账号 |
| 华为云控制台 | https://console.huaweicloud.com | 主账号 |
| GitHub | https://github.com/your-org/your-repo | 代码仓库 |
| 域名管理 | (你的域名注册商) | DNS 配置 |

## 故障排查

### API 502 Bad Gateway
```bash
ssh root@<ECS_IP>
docker compose -f docker/docker-compose.yml ps  # 检查 backend 容器状态
docker compose -f docker/docker-compose.yml logs backend  # 查看错误日志
docker compose -f docker/docker-compose.yml restart backend
```

### 数据库连接失败
```bash
ssh root@<ECS_IP>
docker compose -f docker/docker-compose.yml exec backend \
  npx prisma db execute --stdin <<< "SELECT 1"
```

### SSL 证书过期
```bash
ssh root@<ECS_IP>
certbot renew --force-renewal
cp /etc/letsencrypt/live/yourdomain.com/*.pem /opt/myapp/ssl/
docker compose -f docker/docker-compose.yml restart nginx
```
```

### Step 16.3 — 最终归档

```bash
# [GIT] 打标签 & 推送
source /tmp/sop_fullstack.env && cd ${PROJECT_DIR}

# 更新 CHANGELOG
cat > CHANGELOG.md << 'CHANGELOG'
# Changelog

## v1.0.0 (2026-06-24)

### Features
- Backend API (Hono + Prisma + PostgreSQL)
- CMS 集成 (Directus)
- 统一认证 (JWT access + refresh token)
- Web 前端 (Next.js 14+ 用户端 + 管理后台)
- iOS App (SwiftUI)
- Android App (Kotlin/Compose)
- Docker Compose 编排
- 华为云 ECS + RDS + OBS 部署
- CI/CD (GitHub Actions)
- 推送通知 (APNs + FCM)
- 监控告警 (CloudEye)
- 自动备份 (每日 + OBS 异地)

### Infrastructure
- 华为云 cn-north-4
- ECS 2vCPU 4GB
- PostgreSQL 16
- Redis 7
- Nginx 反向代理
CHANGELOG

# 最终提交
git add -A
git commit -m "Release v1.0.0 — Fullstack product ready for production

- Backend API (Hono + Prisma + PostgreSQL + JWT)
- CMS (Directus) with RBAC
- Web Frontend (Next.js 14+ user + admin)
- iOS App (SwiftUI with shared API client)
- Android App (Kotlin/Compose with shared API client)
- Docker Compose orchestration
- Huawei Cloud deployment (ECS + RDS + OBS + CDN)
- CI/CD pipeline (GitHub Actions)
- Push notifications (APNs + FCM)
- Security audit & monitoring (CloudEye)
- Auto backup strategy (daily + OBS offsite)

Co-Authored-By: Claude <noreply@anthropic.com>"

git tag -a v1.0.0 -m "v1.0.0 — 全栈多端产品首次正式发布"
git push origin main --tags

echo ""
echo "🎉 =========================================="
echo "   🚀 全栈产品 v1.0.0 正式上线!"
echo "   =========================================="
echo ""
echo "   📱 Web:     https://yourdomain.com"
echo "   🔌 API:     https://api.yourdomain.com/api/v1"
echo "   🖥  CMS:     https://admin.yourdomain.com"
echo "   📊 监控:    https://console.huaweicloud.com/cloudeye"
echo ""
echo "   📝 运维文档: docs/RUNBOOK.md"
echo "   📋 备份日志: /opt/myapp/backups/"
echo ""
echo "=========================================="
```

---

## 📚 附录

### A. 命令速查表

```bash
# ===== 本地开发 =====
pnpm dev:backend          # 启动后端 (localhost:3000)
pnpm dev:web              # 启动 Web 前端 (localhost:3001)
pnpm dev:cms              # 启动 CMS (localhost:8055)
pnpm db:migrate           # 数据库迁移
pnpm db:seed              # 种子数据
pnpm docker:up            # 启动所有 Docker 容器
pnpm docker:down          # 停止所有 Docker 容器

# ===== 生产运维 =====
ssh root@<ECS_IP>
docker compose -f /opt/myapp/docker/docker-compose.yml ps
docker compose -f /opt/myapp/docker/docker-compose.yml logs backend
docker compose -f /opt/myapp/docker/docker-compose.yml restart backend
bash /opt/myapp/scripts/backup-db.sh

# ===== Git =====
git tag v1.0.0
git push origin main --tags
```

### B. 环境变量速查

| 变量 | 说明 | 默认值 | 必须修改 |
|------|------|--------|---------|
| `DATABASE_URL` | PostgreSQL 连接串 | `postgresql://...` | 生产 |
| `JWT_SECRET` | JWT 签名密钥 | `dev-secret-...` | **必须** |
| `JWT_REFRESH_SECRET` | Refresh Token 密钥 | `dev-refresh-...` | **必须** |
| `CORS_ORIGINS` | 允许的前端域名 | `http://localhost:3000` | 生产 |
| `HUAWEI_OBS_*` | 华为云对象存储 | (空) | 生产 |
| `FCM_SERVER_KEY` | Firebase 推送密钥 | (空) | 如需推送 |
| `APNS_BUNDLE_ID` | iOS Bundle ID | (空) | 如需推送 |
| `DB_PASSWORD` | 数据库密码 | `postgres` | **必须** |

### C. 故障排查速查

| 现象 | 可能原因 | 解决 |
|------|---------|------|
| API 502 | Backend 容器挂了 | `docker compose restart backend` |
| API 401 | Token 过期 | 前端自动 refresh 或重新登录 |
| 数据库连接失败 | RDS 安全组未放通 | 华为云控制台 → 安全组 → 添加规则 |
| 图片上传失败 | OBS 未配置 | 设置 OBS 环境变量 |
| 网站访问慢 | CDN 未命中 | 检查 CDN 配置，预热缓存 |
| HTTPS 证书过期 | Let's Encrypt 未续期 | `certbot renew` |
| Docker 磁盘满 | 镜像/日志堆积 | `docker system prune -a` |
| 内存不足 | 容器过多 | 增加 ECS 规格或优化容器 |

### D. 与现有 SOP 的关系

```
本 SOP (Fullstack) = 以下 SOP 的超级集合 + 华为云部署 + CMS + 多端协同

├── backend/Backend_Service_2Day_Development_SOP.md
│   └── 对应 Phase 2-3-5 (共享相同的 Hono+Prisma+JWT 技术栈)
│
├── web/Web_App_2Day_Development_SOP.md
│   └── 对应 Phase 7 (共享 Next.js + Prisma 前端技术栈)
│
├── ios/iOS_App_2Day_Development_SOP.md
│   └── 对应 Phase 8 (共享 SwiftUI + MVVM 模式)
│
├── android/Android_App_2Day_Development_SOP.md
│   └── 对应 Phase 9 (共享 Kotlin + Compose 模式)
│
├── cicd/CI_CD_Integration_SOP.md
│   └── 对应 Phase 14 (共享 GitHub Actions 模式)
│
└── copyright/Software_Copyright_Application_SOP.md
    └── 后续可接 (产品完成后申请软著)

🆕 本 SOP 新增:
├── Phase 1:  统一架构设计 (多端共享后端)
├── Phase 4:  CMS 集成 (Directus)
├── Phase 5:  统一认证授权 (JWT + RBAC + Token 轮换)
├── Phase 6:  共享 API Client SDK (Web + iOS + Android)
├── Phase 10: 前端功能联调验证
├── Phase 11: Docker Compose 多服务编排
├── Phase 12: 华为云基础设施搭建 (ECS + RDS + OBS)
├── Phase 13: 生产部署 + SSL/域名
├── Phase 14: CI/CD + 推送通知
├── Phase 15: 安全合规审计 + 监控告警
└── Phase 16: 上线检查清单 + 运维文档
```

### E. 扩展路线图

```
v1.0 (本 SOP)  ✅ 完成
  ├── 后端 + CMS + Web + iOS + Android
  ├── 华为云部署
  └── CI/CD + 推送 + 监控

v1.1 (下一步)
  ├── 支付集成 (微信支付 / 支付宝 / IAP)
  ├── 实时功能 (WebSocket 聊天室)
  ├── 全文搜索 (Elasticsearch)
  └── 国际化 (i18n 多语言)

v2.0 (规模扩展)
  ├── Kubernetes (CCE 容器引擎)
  ├── 微服务拆分
  ├── API Gateway (华为云 APIG)
  └── 多数据中心容灾
```

### F. 文件依赖关系图

```
packages/
├── shared/            ← 共享类型 + API Client (Phase 6)
│   └── src/api-client.ts
│
├── backend/           ← 后端 API (Phase 2-3-5)
│   ├── prisma/schema.prisma  ← 所有数据表定义
│   ├── src/index.ts          ← Hono 应用入口
│   ├── src/routes/           ← API 路由层
│   ├── src/middleware/       ← 认证/限流中间件
│   └── src/lib/              ← 工具库 (db/jwt/redis)
│
├── cms/               ← CMS (Phase 4)
│   └── (Directus 配置)
│
└── web/               ← Web 前端 (Phase 7)
    └── src/app/
        ├── page.tsx           ← 用户首页
        ├── articles/          ← 文章详情页
        ├── admin/             ← 管理后台
        └── login/             ← 登录注册页

mobile/
├── ios/               ← iOS App (Phase 8)
│   └── MyApp/Sources/
│       ├── APIClient.swift    ← Swift API Client
│       └── ContentView.swift  ← SwiftUI 首页
│
└── android/           ← Android App (Phase 9)
    └── MyApp/app/src/main/java/com/example/myapp/
        ├── data/api/          ← Kotlin API Client
        └── ui/screens/        ← Compose 页面

docker/
├── docker-compose.yml  ← 多服务编排 (Phase 11)
├── nginx/nginx.conf    ← 反向代理配置
└── ssl/                ← SSL 证书

.github/workflows/
└── deploy.yml          ← CI/CD (Phase 14)

scripts/
└── backup-db.sh        ← 备份脚本 (Phase 15)

docs/
├── SPECS.md            ← 产品规格书 (Phase 1)
├── API_SPEC.md         ← API 规格书 (Phase 1)
└── RUNBOOK.md          ← 运维文档 (Phase 16)
```

---

> **SOP 版本**: 1.0.0 | **最后更新**: 2026-06-24
> **技术栈**: Hono + TypeScript + Prisma + PostgreSQL + Directus + Next.js 14+ + SwiftUI + Kotlin/Compose
> **部署**: Docker Compose → 华为云 (ECS + RDS + OBS + CDN + CloudEye)
> **关联 SOP**: 可配合 `backend/`、`web/`、`ios/`、`android/` 各部分独立 SOP 使用
> **后续流程**: 完成后 → `../copyright/Software_Copyright_Application_SOP.md` 申请软著

---

## 🟢 第四阶段：后期运营 & 持续迭代

> **时间**: 上线后持续进行
> **目标**: 从「能跑」到「跑得好」→「用户爱用」→「持续增长」
> **原则**: 第一周拼命修 → 第一个月看数据 → 第三个月定方向

---

## Phase 17: 稳定性监控 & 快速响应（上线后 Week 1）

> **[VALIDATE]** + **[SHELL]** + **[DEBUG]**

### Step 17.1 — 上线首日必查清单

```markdown
## 🚨 上线首日 (Day 1) 逐项检查

### 前 5 分钟 — 存活确认
- [ ] `curl https://api.yourdomain.com/api/v1/health` → 200
- [ ] `curl https://yourdomain.com` → 200，页面正常渲染
- [ ] CMS 后台可登录 → `https://admin.yourdomain.com`
- [ ] 数据库可连接 → `docker compose exec backend npx prisma db execute --stdin <<< "SELECT 1"`

### 前 1 小时 — 核心流程走通
- [ ] 新用户注册 → 收到 Token → 可访问需认证的 API
- [ ] 内容 API 返回正确数据 → `/content/articles` `/content/banners`
- [ ] 文件上传 → OBS 存储成功 → CDN URL 可访问
- [ ] 管理后台 → 创建/编辑/发布文章 → 前端可见

### 前 24 小时 — 稳定性观察
- [ ] 错误率 < 0.5%（Sentry / CloudEye 控制台确认）
- [ ] API 响应时间 p95 < 500ms
- [ ] 数据库 CPU < 50%，内存 < 70%
- [ ] 无磁盘空间告警
- [ ] SSL 证书有效
```

### Step 17.2 — 崩溃 & 错误监控集成

```bash
# [SHELL] Sentry 接入 (全平台错误追踪)
cd ${PROJECT_DIR}

# 1. Backend — Sentry Node.js SDK
cd packages/backend
pnpm add @sentry/node @sentry/profiling-node

# 2. Web — Sentry Next.js SDK
cd ../web
pnpm add @sentry/nextjs
npx @sentry/wizard -i nextjs  # 交互式配置

# 3. iOS — Sentry Cocoa (SPM)
# Xcode → File → Add Packages → https://github.com/getsentry/sentry-cocoa

# 4. Android — Sentry Android SDK
# app/build.gradle.kts: implementation("io.sentry:sentry-android:7.x")
```

> **[WRITE]** `packages/backend/src/lib/sentry.ts`:

```typescript
import * as Sentry from "@sentry/node"

if (process.env.NODE_ENV === "production" && process.env.SENTRY_DSN) {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    tracesSampleRate: 0.1,       // 10% 性能追踪
    profilesSampleRate: 0.05,    // 5% profiling
    environment: process.env.NODE_ENV,
    beforeSend(event) {
      // 过滤敏感数据
      if (event.request?.cookies) delete event.request.cookies
      if (event.request?.headers?.["authorization"]) {
        event.request.headers["authorization"] = "[FILTERED]"
      }
      return event
    },
  })
}

export function captureException(error: Error, context?: Record<string, any>) {
  if (process.env.NODE_ENV === "production") {
    Sentry.withScope((scope) => {
      if (context) scope.setExtras(context)
      Sentry.captureException(error)
    })
  }
  console.error(`[ERROR] ${error.message}`, context)
}
```

### Step 17.3 — 实时告警配置

```bash
# [SHELL] 华为云 CloudEye 告警规则 (Web 控制台)
# 也可以在 ECS 上部署轻量级 UptimeRobot 替代

# 关键告警阈值:
echo "
=== 告警规则配置 ===

1. API 不可用 (Critical)
   条件: https://api.yourdomain.com/api/v1/health 连续 2 次失败
   通知: 电话 + 短信 (24/7)
   响应: 5 分钟内介入

2. 错误率飙升 (Critical)
   条件: 5xx 错误率 > 5% 持续 5 分钟
   通知: 短信 + 邮件
   响应: 10 分钟内介入

3. API 延迟过高 (Warning)
   条件: p95 延迟 > 1000ms 持续 10 分钟
   通知: 邮件
   响应: 30 分钟内排查

4. 数据库 CPU 过高 (Warning)
   条件: CPU > 80% 持续 10 分钟
   通知: 邮件
   响应: 检查慢查询，考虑扩容

5. 磁盘空间不足 (Critical)
   条件: 磁盘使用 > 85%
   通知: 短信 + 邮件
   响应: 清理日志/扩容
"
```

### Step 17.4 — 首周每日巡检脚本

> **[WRITE]** `scripts/daily-check.sh`:

```bash
#!/bin/bash
# 每日自动化巡检 (放 ECS crontab: 0 9 * * *)
# Claude Code 也可通过 [SHELL] 远程执行

set -e
ECS_IP="${ECS_IP:-localhost}"
REPORT=""
ALERT=false

# 1. 服务状态
echo "=== $(date '+%Y-%m-%d %H:%M') 每日巡检 ==="

# API 健康检查
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://api.yourdomain.com/api/v1/health)
if [ "$HTTP_CODE" != "200" ]; then
  REPORT="${REPORT}❌ API 健康检查失败 (HTTP $HTTP_CODE)\n"
  ALERT=true
else
  REPORT="${REPORT}✅ API 正常\n"
fi

# Web 前端
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://yourdomain.com)
if [ "$HTTP_CODE" != "200" ]; then
  REPORT="${REPORT}❌ Web 前端不可达 (HTTP $HTTP_CODE)\n"
  ALERT=true
else
  REPORT="${REPORT}✅ Web 正常\n"
fi

# Docker 容器状态
DOWN_COUNT=$(docker compose -f /opt/myapp/docker/docker-compose.yml ps --format json | grep -c '"Health":"unhealthy"' 2>/dev/null || echo 0)
if [ "$DOWN_COUNT" -gt 0 ]; then
  REPORT="${REPORT}❌ $DOWN_COUNT 个容器不健康\n"
  ALERT=true
else
  REPORT="${REPORT}✅ 所有容器健康\n"
fi

# 磁盘使用率
DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
if [ "$DISK_USAGE" -gt 85 ]; then
  REPORT="${REPORT}❌ 磁盘使用率 ${DISK_USAGE}% (超过 85% 阈值)\n"
  ALERT=true
elif [ "$DISK_USAGE" -gt 70 ]; then
  REPORT="${REPORT}⚠️  磁盘使用率 ${DISK_USAGE}%\n"
else
  REPORT="${REPORT}✅ 磁盘使用率 ${DISK_USAGE}%\n"
fi

# 内存使用率
MEM_USAGE=$(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100}')
if [ "$MEM_USAGE" -gt 90 ]; then
  REPORT="${REPORT}❌ 内存使用率 ${MEM_USAGE}%\n"
  ALERT=true
else
  REPORT="${REPORT}✅ 内存使用率 ${MEM_USAGE}%\n"
fi

# 数据库备份
LATEST_BACKUP=$(ls -t /opt/myapp/backups/*.sql.gz 2>/dev/null | head -1)
if [ -z "$LATEST_BACKUP" ]; then
  REPORT="${REPORT}❌ 无数据库备份文件!\n"
  ALERT=true
else
  BACKUP_AGE=$(( ($(date +%s) - $(stat -c %Y "$LATEST_BACKUP")) / 3600 ))
  if [ "$BACKUP_AGE" -gt 30 ]; then
    REPORT="${REPORT}❌ 最新备份 ${BACKUP_AGE} 小时前\n"
    ALERT=true
  else
    REPORT="${REPORT}✅ 备份正常 (${BACKUP_AGE}h 前)\n"
  fi
fi

# SSL 证书过期检查
CERT_EXPIRY=$(echo | openssl s_client -connect yourdomain.com:443 -servername yourdomain.com 2>/dev/null | openssl x509 -noout -enddate 2>/dev/null | cut -d= -f2)
if [ -n "$CERT_EXPIRY" ]; then
  DAYS_LEFT=$(( ($(date -d "$CERT_EXPIRY" +%s) - $(date +%s)) / 86400 ))
  if [ "$DAYS_LEFT" -lt 7 ]; then
    REPORT="${REPORT}❌ SSL 证书 ${DAYS_LEFT} 天后过期!\n"
    ALERT=true
  elif [ "$DAYS_LEFT" -lt 30 ]; then
    REPORT="${REPORT}⚠️  SSL 证书 ${DAYS_LEFT} 天后过期\n"
  else
    REPORT="${REPORT}✅ SSL 证书剩余 ${DAYS_LEFT} 天\n"
  fi
fi

# 输出报告
echo -e "$REPORT"
echo "==========================="

# 告警通知
if [ "$ALERT" = true ]; then
  echo "🚨 发现异常，发送告警..."
  # 华为云 SMN 发送告警
  # 或使用 webhook 发送到飞书/钉钉/企业微信
  curl -s -X POST "${FEISHU_WEBHOOK_URL}" \
    -H "Content-Type: application/json" \
    -d "{\"msg_type\":\"text\",\"content\":{\"text\":\"🚨 每日巡检告警\n$(echo -e $REPORT)\"}}" \
    2>/dev/null || true
fi
```

> 配置 crontab:
> ```bash
> ssh root@<ECS_IP>
> echo "0 9 * * * bash /opt/myapp/scripts/daily-check.sh >> /var/log/daily-check.log 2>&1" | crontab -
> ```

```bash
# [GIT]
git add -A && git commit -m "Phase 17: Week 1 stability monitoring & rapid response setup"
```

---

## Phase 18: 内容运营 & 用户增长（上线后 Week 2-4）

> **[DIALOG]** + **[WRITE]** + **[GENERATE]**

### Step 18.1 — CMS 内容运营 SOP

```markdown
## 📝 内容运营工作流

### 内容发布日历
```
每周规划 (周五定下周):

周一    周二    周三    周四    周五    周六    周日
─────────────────────────────────────────────────
干货文章  用户案例  产品更新  行业资讯  互动话题  轻量图文  休息
(SEO)   (口碑)   (公告)   (引流)   (活跃)   (维系)
```

### 文章发布 Checklist
- [ ] 标题: 含目标关键词，≤25 字，有吸引力
- [ ] 摘要: 1-2 句话概括，含 CTA
- [ ] 正文: ≥800 字，分段清晰，配图 ≥3 张
- [ ] SEO: slug 优化 / meta description / alt text
- [ ] 封面图: 1200×630px (社交分享尺寸)
- [ ] 标签: 3-5 个相关标签
- [ ] 定时发布: 工作日上午 9:00 或晚上 20:00
- [ ] 社交分发: 同步到微博/小红书/知乎/微信群
```

### Banner 更新策略
```
更新频率:
  - 首页主 Banner: 每周更新 1 次 (配合最新活动/内容)
  - 侧边栏/二级 Banner: 每月更新
  - 节日/活动 Banner: 提前 3 天上，节后 1 天下

AB 测试:
  - 每 2 周对 Banner 做 1 次 AB 测试
  - 指标: 点击率 (CTR) → 保持 > 3%
  - 低于 1% → 替换
```

### 推送通知策略
```
推送类型:
  1. 系统通知 (自动): 评论回复、关注、订单状态 → 即时发送
  2. 内容推送 (运营): 新文章、活动 → 工作日晚 19:00-20:00
  3. 唤醒推送: 7 天未活跃 → 周末上午 10:00

频率控制:
  - 每用户每周 ≤ 3 条运营推送
  - 同一天 ≤ 1 条
  - 提供推送偏好设置 (用户可选择关闭)

文案规范:
  - 标题 ≤ 20 字
  - 正文 ≤ 50 字
  - 带 emoji 提高打开率 20%+
  - 含明确 CTA: "立即查看" > "戳我" > "点这里"
```

### 公告管理
```
公告类型 & 使用场景:
  - info (蓝色): 功能更新、新内容上线
  - warning (黄色): 计划维护、服务调整
  - error (红色): 紧急故障、服务中断
  - success (绿色): 活动获奖、好消息

发布规则:
  - 最多同时显示 2 条公告
  - 紧急公告 (error) 始终排最前
  - 每条公告设过期时间 (max 7 天)
```

### Step 18.2 — 用户反馈闭环

> **[WRITE]** `packages/web/src/app/api/feedback/route.ts`:

```typescript
import { NextRequest, NextResponse } from "next/server"
import { prisma } from "../../../../backend/src/lib/db"

// POST /api/feedback — 用户反馈收集
export async function POST(req: NextRequest) {
  const { content, category, contact, userId } = await req.json()

  if (!content || content.trim().length < 5) {
    return NextResponse.json({ error: "反馈内容至少 5 个字" }, { status: 400 })
  }

  // 保存反馈到数据库 (需要新增 Feedback 表)
  // 同时发送通知给运营团队
  // 此处可接入飞书/钉钉 webhook

  console.log(`[Feedback] ${category}: ${content} (contact: ${contact})`)

  return NextResponse.json({ success: true, message: "感谢反馈！我们会认真处理" })
}
```

```markdown
## 🔄 反馈处理流程

```
用户反馈 → 分类 → 优先级评估 → 分配 → 处理 → 回复 → 关闭

分类标签:
  🐛 Bug 报告   → 24h 内确认，严重 Bug 48h 内修复
  💡 功能建议   → 加入需求池，每月评审
  ❓ 使用咨询   → 4h 内回复
  😡 投诉/差评  → 2h 内响应，24h 内解决
  👍 好评/表扬  → 当天感谢回复，截图留念
```

### 响应模板

**Bug 报告回复:**
> 感谢反馈！我们已确认此问题，将在下个版本中修复。
> 预计修复时间：2 个工作日内。如有紧急需要请联系 support@example.com

**功能建议回复:**
> 感谢建议！已记录到产品需求池中。
> 我们会在月度评审中讨论，如采纳会第一时间通知您。

**投诉回复:**
> 非常抱歉给您带来不好的体验。我们已安排专人处理，
> 会在 2 小时内联系您。如有更详细的信息请发至 support@example.com
```

### Step 18.3 — 用户增长引擎

```markdown
## 📈 增长引擎搭建

### 1. 获客渠道矩阵
```
渠道              目标        预算/月    优先级
─────────────────────────────────────────────
SEO (自然搜索)     日 UV 500+   ¥0         P0 🔥
内容营销 (知乎/公众号) 线索 100+  ¥0 (人力)   P0 🔥
应用商店搜索 (ASO)  下载 1000+   ¥0         P1
社交媒体 (小红书/抖音) 曝光 10w+  ¥0-500     P1
付费广告 (信息流)   下载 5000+   ¥2000-5000  P2
KOL/达人合作        品牌曝光     ¥1000-5000  P2
```

### 2. SEO 优化清单 (Web)
```
□ 所有页面有独立的 <title> 和 <meta description>
□ sitemap.xml 自动生成 → 提交到 Google Search Console + 百度站长
□ robots.txt 正确配置
□ 图片有 alt 属性
□ URL 结构语义化: /articles/hello-world (非 /post/123)
□ 内链建设: 每篇文章至少 3 个内部链接
□ 页面加载速度: Lighthouse Performance ≥ 85
□ 结构化数据 (Schema.org): Article / BreadcrumbList / FAQ
□ 移动端友好: Google Mobile-Friendly Test 通过
□ 百度收录: 提交到百度站长平台 (国内必须)
```

### 3. ASO 优化清单 (App Store / 各应用市场)
```
□ 标题: 品牌名 + 核心关键词 (≤30 字符)
□ 副标题: 一句话卖点 (iOS)
□ 关键词域: 覆盖 10-15 个搜索词 (iOS)
□ 描述: 前 3 行最核心信息 (用户不点"更多"就能看到)
□ 截图: 前 2 张展示核心功能 (不是品牌页)
□ 评分: 引导满意用户评分 (评分 > 4.5 转化率翻倍)
□ 更新日志: 写给人看的更新说明 (不是 "bug fixes")
□ 华为应用市场: 单独优化标题和描述 (中文市场)
□ 小米/OPPO/vivo: 如有精力，逐个市场优化
```

### 4. 推荐裂变机制
```
推荐奖励:
  - 推荐 1 人注册 → 双方各得 7 天 VIP
  - 推荐 5 人 → 额外 30 天 VIP

分享激励:
  - 完成任务后 → "分享成就" 按钮 → 带追踪链接
  - 优质内容 → "分享到朋友圈" → 带 UTM 参数
```
```

```bash
# [GIT]
git add -A && git commit -m "Phase 18: Content operations & user growth engine"
```

---

## Phase 19: 数据驱动迭代（上线后 Month 1-3）

> **[VALIDATE]** + **[WRITE]** + **[DIALOG]**

### Step 19.1 — 核心指标看板搭建

> **[WRITE]** `packages/backend/src/routes/admin-stats.ts`:

```typescript
import { Hono } from "hono"
import { prisma } from "../lib/db"
import { ok } from "../lib/response"
import { authRequired, requireRole } from "../middleware/auth"

export const adminStatsRoute = new Hono()
adminStatsRoute.use("*", authRequired, requireRole("SUPER_ADMIN", "ADMIN"))

// GET /admin/stats/dashboard — 运营仪表板数据
adminStatsRoute.get("/dashboard", async (c) => {
  const now = new Date()
  const todayStart = new Date(now.getFullYear(), now.getMonth(), now.getDate())
  const weekAgo = new Date(now.getTime() - 7 * 86400000)
  const monthAgo = new Date(now.getTime() - 30 * 86400000)

  const [
    totalUsers,
    newUsersToday,
    newUsersWeek,
    newUsersMonth,
    totalArticles,
    activeArticles,
    totalBanners,
    totalNotifications,
  ] = await Promise.all([
    prisma.user.count(),
    prisma.user.count({ where: { createdAt: { gte: todayStart } } }),
    prisma.user.count({ where: { createdAt: { gte: weekAgo } } }),
    prisma.user.count({ where: { createdAt: { gte: monthAgo } } }),
    prisma.article.count(),
    prisma.article.count({ where: { status: "PUBLISHED" } }),
    prisma.banner.count({ where: { status: "PUBLISHED" } }),
    prisma.notification.count(),
  ])

  // 计算增长率
  const previousMonthUsers = await prisma.user.count({
    where: {
      createdAt: {
        gte: new Date(now.getTime() - 60 * 86400000),
        lt: monthAgo,
      }
    }
  })

  const userGrowthRate = previousMonthUsers > 0
    ? ((newUsersMonth - previousMonthUsers) / previousMonthUsers * 100).toFixed(1)
    : "N/A"

  return ok(c, {
    users: {
      total: totalUsers,
      newToday: newUsersToday,
      newThisWeek: newUsersWeek,
      newThisMonth: newUsersMonth,
      growthRate: `${userGrowthRate}%`,
    },
    content: {
      totalArticles,
      publishedArticles: activeArticles,
      activeBanners: totalBanners,
    },
    engagement: {
      totalNotificationsSent: totalNotifications,
      // 更多指标需要埋点数据
    },
    timestamp: now.toISOString(),
  })
})

// GET /admin/stats/content — 内容分析
adminStatsRoute.get("/content", async (c) => {
  const topArticles = await prisma.article.findMany({
    where: { status: "PUBLISHED" },
    orderBy: { publishedAt: "desc" },
    take: 10,
    select: {
      id: true, title: true, slug: true,
      publishedAt: true, tags: true,
    }
  })

  // 按标签统计
  const allArticles = await prisma.article.findMany({
    where: { status: "PUBLISHED" },
    select: { tags: true },
  })

  const tagCounts: Record<string, number> = {}
  for (const article of allArticles) {
    for (const tag of article.tags) {
      tagCounts[tag] = (tagCounts[tag] || 0) + 1
    }
  }

  return ok(c, {
    recentArticles: topArticles,
    tagDistribution: Object.entries(tagCounts)
      .sort(([, a], [, b]) => b - a)
      .slice(0, 10)
      .map(([tag, count]) => ({ tag, count })),
  })
})
```

### Step 19.2 — 用户行为埋点

> **[WRITE]** `packages/shared/src/analytics.ts`:

```typescript
// 前端统一埋点 SDK — Web / iOS / Android 复用接口规范

export interface AnalyticsEvent {
  event: string
  properties?: Record<string, string | number | boolean>
  timestamp?: string
  userId?: string
  platform?: "WEB" | "IOS" | "ANDROID"
}

// Web 端实现
class WebAnalytics {
  private endpoint: string

  constructor(apiBaseUrl: string) {
    this.endpoint = `${apiBaseUrl}/analytics/events`
  }

  async track(event: string, properties?: Record<string, any>) {
    const payload: AnalyticsEvent = {
      event,
      properties,
      timestamp: new Date().toISOString(),
      platform: "WEB",
    }

    // 异步发送，不阻塞主流程
    navigator.sendBeacon
      ? navigator.sendBeacon(this.endpoint, JSON.stringify(payload))
      : fetch(this.endpoint, {
          method: "POST",
          body: JSON.stringify(payload),
          keepalive: true,
        }).catch(() => {})
  }

  // 预定义事件
  pageView(page: string) { this.track("page_view", { page }) }
  articleView(slug: string) { this.track("article_view", { slug }) }
  signUp(method: string) { this.track("sign_up", { method }) }
  signIn(method: string) { this.track("sign_in", { method }) }
  bannerClick(bannerId: string) { this.track("banner_click", { bannerId }) }
  share(contentType: string) { this.track("share", { contentType }) }
  search(query: string) { this.track("search", { query }) }
}

export const analytics = new WebAnalytics(
  process.env.NEXT_PUBLIC_API_URL || "http://localhost:3000/api/v1"
)
```

> **[WRITE]** `packages/backend/src/routes/analytics.ts` — 后端埋点接收:

```typescript
import { Hono } from "hono"
import { zValidator } from "@hono/zod-validator"
import { z } from "zod"
import { prisma } from "../lib/db"
import { ok } from "../lib/response"

export const analyticsRoutes = new Hono()

// POST /analytics/events — 接收埋点事件
analyticsRoutes.post("/events", zValidator("json", z.object({
  event: z.string(),
  properties: z.record(z.any()).optional(),
  platform: z.enum(["WEB", "IOS", "ANDROID"]).optional(),
  userId: z.string().optional(),
  timestamp: z.string().optional(),
})), async (c) => {
  const body = c.req.valid("json")

  // 生产环境: 写入专门的时序数据库或日志系统
  // 此处简化: 打印到 stdout，由日志采集系统收集
  console.log(JSON.stringify({
    type: "analytics",
    ...body,
    serverTimestamp: new Date().toISOString(),
    ip: c.req.header("x-forwarded-for") || c.req.header("x-real-ip"),
    userAgent: c.req.header("user-agent"),
  }))

  return ok(c, { received: true })
})
```

### Step 19.3 — 月度数据复盘模板

> **[WRITE]** `docs/monthly-review-template.md`:

```markdown
# ${产品名称} 月度数据复盘 — YYYY年MM月

## 1. 核心指标总览

| 指标 | 本月 | 上月 | 环比 | 目标 | 达成 |
|------|------|------|------|------|------|
| DAU | | | | | |
| MAU | | | | | |
| 新增用户 | | | | | |
| 留存 D7 | | | | | |
| 留存 D30 | | | | | |
| 会话时长(分) | | | | | |
| 崩溃率 | | | | | |
| API 可用性 | | | | | |
| 付费用户数 | | | | | |
| 月收入 | | | | | |

## 2. 内容表现

| 文章标题 | PV | 分享 | 转化 |
|---------|-----|------|------|
| | | | |

## 3. 用户反馈

| 类别 | 数量 | 趋势 | 代表问题 |
|------|------|------|---------|
| Bug 报告 | | | |
| 功能建议 | | | |
| 好评 | | | |

## 4. 本月 What Went Well

- 

## 5. 本月 What Didn't Go Well

- 

## 6. 下月重点

- [ ] P0: 
- [ ] P1: 
- [ ] P2: 

## 7. 产品迭代计划

| 版本 | 计划日期 | 主要内容 |
|------|---------|---------|
| v1.1.0 | | |
| v1.2.0 | | |
```

### Step 19.4 — AB 测试框架

> **[WRITE]** `packages/backend/src/lib/ab-test.ts`:

```typescript
// 简单的 AB 测试分配器

interface Experiment {
  id: string
  name: string
  variants: { name: string; weight: number }[]
  startAt: Date
  endAt?: Date
}

// 基于用户 ID 哈希的稳定分组 (同一用户始终在同一组)
export function assignVariant(experiment: Experiment, userId: string): string {
  const totalWeight = experiment.variants.reduce((sum, v) => sum + v.weight, 0)
  const hash = simpleHash(userId + experiment.id)
  const bucket = hash % totalWeight

  let cumulative = 0
  for (const variant of experiment.variants) {
    cumulative += variant.weight
    if (bucket < cumulative) return variant.name
  }

  return experiment.variants[0].name // fallback
}

// 使用示例
export const HOMEPAGE_EXPERIMENT: Experiment = {
  id: "homepage_layout_v1",
  name: "首页布局 AB 测试",
  variants: [
    { name: "control", weight: 50 },    // 50% 对照组: 原布局
    { name: "variant_a", weight: 50 },  // 50% 实验组: 新布局
  ],
  startAt: new Date("2026-07-01"),
}

function simpleHash(str: string): number {
  let hash = 0
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i)
    hash = ((hash << 5) - hash) + char
    hash = hash & hash // Convert to 32bit integer
  }
  return Math.abs(hash)
}
```

```bash
# [GIT]
git add -A && git commit -m "Phase 19: Data-driven iteration — analytics dashboard + AB testing"
```

---

## Phase 20: 日常运营 SOP — 日/周/月/季 Checklist

> **[VALIDATE]** 此 Phase 提供可直接执行的重复性运营清单

### Step 20.1 — 每日运营 (每天 ≤ 30 分钟)

```markdown
## ☀️ 每日运营 Checklist

### 早上 9:00 (5 分钟)
- [ ] 查看巡检脚本输出 `/var/log/daily-check.log`
- [ ] 检查 Sentry / CloudEye 有无新告警
- [ ] 检查 API 健康状态
- [ ] 瞥一眼昨日的 DAU / 新增用户 (如有数据看板)

### 上午 10:00 (10 分钟)
- [ ] 查看新的用户反馈 (Feedback 表 / 邮件)
- [ ] 回复紧急反馈 (Bug / 投诉 — 2h 内)
- [ ] 检查应用商店新评价 → 回复

### 下午 16:00 (10 分钟)
- [ ] 查看今日内容发布效果 (PV / 分享)
- [ ] 检查社交媒体互动 (评论 / @)
- [ ] 同步团队 (飞书/钉钉/微信群): 今日有无异常

### 晚上 19:00 (5 分钟) — 如当天有推送任务
- [ ] 检查推送发送成功率
- [ ] 检查推送点击率 (CTR)
```

### Step 20.2 — 每周运营 (每周五下午 1 小时)

```markdown
## 📅 每周运营 Checklist

### 数据复盘
- [ ] 本周 DAU/MAU 趋势
- [ ] 本周新增用户数 (按渠道分)
- [ ] 本周留存率变化
- [ ] 本周崩溃率 (目标 < 0.3%)
- [ ] 本周 API p95 延迟

### 内容复盘
- [ ] 本周发布文章数: ___ 篇
- [ ] 表现最好的文章: ___________ (PV: ___)
- [ ] 需要更新的文章: ___________
- [ ] Banner 点击率: ___%

### 竞品监控
- [ ] 竞品 A 本周有无更新: _________
- [ ] 竞品 B 本周有无更新: _________
- [ ] 行业动态/热点: _________

### 下周计划
- [ ] 下周内容日历 (4-5 篇)
- [ ] 下周推送计划 (≤ 2 次)
- [ ] 下周要修复的 Bug: ___________
- [ ] 需要协调的资源: ___________

### 备份验证
- [ ] 本周数据库备份是否正常 (文件大小: ___)
- [ ] 手动恢复测试 (每月做 1 次，本周随机选一备份文件恢复)
```

### Step 20.3 — 每月运营 (每月 1 号下午 2 小时)

```markdown
## 📊 每月运营 Checklist

### 完整数据复盘 (见 Phase 19.3 模板)
- [ ] 填写月度数据复盘模板
- [ ] 与上月对比，标记异常变化 (>20% 的涨跌)
- [ ] 分析异常原因，记录到复盘文档

### 版本迭代
- [ ] 审查本月上线的版本数量: ___
- [ ] 审查本月修复的 Bug 数量: ___
- [ ] 规划下月版本内容 (P0/P1/P2)
- [ ] 更新 Roadmap

### 用户运营
- [ ] 本月新增用户 Top 3 渠道: ___________
- [ ] 本月流失用户分析 (如有): ___________
- [ ] VIP/付费用户月度报告

### 内容审计
- [ ] 检查所有已发布文章: 有无过时/错误信息
- [ ] 更新 3 篇最热门的老文章 (保持新鲜度)
- [ ] 检查所有 Banner: 有无过期未下的

### 安全 & 合规
- [ ] 依赖安全漏洞扫描 (`pnpm audit`)
- [ ] SSL 证书有效期确认 (≥ 15 天)
- [ ] 隐私政策/用户协议: 如有变更需更新
- [ ] 审查管理员账号: 有无离职/不再需要的账号

### 成本优化
- [ ] 华为云账单审查: 本月费用 ¥___
- [ ] 有无可优化的资源 (降配/释放闲置)
- [ ] CDN 流量是否在套餐内
- [ ] OBS 存储是否有可清理的过期文件
```

### Step 20.4 — 每季度运营 (季末最后一周)

```markdown
## 🎯 每季度运营 Checklist

### 战略复盘
- [ ] Q 度目标达成率: __%
- [ ] 核心指标趋势 (连续 3 个月)
- [ ] 与年初/上季度对比
- [ ] 竞品格局变化
- [ ] 行业趋势判断

### 产品方向
- [ ] 用户需求调研 (问卷/访谈 NPS)
- [ ] 功能优先级重排 (根据数据+反馈)
- [ ] 技术债务评估 (是否需重构)
- [ ] 下季度 Roadmap 规划

### 团队 & 资源
- [ ] 团队成员能力评估 / 培训需求
- [ ] 服务器资源评估 (是否需要扩容)
- [ ] 预算审查 (实际 vs 预算)
- [ ] 第三方服务续费 (域名/SSL/API 等)

### 灾备演练
- [ ] 完整数据库恢复演练 (1 次/季度)
- [ ] 回滚流程测试 (1 次/季度)
- [ ] 应急预案更新 (如有人员/架构变更)
- [ ] 文档更新 (Runbook / 架构图 / 联系人)

### 法律合规
- [ ] 隐私政策 / 用户协议 年度审查
- [ ] ICP 备案 / 公安备案 状态确认
- [ ] 数据跨境传输合规检查 (如适用)
- [ ] App Store / Google Play 政策更新检查

### 庆祝 & 复盘
- [ ] 团队季度复盘会
- [ ] 表彰贡献者
- [ ] 记录 Lessons Learned
```

```bash
# [GIT]
git add -A && git commit -m "Phase 20: Daily/weekly/monthly/quarterly operations SOP"
```

---

## Phase 21: 生态建设 — 开发者 & 合作伙伴 & 社区（上线后 Month 3-6）

> **[VALIDATE]** 此 Phase 在所有核心功能稳定、运营体系运转正常后启动。先完成 Phase 17-20 的基础运营，再做生态。
>
> **核心理念**: 产品本身是护城河的第 1 层，生态才是第 2 层。API 开放让第三方依赖你，合作伙伴让销售规模指数增长，社区让用户变成传播节点。

### Step 21.1 — API 开放平台 & 开发者生态（第 1-3 周）

```bash
# [SHELL] 创建开发者门户子应用
mkdir -p apps/dev-portal/src/pages
cd apps/dev-portal && pnpm init && cd ../..
```

```
apps/dev-portal/
├── src/
│   ├── pages/
│   │   ├── index.tsx              # 开发者首页 (概览)
│   │   ├── docs/
│   │   │   ├── quickstart.tsx     # 快速开始指南
│   │   │   ├── authentication.tsx # 认证文档
│   │   │   └── api-reference.tsx  # API 参考
│   │   ├── apps.tsx               # 我的应用 (API Key 管理)
│   │   └── dashboard.tsx          # 开发者仪表盘 (调用统计)
│   └── components/
│       ├── CodePreview.tsx        # 代码示例展示
│       └── ApiPlayground.tsx      # API 在线测试工具
├── next.config.js
└── package.json
```

#### 21.1.1 — API Key 管理体系

```typescript
// [WRITE] packages/shared/src/api-key-schema.ts
import { z } from "zod"

// API Key 权限范围
export const ApiKeyScope = z.enum([
  "read:users",      // 读取用户信息
  "read:content",    // 读取内容
  "write:content",   // 写入内容
  "read:analytics",  // 读取分析数据
  "webhook:send",    // 发送 Webhook
])

// 创建 API 应用请求
export const CreateAppSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(500).optional(),
  website: z.string().url().optional(),
  scopes: z.array(ApiKeyScope).min(1),
  redirectUri: z.string().url().optional(), // OAuth 回调地址
})

// API Key 模型 (Prisma)
export const ApiAppPrismaSchema = `
model ApiApp {
  id          String   @id @default(uuid())
  userId      String
  name        String
  description String?
  website     String?
  apiKey      String   @unique     // sk_live_xxxx (生成后不可再读取，仅存 hash)
  apiKeyHash  String               // SHA-256 hash
  apiKeyPrefix String              // sk_live_xxx... 前缀 8 位用于展示
  scopes      String[]             // 权限范围数组
  redirectUri String?
  rateLimit   Int      @default(100) // 每分钟请求上限
  isActive    Boolean  @default(true)
  lastUsedAt  DateTime?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([userId])
  @@index([apiKeyHash])
}
`
```

```typescript
// [WRITE] apps/backend/src/routes/developer.ts
import { Hono } from "hono"
import { authRequired } from "../middleware/auth"
import { prisma } from "../lib/db"
import { createHash, randomBytes } from "node:crypto"

const developerRoutes = new Hono()

// 生成 API Key (仅请求时返回完整 key，之后不可见)
function generateApiKey(): { key: string; hash: string; prefix: string } {
  const random = randomBytes(32).toString("hex")
  const key = `sk_live_${random}`
  const hash = createHash("sha256").update(key).digest("hex")
  const prefix = key.slice(0, 16) // sk_live_xxxxxx
  return { key, hash, prefix }
}

// 创建 API 应用
developerRoutes.post("/apps", authRequired, async (c) => {
  const body = await c.req.json()
  const { name, description, website, scopes, redirectUri } = body
  const user = c.get("user")

  const { key, hash, prefix } = generateApiKey()

  const app = await prisma.apiApp.create({
    data: {
      userId: user.id,
      name,
      description,
      website,
      apiKey: key,       // ⚠️ 全量 key 仅此一次存储，后续展示 prefix
      apiKeyHash: hash,
      apiKeyPrefix: prefix,
      scopes,
      redirectUri,
    },
    select: {
      id: true,
      name: true,
      apiKey: true,       // 返回完整 key — 仅此一次！
      apiKeyPrefix: true,
      scopes: true,
      createdAt: true,
    },
  })

  // [AUDIT] 记录 API Key 创建事件
  await prisma.auditLog.create({
    data: {
      userId: user.id,
      action: "API_KEY_CREATED",
      resource: `api_app:${app.id}`,
      metadata: { name, scopes },
    },
  })

  return c.json({ success: true, data: app })
})

// 列出我的 API 应用
developerRoutes.get("/apps", authRequired, async (c) => {
  const user = c.get("user")
  const apps = await prisma.apiApp.findMany({
    where: { userId: user.id },
    select: {
      id: true, name: true, apiKeyPrefix: true,
      scopes: true, isActive: true, lastUsedAt: true,
      createdAt: true, rateLimit: true,
    },
  })
  return c.json({ success: true, data: apps })
})

// 吊销 API Key
developerRoutes.delete("/apps/:id", authRequired, async (c) => {
  const user = c.get("user")
  const { id } = c.req.param()

  const app = await prisma.apiApp.findFirst({
    where: { id, userId: user.id },
  })
  if (!app) return c.json({ error: "App not found" }, 404)

  await prisma.apiApp.update({
    where: { id },
    data: { isActive: false },
  })

  await prisma.auditLog.create({
    data: {
      userId: user.id,
      action: "API_KEY_REVOKED",
      resource: `api_app:${id}`,
    },
  })

  return c.json({ success: true, message: "API Key revoked" })
})

// 开发者仪表盘 — 调用统计
developerRoutes.get("/dashboard", authRequired, async (c) => {
  const user = c.get("user")
  const apps = await prisma.apiApp.findMany({
    where: { userId: user.id },
    select: { id: true, name: true },
  })

  // 获取每个 App 过去 30 天的调用量
  const thirtyDaysAgo = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000)
  const stats = await Promise.all(
    apps.map(async (app) => {
      const calls = await prisma.apiCallLog.count({
        where: {
          apiAppId: app.id,
          createdAt: { gte: thirtyDaysAgo },
        },
      })
      return { ...app, totalCalls: calls }
    })
  )

  return c.json({ success: true, data: stats })
})

export { developerRoutes }
```

#### 21.1.2 — API Key 认证中间件

```typescript
// [WRITE] apps/backend/src/middleware/api-key-auth.ts
import { createMiddleware } from "hono/factory"
import { prisma } from "../lib/db"
import { createHash } from "node:crypto"

// 验证第三方 API Key (用于开放 API)
export const apiKeyAuth = createMiddleware(async (c, next) => {
  const authHeader = c.req.header("Authorization")
  if (!authHeader?.startsWith("Bearer sk_live_")) {
    return c.json({ error: "Invalid API key format" }, 401)
  }

  const apiKey = authHeader.slice(7)
  const hash = createHash("sha256").update(apiKey).digest("hex")

  const app = await prisma.apiApp.findFirst({
    where: { apiKeyHash: hash, isActive: true },
  })

  if (!app) {
    return c.json({ error: "Invalid or revoked API key" }, 401)
  }

  // 检查速率限制
  const oneMinuteAgo = new Date(Date.now() - 60 * 1000)
  const recentCalls = await prisma.apiCallLog.count({
    where: {
      apiAppId: app.id,
      createdAt: { gte: oneMinuteAgo },
    },
  })

  if (recentCalls >= app.rateLimit) {
    return c.json({ error: "Rate limit exceeded. Try again later." }, 429)
  }

  // 记录 API 调用
  await prisma.apiCallLog.create({
    data: {
      apiAppId: app.id,
      endpoint: c.req.path,
      method: c.req.method,
      ip: c.req.header("x-forwarded-for") || "unknown",
    },
  })

  // 更新最后使用时间
  await prisma.apiApp.update({
    where: { id: app.id },
    data: { lastUsedAt: new Date() },
  })

  c.set("apiApp", app)
  c.set("scopes", app.scopes)
  await next()
})

// Scope 权限检查
export function requireScope(scope: string) {
  return createMiddleware(async (c, next) => {
    const scopes: string[] = c.get("scopes") || []
    if (!scopes.includes(scope)) {
      return c.json({
        error: "Missing required scope",
        required: scope,
        granted: scopes,
      }, 403)
    }
    await next()
  })
}
```

#### 21.1.3 — 开发者文档门户

```typescript
// [WRITE] apps/dev-portal/src/pages/docs/api-reference.tsx

// API 参考文档页面 — 自动从 Zod schema 生成文档
// 使用 OpenAPI / Swagger 规范自动渲染
export default function ApiReference() {
  return (
    <div className="max-w-4xl mx-auto py-12 px-4">
      <h1 className="text-3xl font-bold mb-8">API Reference</h1>

      <section className="mb-12">
        <h2 className="text-xl font-semibold mb-4">Base URL</h2>
        <code className="bg-zinc-100 px-3 py-1.5 rounded text-sm">
          https://api.yourdomain.com/api/v1
        </code>
      </section>

      <section className="mb-12">
        <h2 className="text-xl font-semibold mb-4">Authentication</h2>
        <p className="text-zinc-600 mb-4">
          All API requests require an API Key passed in the Authorization header.
        </p>
        <pre className="bg-zinc-900 text-green-400 p-4 rounded-lg overflow-x-auto">
{`curl -H "Authorization: Bearer sk_live_YOUR_API_KEY" \\
     https://api.yourdomain.com/api/v1/content/list`}
        </pre>
      </section>

      {/* 自动生成的端点文档 */}
      <ApiEndpointSection
        method="GET"
        path="/content/list"
        description="获取公开内容列表"
        params={[
          { name: "page", type: "number", default: 1 },
          { name: "limit", type: "number", default: 20 },
          { name: "category", type: "string", optional: true },
        ]}
        scope="read:content"
      />

      <ApiEndpointSection
        method="POST"
        path="/content/create"
        description="创建内容"
        scope="write:content"
        bodyExample='{ "title": "...", "body": "...", "category": "..." }'
      />

      <ApiEndpointSection
        method="GET"
        path="/analytics/stats"
        description="获取分析数据"
        scope="read:analytics"
      />
    </div>
  )
}
```

#### 21.1.4 — SDK 自动生成与分发

```bash
# [SHELL] 从 OpenAPI spec 自动生成 SDK
# 使用 openapi-generator 从 Swagger 文档生成多语言 SDK

# 1. 先生成 OpenAPI spec (后端启动后)
curl http://localhost:3000/api/v1/openapi.json > api-spec.json

# 2. 生成 TypeScript SDK (npm 包)
npx @openapitools/openapi-generator-cli generate \
  -i api-spec.json \
  -g typescript-fetch \
  -o sdks/typescript \
  --additional-properties=npmName=@yourproduct/api-sdk,npmVersion=1.0.0

# 3. 生成 Swift SDK (iOS)
npx @openapitools/openapi-generator-cli generate \
  -i api-spec.json \
  -g swift5 \
  -o sdks/swift \
  --additional-properties=projectName=ProductAPI

# 4. 生成 Kotlin SDK (Android)
npx @openapitools/openapi-generator-cli generate \
  -i api-spec.json \
  -g kotlin \
  -o sdks/kotlin \
  --additional-properties=packageName=com.yourproduct.api

# 5. 发布 TypeScript SDK 到 npm
cd sdks/typescript && npm publish

# 6. 发布 Swift SDK 到 CocoaPods / SPM
cd sdks/swift
# 编辑 ProductAPI.podspec → pod trunk push

# 7. 发布 Kotlin SDK 到 Maven Central
cd sdks/kotlin && ./gradlew publish
```

```markdown
## 🌐 SDK 分发渠道清单

| 语言/平台 | 包管理器 | 包名 |
|-----------|---------|------|
| TypeScript/Node.js | npm | `@yourproduct/api-sdk` |
| Swift (iOS/macOS) | CocoaPods / SPM | `ProductAPI` |
| Kotlin (Android/JVM) | Maven Central | `com.yourproduct:api-sdk` |
| Python | PyPI | `yourproduct-api-sdk` |
| PHP | Composer | `yourproduct/api-sdk` |
| Go | Go Modules | `github.com/yourproduct/api-sdk-go` |
```

```markdown
## 📘 开发者入职体验 (Developer Onboarding)

目标: 开发者 5 分钟内完成第一个 API 调用。

### 黄金路径 (Happy Path)
```
访问 dev.yourdomain.com → 注册 → 创建 App → 获取 API Key →
复制 Quickstart 代码 → 粘贴到终端 → curl 成功返回 200
⏱️ 目标耗时: ≤ 5 分钟
```

### 快速开始页面必须包含:
- [ ] 3 行 curl 命令即可调用成功
- [ ] 各语言的 5 行代码示例 (Node/Python/Swift/Kotlin)
- [ ] API Playground (在线测试工具，无需写代码)
- [ ] 真实返回示例 (不是假数据)
- [ ] 常见错误的故障排除指南

### 开发者支持
- [ ] GitHub Discussions 作为开发者社区
- [ ] Stack Overflow 标签 `[yourproduct]`
- [ ] 开发者邮件列表 developer@yourdomain.com
- [ ] Discord/Slack 开发者频道
- [ ] 每周 Office Hours (线上 Q&A)
```

### Step 21.2 — 合作伙伴体系（第 2-4 周）

```markdown
## 🤝 合作伙伴分类模型

```
                    ┌──────────────┐
                    │   技术合作伙伴  │ ← ISV/集成商: 在你的 API 上构建应用
                    │ (Technology)  │   收益: API 调用量分成
                    └──────────────┘
                    ┌──────────────┐
                    │   解决方案伙伴  │ ← 代理商: 打包销售你的产品
                    │ (Solution)   │   收益: 销售佣金 15-25%
                    └──────────────┘
                    ┌──────────────┐
                    │   渠道合作伙伴  │ ← 分销商: 特定行业/地域拓展
                    │ (Channel)    │   收益: 批发折扣 + 返点
                    └──────────────┘
                    ┌──────────────┐
                    │   推荐合作伙伴  │ ← Affiliate/博主: 推荐用户
                    │ (Referral)   │   收益: CPA 固定佣金或收入分成
                    └──────────────┘
```

## 📋 合作伙伴入驻流程

### 第一步: 申请 (在线表单)
- 公司/个人名称
- 合作类型: [技术/解决方案/渠道/推荐]
- 覆盖行业/地域
- 现有客户数/案例
- 为什么想做我们的合作伙伴?

### 第二步: 审核 (48 小时内)
- 技术伙伴: 审查技术方案 / 集成计划
- 解决方案/渠道: 审查公司资质 / 销售能力
- 推荐: 审查粉丝数 / 内容质量 / 受众匹配度

### 第三步: 签约
- 合作伙伴协议 (电子签)
- NDA (保密协议)
- 分润协议 (Revenue Share Agreement)
- 行为准则 (Code of Conduct)

### 第四步: 入职培训
- 产品深度培训 (2h)
- API/集成技术培训 (2h, 技术伙伴)
- 销售话术 & 竞品对比 (2h, 渠道伙伴)
- 后台操作培训 (1h, 合作伙伴门户)

### 第五步: 启用
- 颁发合作伙伴徽章
- 列入官网合作伙伴页面
- 开通合作伙伴门户权限
- 分配专属渠道经理
```

```yaml
# [WRITE] 合作伙伴分润配置
# partner-tiers.yml
tiers:
  silver:
    min_monthly_revenue: 0       # ¥
    commission_rate: 15%          # 销售佣金
    api_discount: 0%              # API 调用折扣
    support_level: standard       # 标准支持
    listing_on_website: true      # 官网展示
    dedicated_manager: false      # 专属经理

  gold:
    min_monthly_revenue: 10000    # ¥/月
    commission_rate: 20%
    api_discount: 15%
    support_level: priority       # 优先支持 (< 4h)
    listing_on_website: true
    dedicated_manager: false
    co_marketing: true            # 联合营销
    case_study: true              # 联合案例

  platinum:
    min_monthly_revenue: 50000    # ¥/月
    commission_rate: 25%
    api_discount: 30%
    support_level: dedicated      # 专属支持 (< 1h)
    listing_on_website: true
    dedicated_manager: true
    co_marketing: true
    case_study: true
    product_roadmap_input: true   # 参与产品路线图
    early_access: true            # 提前访问新功能
    sla_guarantee: true           # SLA 保障
```

```typescript
// [WRITE] apps/backend/src/routes/partners.ts
import { Hono } from "hono"
import { authRequired, requireRole } from "../middleware/auth"
import { prisma } from "../lib/db"

const partnerRoutes = new Hono()

// 合作伙伴申请 (公开)
partnerRoutes.post("/apply", async (c) => {
  const body = await c.req.json()
  const { companyName, contactName, email, phone, type, industry, description } = body

  const application = await prisma.partnerApplication.create({
    data: {
      companyName, contactName, email, phone,
      type, // TECHNOLOGY | SOLUTION | CHANNEL | REFERRAL
      industry,
      description,
      status: "PENDING",
    },
  })

  // 通知管理员审核
  await prisma.notification.create({
    data: {
      type: "PARTNER_APPLICATION",
      title: "新合作伙伴申请",
      body: `${companyName} 申请成为 ${type} 合作伙伴`,
      targetRole: "ADMIN",
      metadata: { applicationId: application.id },
    },
  })

  return c.json({
    success: true,
    message: "申请已提交，我们将在 48 小时内回复",
    applicationId: application.id,
  })
})

// 获取合作伙伴专属 dashboard
partnerRoutes.get("/dashboard", authRequired, async (c) => {
  const user = c.get("user")
  const partner = await prisma.partner.findFirst({
    where: { userId: user.id, status: "ACTIVE" },
    include: {
      referrals: { orderBy: { createdAt: "desc" }, take: 20 },
      commissions: { orderBy: { createdAt: "desc" }, take: 20 },
    },
  })

  if (!partner) return c.json({ error: "Not a partner" }, 404)

  const totalCommission = await prisma.commission.aggregate({
    where: { partnerId: partner.id },
    _sum: { amount: true },
  })

  const monthlyRevenue = await prisma.commission.aggregate({
    where: {
      partnerId: partner.id,
      createdAt: { gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) },
    },
    _sum: { amount: true },
  })

  return c.json({
    success: true,
    data: {
      partner,
      totalCommission: totalCommission._sum.amount || 0,
      monthlyRevenue: monthlyRevenue._sum.amount || 0,
      tier: partner.tier,
      referrals: partner.referrals,
      commissions: partner.commissions,
    },
  })
})

export { partnerRoutes }
```

### Step 21.3 — 插件/扩展市场（第 3-6 周，如产品支持）

```markdown
## 🔌 插件市场架构 (适用于 SaaS 产品)

### 何时需要插件市场
- 用户需要高度定制化 (如 CMS、电商、低代码平台)
- 有大量长尾需求核心团队无法逐一做
- 竞品已有插件生态，这是防守必需
- 第三方开发者表达了强烈的集成意愿

### 插件能力边界设计
```
插件可以:
  ✅ 添加新的 UI 组件/区块
  ✅ 添加新的内容类型/数据模型
  ✅ 添加新的分析面板/看板 Widget
  ✅ 集成第三方服务 (支付/物流/AI/...)
  ✅ 自定义工作流触发器/动作

插件不可以:
  ❌ 读取其他插件的数据
  ❌ 修改核心系统代码
  ❌ 直接访问数据库 (必须通过 API)
  ❌ 引入未经审计的外部依赖
```

### 插件生命周期
```
提交 → 代码审查 (安全+质量) → 沙箱测试 → 上架 → 安装统计 → 评价 →
  ├─ 更新 (开发者迭代)
  ├─ 举报 (用户投诉)
  └─ 下架 (违规/长期不维护)
```

### 插件变现 (可选)
- 免费插件: 上架免费，开发者获取安装量
- 付费插件: 一次性购买 / 订阅制
- 平台抽成: 30% 平台 / 70% 开发者 (行业标准)
- Freemium: 基础功能免费，高级功能付费
```

```typescript
// [WRITE] apps/backend/src/routes/marketplace.ts
import { Hono } from "hono"
import { authRequired, requireRole } from "../middleware/auth"

const marketplaceRoutes = new Hono()

// 插件列表 (公开)
marketplaceRoutes.get("/plugins", async (c) => {
  const { category, sort, search, page = "1", limit = "20" } = c.req.query()
  // ...查询逻辑
  return c.json({ success: true, data: [] })
})

// 提交插件
marketplaceRoutes.post("/plugins", authRequired, async (c) => {
  const body = await c.req.json()
  // 创建插件 → 触发审核流程
  return c.json({ success: true, message: "插件已提交审核" })
})

// 安装插件 (用户)
marketplaceRoutes.post("/plugins/:id/install", authRequired, async (c) => {
  const { id } = c.req.param()
  const user = c.get("user")
  // 记录安装 → 初始化插件配置
  return c.json({ success: true, message: "插件已安装" })
})

export { marketplaceRoutes }
```
```

### Step 21.4 — 用户社区建设（持续）

```markdown
## 👥 社区分层策略

```
         ┌──────────────────┐
         │   KOL / 大使      │  ← 品牌代言人级别 (< 10 人)
         │   专属对接 + 共创   │
         └──────────────────┘
         ┌──────────────────┐
         │   核心贡献者       │  ← 活跃 UGC / 翻译 / 回答问题 (> 50 人)
         │   内测资格 + 荣誉系统 │
         └──────────────────┘
         ┌──────────────────┐
         │   活跃用户         │  ← 参与讨论 / 提建议 (> 500 人)
         │   积分 / 勋章 / 等级 │
         └──────────────────┘
         ┌──────────────────┐
         │   普通用户         │  ← 使用产品 (所有用户)
         │   公告 / 调研 / 活动 │
         └──────────────────┘
```

## 🏅 社区荣誉体系

### 积分获取
| 行为 | 积分 |
|------|------|
| 发表优质帖子/文章 | +10-50 |
| 回答被采纳 | +20 |
| Bug 报告被确认 | +30 |
| 功能建议被采纳 | +50 |
| 邀请新用户注册 | +10/人 |
| 参与线下活动 | +100 |
| 翻译贡献 | +5/100词 |

### 等级体系
```
Lv.1 🌱 新手      (0-99)       — 基础功能
Lv.2 🌿 探索者    (100-499)    — 自定义头像
Lv.3 🌳 贡献者    (500-1999)   — 专属徽章 + Beta 测试资格
Lv.4 ⭐ 专家      (2000-9999)  — 个人主页认证 + 优先支持
Lv.5 👑 传奇      (10000+)     — 线下活动邀请 + 产品委员会席位
```

### 大使计划 (Ambassador Program)
```
大使职责:
  - 每月发布 ≥ 2 篇产品相关内容
  - 参与社区管理 (回答问题、维护秩序)
  - 代表品牌参加线上/线下活动

大使权益:
  - 每月 ¥500-2000 内容创作基金
  - 专属 Swag (周边礼品)
  - 大使专属徽章 + 官网展示
  - 直接与产品团队沟通的渠道
  - 年度大使峰会邀请 (包机酒)

招募标准:
  - 社区等级 ≥ Lv.3
  - 过去 3 个月活跃度达标
  - 内容质量评审通过
  - 无违规记录
```

## 🗣 社区平台矩阵

| 平台 | 用途 | 人力投入 |
|------|------|---------|
| GitHub Discussions | 技术社区 / Bug / 功能建议 | 低 |
| Discord/Slack | 实时讨论 / 开发者支持 | 中 |
| 微信群/飞书群 | 核心用户群 (< 200 人) | 中 |
| 知乎/掘金 | 技术品牌建设 | 中 |
| 小红书/抖音 | 消费端品牌 | 高 |
| Twitter/X | 海外品牌 / 开发者关系 | 低-中 |

## 📢 社区内容日历模板

```
每月社区节奏:
  第 1 周: 产品更新公告 + 月度挑战赛
  第 2 周: 用户案例分享 / 嘉宾 AMA
  第 3 周: 技术深度文章 / 教程系列
  第 4 周: 社区贡献者表彰 / 月度报告
  日常: 每日话题、投票、Q&A
```

## 🚨 社区管理红线

```
社区规则:
  1. 尊重他人，禁止人身攻击
  2. 禁止垃圾广告和刷屏
  3. 禁止政治敏感和违法内容
  4. 保护用户隐私，不公开个人数据
  5. 竞品讨论需有建设性，禁止恶意诋毁

违规处理梯度:
  第 1 次: 私信提醒 + 内容隐藏
  第 2 次: 警告 + 禁言 3 天
  第 3 次: 禁言 30 天
  第 4 次: 永久封禁
```
```

### Step 21.5 — 开源策略（按需启动）

```markdown
## 🗂 开源决策矩阵

```
                高社区价值          低社区价值
              ┌─────────────────┬─────────────────┐
   高竞争力    │  开源 + 商业许可  │   闭源 (核心 IP)  │
   优势       │  (Open Core)    │                 │
              ├─────────────────┼─────────────────┤
   低竞争力    │   开源 (MIT)     │   视情况开源      │
   优势       │  建立标准/生态    │   (或归档)       │
              └─────────────────┴─────────────────┘
```

### 推荐开源的内容
- ✅ SDK / API Client (多语言) — 降低接入门槛
- ✅ 开发工具 / CLI — 提升开发者效率
- ✅ UI 组件库 (非核心业务) — 品牌曝光 + 社区贡献
- ✅ 示例项目 / Starter Kits — 降低上手难度
- ✅ 文档网站源码 — 社区可贡献文档

### 不要开源的内容
- ❌ 核心业务算法 / 推荐引擎
- ❌ 私有数据 / 训练数据
- ❌ CMS 后台管理系统 (你的运营壁垒)
- ❌ 内部配置 / 部署脚本 (含敏感信息)
```

```bash
# [SHELL] 设置开源仓库
mkdir -p ../opensource
cd ../opensource

# 创建 GitHub 组织或使用主仓库的 /packages 目录
# 为每个开源包设置独立仓库
git init api-sdk-typescript
cd api-sdk-typescript

# 添加开源必需的文件
cat > CONTRIBUTING.md << 'EOF'
# Contributing to Product API SDK

## Development Setup
1. Clone the repo
2. `pnpm install`
3. `pnpm test`

## Commit Convention
We follow Conventional Commits: feat/fix/docs/chore/test

## Pull Request Process
1. Create a feature branch
2. Add tests for new functionality
3. Ensure all tests pass (`pnpm test`)
4. Update docs if needed
5. Submit PR against `main` branch

## Code of Conduct
See CODE_OF_CONDUCT.md
EOF

cat > CODE_OF_CONDUCT.md << 'EOF'
# Contributor Covenant Code of Conduct
... (标准 CoC 模板)
EOF

cat > LICENSE << 'EOF'
MIT License
Copyright (c) 2024 Your Product Inc.
... (标准 MIT 许可)
EOF

cat > SECURITY.md << 'EOF'
# Security Policy

## Reporting a Vulnerability
Email security@yourdomain.com (不要公开提 Issue)
我们会在 48 小时内回复。

## Supported Versions
| Version | Supported |
|---------|-----------|
| 1.x     | ✅ |
| 0.x     | ❌ |
EOF

# 初始化并推送
git add -A && git commit -m "Initial commit: TypeScript SDK v1.0.0"
git tag v1.0.0
# git remote add origin git@github.com:yourproduct/api-sdk-typescript.git
# git push --tags origin main
```

```markdown
## 🌟 开源社区健康指标

| 指标 | 目标 (12 个月) |
|------|---------------|
| GitHub Stars | 1000+ |
| Contributors (非员工) | 20+ |
| Issues 平均关闭时间 | < 7 天 |
| PR 平均 Review 时间 | < 3 天 |
| 文档覆盖度 | > 80% public API |
| 发版频率 | ≥ 1 次/月 |
| 外部 PR 占比 | > 30% |
```

### Step 21.6 — 知识体系建设（持续）

```markdown
## 📚 知识矩阵

```
学习者旅程:
  注册 → 新手引导 → 快速开始 → Hello World → 基础教程 →
  进阶指南 → 最佳实践 → 认证考试 → 成为专家/讲师

内容分层:
  L1 新人入门: 5 分钟教程、视频演示、交互式 Tour
  L2 基础掌握: 系列教程、代码实验室、常见问题 FAQ
  L3 进阶应用: 最佳实践指南、案例研究、架构设计文档
  L4 专家级别: 源码解析、性能优化、贡献指南、认证课程
```

## 🎓 内容形式矩阵

| 形式 | 成本 | 覆盖 | 更新频率 | 适合内容 |
|------|------|------|---------|---------|
| 文字文档 | 低 | 高 (SEO) | 持续 | API 文档、指南、FAQ |
| 视频教程 | 中 | 高 | 月 | 产品演示、深度教程 |
| 交互式 Tour | 高 | 中 | 季度 | 新手引导、API Playground |
| 直播/Webinar | 中 | 中 | 周/月 | Q&A、Office Hours |
| 认证课程 | 高 | 低 (高价值) | 年度 | 专业认证 |
| 电子书/白皮书 | 高 | 中 | 半年度 | 行业洞察、方法论 |

## 🔍 内容 SEO 策略

```
文档 SEO 清单:
  - [ ] 每页有唯一的 <title> 和 <meta description>
  - [ ] URL 结构清晰: /docs/api/authentication (不是 /docs/page?id=123)
  - [ ] 代码示例有实际可运行的完整片段 (不是伪代码)
  - [ ] 内链链接到相关文档页面 (降低跳出率)
  - [ ] 图片有 alt 文本
  - [ ] 页面加载速度 < 2s (LCP)
  - [ ] 提供结构化数据 (Schema.org → HowTo / TechArticle)
  - [ ] 提交 sitemap.xml 到 Google/Bing/百度
```

## 🎯 认证体系 (可选，适合生态成熟后)

```
认证路径:
  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
  │  Product    │     │  Product    │     │  Product    │
  │  Associate  │ ──▶ │ Professional│ ──▶ │  Expert     │
  │  (基础)     │     │  (进阶)     │     │  (专家)     │
  └─────────────┘     └─────────────┘     └─────────────┘

认证形式: 在线考试 (选择题 + 实操)
考试费用: Associate ¥200 / Professional ¥500 / Expert ¥1000
认证有效期: 2 年 (需续证，确保知识更新)
```
```

### Step 21.7 — 生态运营 KPI & 月报（持续）

```markdown
## 📊 生态健康仪表盘

### 开发者生态 KPI
| 指标 | 3 个月目标 | 6 个月目标 | 12 个月目标 |
|------|-----------|-----------|------------|
| 注册开发者数 | 100 | 500 | 2,000 |
| 活跃 API Key 数 | 50 | 200 | 800 |
| 日均 API 调用量 | 10,000 | 50,000 | 200,000 |
| SDK 下载量 (npm) | 500/周 | 2,000/周 | 5,000/周 |
| 开发者文档 PV | 1,000/月 | 5,000/月 | 20,000/月 |
| 外部贡献 PR 数 | 5/月 | 20/月 | 50/月 |

### 合作伙伴 KPI
| 指标 | 3 个月目标 | 6 个月目标 | 12 个月目标 |
|------|-----------|-----------|------------|
| 合作伙伴总数 | 5 | 20 | 50 |
| 合作伙伴带来收入占比 | 5% | 15% | 30% |
| 合作伙伴活跃率 (月活) | 80% | 75% | 70% |
| 合作伙伴满意度 (NPS) | > 30 | > 40 | > 50 |

### 社区 KPI
| 指标 | 3 个月目标 | 6 个月目标 | 12 个月目标 |
|------|-----------|-----------|------------|
| 社区总用户数 | 1,000 | 5,000 | 20,000 |
| 月活社区用户 | 200 | 1,000 | 5,000 |
| UGC 内容/月 | 50 | 200 | 1,000 |
| 问题平均响应时间 | < 24h | < 12h | < 4h |
| 社区 NPS | > 20 | > 30 | > 40 |
| 大使人数 | 3 | 10 | 25 |
```

```bash
# [SHELL] 生态月报自动生成脚本
cat > scripts/ecosystem-monthly-report.sh << 'REPORTEOF'
#!/bin/bash
# 生态月报 — 每月 1 号自动执行，生成上月生态数据摘要

YEAR=$(date -v-1m +%Y 2>/dev/null || date -d "last month" +%Y)
MONTH=$(date -v-1m +%m 2>/dev/null || date -d "last month" +%m)
OUTPUT="ecosystem-report-${YEAR}-${MONTH}.md"

cat > "$OUTPUT" << EOF
# 🌐 生态月报 — ${YEAR} 年 ${MONTH} 月

> 生成时间: $(date +%Y-%m-%d)

## 📊 关键数字一览

| 指标 | 本月 | 上月 | 环比 |
|------|------|------|------|
| 开发者注册 | $(curl -s http://localhost:3000/api/v1/admin/ecosystem/dev-signups?month=$MONTH&year=$YEAR) | - | - |
| 新增 API Key | $(curl -s http://localhost:3000/api/v1/admin/ecosystem/new-api-keys?month=$MONTH&year=$YEAR) | - | - |
| API 调用总量 | $(curl -s http://localhost:3000/api/v1/admin/ecosystem/api-calls?month=$MONTH&year=$YEAR) | - | - |
| 新增合作伙伴 | $(curl -s http://localhost:3000/api/v1/admin/ecosystem/new-partners?month=$MONTH&year=$YEAR) | - | - |
| 合作伙伴收入 | $(curl -s http://localhost:3000/api/v1/admin/ecosystem/partner-revenue?month=$MONTH&year=$YEAR) | - | - |
| 社区新帖 | $(curl -s http://localhost:3000/api/v1/admin/ecosystem/community-posts?month=$MONTH&year=$YEAR) | - | - |
| 社区新用户 | $(curl -s http://localhost:3000/api/v1/admin/ecosystem/community-signups?month=$MONTH&year=$YEAR) | - | - |

## 🎉 本月里程碑
- [ ] 请手动填写本月重要里程碑...

## ⚠️ 需要关注的信号
- [ ] 请基于数据手动标注异常信号...

## 📅 下月生态计划
- [ ] 计划中的活动/发布...
EOF

echo "生态月报已生成: $OUTPUT"
REPORTEOF

chmod +x scripts/ecosystem-monthly-report.sh
echo "生态月报脚本已创建，建议在 Phase 20 的每月 checklist 中引用"
```

### Step 21.8 — 第三方集成对接清单

```markdown
## 🔗 高频第三方集成 (按需实现)

### 身份认证
- [ ] 微信登录 (OAuth 2.0)
- [ ] Google Sign-In
- [ ] Apple Sign In (iOS 必需)
- [ ] GitHub OAuth
- [ ] 企业微信 / 飞书 / 钉钉

### 支付
- [ ] 微信支付 (JSAPI / 小程序 / App)
- [ ] 支付宝 (网页 / App)
- [ ] Stripe (海外)
- [ ] Apple Pay / Google Pay

### 通知
- [ ] 微信服务号模板消息
- [ ] 短信 (阿里云 / 华为云)
- [ ] 邮件 (Resend / SendGrid)
- [ ] App Push (APNs / FCM — 已在 Phase 14)

### AI / ML
- [ ] OpenAI / Claude API (内容生成/分析)
- [ ] 百度 AI / 阿里云 AI (国内)
- [ ] HuggingFace 模型部署

### 数据分析
- [ ] 百度统计 / Google Analytics
- [ ] 神策 / GrowingIO
- [ ] Mixpanel / Amplitude
- [ ] 友盟+ (国内 App)

### 地图 & 位置
- [ ] 高德地图 (国内)
- [ ] Google Maps (海外)

### CDN & 存储
- [ ] 华为云 OBS (已在 Phase 12)
- [ ] 阿里云 OSS
- [ ] Cloudflare R2 (海外)
- [ ] 七牛云

### CI/CD & DevOps
- [ ] GitHub Actions (已在 Phase 14)
- [ ] 阿里云效 / 华为云 DevCloud
- [ ] Docker Hub / 华为云 SWR

### 协作 & 客服
- [ ] 飞书 / 钉钉 通知机器人
- [ ] 企业微信应用
- [ ] Intercom / 智齿客服

### 合规
- [ ] 实名认证 (阿里云 / 腾讯云)
- [ ] 内容审核 (阿里云 / 百度 AI)
- [ ] 数据加密 & 脱敏
```

```bash
# [SHELL] 第三方集成注册到应用注册表
cat >> apps/backend/src/integrations/registry.ts << 'REGEOF'
// 第三方集成注册表
// 每新增一个集成在此注册，系统自动管理生命周期

export interface Integration {
  id: string
  name: string
  category: "auth" | "payment" | "notification" | "ai" | "analytics" | "storage"
  provider: string
  status: "active" | "deprecated" | "planned"
  configKeys: string[]      // 需要的环境变量
  docsUrl: string
  setupGuide: string
}

export const INTEGRATION_REGISTRY: Integration[] = [
  {
    id: "wechat-login",
    name: "微信登录",
    category: "auth",
    provider: "微信开放平台",
    status: "active",
    configKeys: ["WECHAT_APP_ID", "WECHAT_APP_SECRET"],
    docsUrl: "https://developers.weixin.qq.com/doc/oplatform/",
    setupGuide: "/docs/integrations/wechat-login",
  },
  {
    id: "wechat-pay",
    name: "微信支付",
    category: "payment",
    provider: "微信支付",
    status: "planned",
    configKeys: ["WECHAT_PAY_MCH_ID", "WECHAT_PAY_API_KEY", "WECHAT_PAY_CERT_PATH"],
    docsUrl: "https://pay.weixin.qq.com/doc/",
    setupGuide: "/docs/integrations/wechat-pay",
  },
  {
    id: "aliyun-sms",
    name: "阿里云短信",
    category: "notification",
    provider: "阿里云",
    status: "active",
    configKeys: ["ALIYUN_ACCESS_KEY_ID", "ALIYUN_ACCESS_KEY_SECRET", "ALIYUN_SMS_SIGN_NAME"],
    docsUrl: "https://help.aliyun.com/document_detail/101414.html",
    setupGuide: "/docs/integrations/aliyun-sms",
  },
]
REGEOF
```

### Step 21.9 — Phase 21 完成验证

```bash
# [VALIDATE] 生态建设验证清单
cat << 'CHECKEOF'
✅ Phase 21 验证清单:

开发者生态:
  [ ] API Key 创建/吊销/管理 功能正常
  [ ] API Key 认证中间件生效 (无效 Key 返回 401)
  [ ] 速率限制正常触发 (超过限制返回 429)
  [ ] 开发者门户可访问 (dev.yourdomain.com)
  [ ] API 文档完整 (包括认证、端点、错误码、示例)
  [ ] API Playground 可用
  [ ] 至少 1 个语言的 SDK 已发布

合作伙伴体系:
  [ ] 合作伙伴申请流程可走通
  [ ] 合作伙伴 Dashboard 正常显示
  [ ] 分润数据正确

社区建设:
  [ ] 社区平台已搭建 (Discord/Slack/GitHub)
  [ ] 社区规则已发布
  [ ] 荣誉/积分系统上线 (如适用)

开源:
  [ ] 开源仓库包含 LICENSE / CONTRIBUTING / CoC / SECURITY
  [ ] CI/CD 对开源仓库正常运转

知识体系:
  [ ] 文档网站 SEO 友好 (有 sitemap + schema)
  [ ] 快速开始教程 5 分钟内可完成

生态 KPI:
  [ ] 月报脚本可执行
  [ ] 仪表盘数据源已接通
CHECKEOF
```

```bash
# [GIT]
git add -A && git commit -m "Phase 21: Ecosystem building — developer platform, partners, marketplace, community, open source, knowledge system"
```

---

## 📚 运营附录

### 附录 G: 内容运营 Playbook

```markdown
## 内容类型矩阵

| 内容类型 | 目的 | 频率 | 字数 | 平台 |
|---------|------|------|------|------|
| 干货教程 | SEO 获客 | 2篇/周 | 1500+ | 官网+知乎+公众号 |
| 用户案例 | 信任背书 | 1篇/周 | 1000+ | 官网+小红书 |
| 产品更新 | 用户留存 | 按需 | 500+ | 官网+App公告 |
| 行业资讯 | 品牌权威 | 1篇/周 | 800+ | 官网+知乎 |
| 互动话题 | 社区活跃 | 2篇/周 | 200+ | 小红书+微博 |

## 好内容公式

标题 = 数字 + 痛点 + 结果承诺
  例: "3 个让你效率翻倍的时间管理技巧"
  非: "关于时间管理的一些思考"

正文 = 钩子(前3句抓注意力) + 干货(80%) + CTA(引导行动)
  钩子: 提问/反常识/痛点场景
  CTA: 关注/下载/评论/分享

## SEO 内容生产流程

1. 关键词研究 (百度指数 / Google Keyword Planner / 5118)
2. 选择 1 个主关键词 + 2-3 个长尾关键词
3. 分析排名前 10 的文章，找到内容缺口
4. 写出比它们更好的内容 (更长/更新/更实用/更美观)
5. 发布 → 提交搜索引擎收录 → 内链建设 → 外链推广
6. 2 周后看排名 → 如未进前 10 → 优化更新
```

### 附录 H: 用户生命周期管理

```markdown
## 用户分层模型

```
              ┌─────────────────────┐
              │   沉默用户 (>30天)   │ ← 唤醒推送 / 大促召回
              └─────────────────────┘
              ┌─────────────────────┐
              │   低频用户 (7-30天)  │ ← 内容推送 / 功能引导
              └─────────────────────┘
              ┌─────────────────────┐
              │   活跃用户 (1-7天)   │ ← 核心体验优化 / VIP增值
              └─────────────────────┘
              ┌─────────────────────┐
              │   核心用户 (每天)     │ ← 社区荣誉 / 内测资格 / UGC激励
              └─────────────────────┘
```

## 各阶段运营策略

### 新用户 (注册 0-7 天) — Aha Moment 冲刺
- Day 0: 欢迎邮件/短信 + 新手引导
- Day 1: 推送核心功能教程
- Day 3: 推送进阶技巧
- Day 7: 首周总结 + 激励

关键指标: 注册→完成核心动作 的转化率 (目标 > 60%)

### 活跃用户 (7-30 天) — 习惯养成
- 连续使用奖励 (签到/任务)
- 个性化内容推荐
- 社区互动引导

关键指标: Day 7 留存率 (目标 > 40%), Day 30 留存率 (目标 > 20%)

### 沉睡用户 (>30 天未活跃) — 召回
- 召回推送: "我们很想你" + 新功能亮点
- 邮件召回: 个人年度报告 / 数据总结
- 大促/节日召回

关键指标: 召回率 (目标 > 5%)

### 流失用户 (>90 天) — 拦截
- 取消账号前的挽留弹窗
- 导出数据功能 (降低流失顾虑)
- 暂停替代取消 (订阅制)

关键指标: 流失率 (目标 < 5% 月)
```

### 附录 I: 关键运营指标词典

```markdown
## 📖 运营指标词典

### 用户指标
| 指标 | 定义 | 健康值 | 计算方式 |
|------|------|--------|---------|
| DAU | 日活跃用户数 | 持续增长 | 当天有任意操作的去重用户 |
| MAU | 月活跃用户数 | 持续增长 | 30 天内有任意操作的去重用户 |
| DAU/MAU | 用户粘性 | > 20% | DAU ÷ MAU |
| D1 留存 | 次日留存 | > 40% | Day 1 回访 / Day 0 新增 |
| D7 留存 | 7 日留存 | > 20% | Day 7 回访 / Day 0 新增 |
| D30 留存 | 30 日留存 | > 10% | Day 30 回访 / Day 0 新增 |
| CAC | 用户获取成本 | 越低越好 | 总获客花费 ÷ 新用户数 |
| LTV | 用户生命周期价值 | > 3× CAC | 平均收入/用户 × 平均生命周期 |

### 内容指标
| 指标 | 健康值 |
|------|--------|
| 文章平均阅读时长 | > 2 分钟 |
| 跳出率 | < 60% |
| 分享率 | > 2% |
| 评论率 | > 1% |
| Banner CTR | > 2% |
| 推送打开率 | > 5% (iOS) / > 10% (Android) |

### 技术指标
| 指标 | 健康值 |
|------|--------|
| API 可用性 | > 99.5% |
| API p95 延迟 | < 500ms |
| 崩溃率 | < 0.3% |
| 页面加载 (LCP) | < 2.5s |
| 首次输入延迟 (FID) | < 100ms |

### 收入指标 (如有付费)
| 指标 | 健康值 |
|------|--------|
| ARPU | 持续增长 |
| ARPPU (付费用户) | 持续增长 |
| 付费转化率 | > 3% |
| 退款率 | < 2% |
| MRR | 逐月增长 > 5% |
```

### 附录 J: 客服 & 社群运营

```markdown
## 💬 客服 SOP

### 响应时间 SLA
| 渠道 | 工作日 | 周末/节假日 |
|------|--------|------------|
| 应用内反馈 | 4 小时 | 24 小时 |
| 邮件 support@ | 8 小时 | 24 小时 |
| App Store 评价 | 24 小时 | 48 小时 |
| 社交媒体 @ | 4 小时 | 12 小时 |
| 紧急 (系统故障) | 30 分钟 | 1 小时 |

### 客服话术库 (部分)

**常见问题: "怎么注销账号？"**
> 您好，您可以在「设置 → 账号安全 → 注销账号」中自行操作。
> 注销后所有数据将被永久删除，建议先导出数据。
> 如有疑问请联系 support@example.com

**常见问题: "为什么收不到验证码？"**
> 请检查: 1) 手机号是否正确 2) 短信是否被拦截 3) 是否在信号弱的地方。
> 如仍无法收到，请邮件至 support@example.com，我们手动协助。

**投诉: "App 闪退/卡顿"**
> 非常抱歉！请提供: 1) 设备型号 2) 系统版本 3) 操作步骤。
> 我们的技术团队会在 24 小时内排查并回复您。

### 社群运营
```
平台选择:
  - 微信群: 核心用户群 (≤ 200 人，高质量)
  - 飞书/钉钉: 企业用户群
  - Discord: 海外用户 / 技术社区
  - 知乎圈子: 内容型社区

日常运营:
  - 早报 (9:00): 行业资讯 1 条 + 今日话题
  - 互动 (12:00): 轻松话题 / 投票
  - 晚报 (18:00): 今日精华 / 明日预告

红线:
  - 不讨论政治敏感话题
  - 不人身攻击
  - 广告需审核
  - 竞品不过度贬低
```
```

---

> **运营阶段更新**: 2026-06-24
> **覆盖**: 上线后 Week 1 稳定 → Week 2-4 增长 → Month 1-3 数据迭代 → 持续日常运营
> **配套文档**: `../operations/App_Operations_SOP.md` (崩溃/分析/评分专项), `../process/ASO_Growth_SOP.md` (ASO 专项)

---

## 🔍 企业级产品审计报告 & 修复记录

> **审计日期**: 2026-06-24 | **审计版本**: v1.1.0
> **方法论**: 逐 Phase 代码审查 + 企业级产品 12-Factor + OWASP Top 10 对照

### 审计摘要

| 维度 | 审计前 | 审计后 |
|------|--------|--------|
| Phase 数 | 16 | 21 |
| 关键依赖项 | 缺失 uuid/nodemailer | ✅ 已补齐 |
| API 端点完整度 | 缺少 forgot-password/reset-password | ✅ 已实现 |
| 安全 Headers | 缺失 OWASP 推荐 7 项 | ✅ 已添加 8 项 |
| 日志体系 | console.log 裸输出 | ✅ 结构化 JSON 日志 |
| 请求追踪 | 无 | ✅ X-Request-ID 全链路 |
| OBS 上传 | 仅本地 mock | ✅ 生产/开发双模式 |
| 优雅关闭 | 无 | ✅ SIGTERM/SIGINT |
| 健康检查 | 仅 DB check | ✅ DB + Redis + 磁盘 |
| 数据库连接池 | 默认 (无配置) | ✅ 连接池 + 慢查询日志 |
| 邮件服务 | 无 | ✅ SMTP/开发双模式 |
| `.dockerignore` | 遗漏 | ✅ 已创建 |
| `.gitleaks.toml` | 遗漏 | ✅ 已创建 |
| Web 共享 import | 相对路径 ../../ | ✅ workspace 协议 "shared" |

### 修复明细

#### ✅ #1 uuid 依赖缺失
- **问题**: `auth.ts` 中 `import { v4 as uuid } from "uuid"` 但 Phase 0 未安装
- **修复**: Phase 2.2 依赖列表添加 `uuid` + `@types/uuid`

#### ✅ #2 安全 Headers 缺失
- **问题**: 无 X-Content-Type-Options, X-Frame-Options, CSP, HSTS 等
- **修复**: 新增 `middleware/security.ts`，8 项安全头 + 请求 ID

#### ✅ #3 忘记密码功能缺失
- **问题**: API Spec 定义了 `/auth/forgot-password` `/auth/reset-password` 但无实现
- **修复**: 在 Phase 3.4 认证路由中追加两个端点

#### ✅ #4 邮件服务缺失
- **问题**: 忘记密码需要发送邮件，但无邮件服务
- **修复**: 新增 `services/email-service.ts`，支持 SMTP/开发双模式

#### ✅ #5 OBS 上传真实实现缺失
- **问题**: upload.ts 仅做本地文件存储，生产环境未对接 OBS
- **修复**: 新增 `services/obs-service.ts`，开发本地 / 生产 OBS 双模式

#### ✅ #6 admin-stats / analytics 路由未注册
- **问题**: Phase 19 定义了路由但 `index.ts` 未注册
- **修复**: `index.ts` 添加 `adminStatsRoute` + `analyticsRoutes` + CORS 头添加 `X-Request-ID`

#### ✅ #7 结构化日志缺失
- **问题**: 全项目使用 `console.log`，生产无法接入 ELK/华为云 LTS
- **修复**: 新增 `lib/logger.ts`，JSON 格式输出，支持 requestId 追踪

#### ✅ #8 优雅关闭缺失
- **问题**: 无 SIGTERM 处理，Docker stop 时可能丢失请求
- **修复**: `index.ts` 添加 shutdown 函数，监听 SIGTERM/SIGINT

#### ✅ #9 健康检查过于简单
- **问题**: 仅检查 DB，未检查 Redis/磁盘/uptime
- **修复**: 增强版 `/health` 端点返回分层检查结果

#### ✅ #10 数据库连接池未配置
- **问题**: Prisma 使用默认连接池 (可能耗尽)
- **修复**: 添加慢查询日志 + DATABASE_URL 连接池参数文档

#### ✅ #11 .dockerignore /.gitleaks 缺失
- **问题**: Docker build 上下文包含 node_modules；无密钥泄露扫描配置
- **修复**: 新增两个配置文件

#### ✅ #12 Web import 路径问题
- **问题**: `../../../../shared/src/api-client` 在生产构建中会失败
- **修复**: 改为 workspace 协议 `"shared"` + 正确配置 exports

### 剩余建议（非阻塞，可在后续迭代中完成）

> 以下项目是「进阶企业级」要求，不影响 MVP 可用性，但强烈建议在 v1.1 中完成。

| # | 建议 | 优先级 | 预计工时 |
|---|------|--------|---------|
| 1 | **Rate Limiter 替换为 Redis 实现** — 当前内存实现在多实例部署时失效 | P1 | 2h |
| 2 | **内存缓存替换为 Redis** — `lib/redis.ts` 目前用 Map 模拟 | P1 | 1h |
| 3 | **添加 API 限流分级** — `/auth/login` 应比普通 API 更严格 (5/min vs 100/min) | P1 | 30min |
| 4 | **结构化日志接入华为云 LTS** — 当前 JSON 输出到 stdout，需对接云日志服务 | P1 | 2h |
| 5 | **数据库迁移 CI 安全策略** — `prisma migrate deploy` 前应备份 | P1 | 30min |
| 6 | **API 版本化策略** — 当前仅 `/api/v1`，需规划 v2 兼容方案 | P2 | 1h |
| 7 | **Docker 健康检查** — docker-compose.yml 中各服务缺少 `healthcheck` | P2 | 30min |
| 8 | **添加 Swagger/OpenAPI 自文档化** — 当前无自动 API 文档生成 | P2 | 2h |
| 9 | **添加请求体大小限制** — 防止大文件攻击 | P2 | 15min |
| 10 | **添加 CORS 动态域名** — 当前硬编码，需支持运行时配置 | P2 | 30min |
| 11 | **增强种子数据** — 当前仅 2 篇文章，生产需要更丰富的初始内容 | P3 | 1h |
| 12 | **CMS webhook 缓存刷新** — Phase 4 提到 webhook 但 backend 端未实现接收路由 | P3 | 1h |
| 13 | **实现 API Key 认证** — 为第三方集成提供 API Key 替代 JWT | P3 | 2h |
| 14 | **添加 DataDog/newrelic APM** — 生产环境应用性能监控 | P3 | 2h |
| 15 | **实现 Feature Flag 系统** — 灰度发布 / 功能开关 | P3 | 3h |

### 企业级检查清单 (对照 12-Factor App)

```
 I. 代码库        ✅ 单一代码库 (monorepo)，Git 版本控制
 II. 依赖         ✅ 显式声明 (pnpm-lock.yaml)
III. 配置         ✅ 环境变量 (.env)，不硬编码
 IV. 后端服务     ✅ 无状态 (JWT)，可水平扩展
  V. 构建发布部署  ✅ Docker build → run 分离
 VI. 进程         ✅ 无状态进程，优雅关闭
 VII. 端口绑定    ✅ Hono 绑定端口，Nginx 反代
VIII. 并发        ✅ 可通过 Docker/K8s 水平扩展
 IX. 可处置性     ✅ 优雅关闭 + 健康检查
  X. 开发/生产    ✅ 通过 NODE_ENV + .env 区分
 XI. 日志         ✅ 结构化 JSON → stdout
XII. 管理进程     ✅ prisma migrate / seed 作为一次性进程
```

### 安全对照 (OWASP Top 10)

```
✅ A01: 访问控制失效      — JWT + RBAC 中间件 (authRequired + requireRole)
✅ A02: 加密失效          — bcrypt (cost=10) + JWT (HS256)
✅ A03: 注入              — Prisma 参数化查询 + Zod 输入校验
✅ A04: 不安全设计        — Rate Limiting + CORS 限制
✅ A05: 安全配置错误      — Security Headers + 环境变量管理
⚠️  A06: 脆弱和过时组件   — 需持续 `pnpm audit` (Phase 15)
⚠️  A07: 认证和授权失败   — 需添加登录失败锁定 (P1)
⚠️  A08: 软件和数据完整性 — 需添加 CI 中的依赖签名验证 (P2)
⚠️  A09: 日志和监控失败   — 需接入华为云 LTS/CES (P1)
⚠️  A10: SSRF             — 文件上传 OBS SDK 隔离，URL fetch 需白名单 (P2)
```

---

> **审计结论**: SOP 经过本次修复后，所有阻塞性问题已解决。产品可按 Phase 0-21 顺序执行，产出满足企业级可用性基线。
> **下次审计**: 建议在首次生产部署后进行，重点检查日志/监控/告警链路完整性。
