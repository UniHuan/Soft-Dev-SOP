# 小程序 App 2天开发 SOP 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 编写完整的小程序 App 2天开发 SOP 文档，供 Claude Code 自动执行

**Architecture:** 参考 iOS/HarmonyOS SOP 文档结构，创建 16 Phase 的详细执行指南，每个 Phase 包含多个 Step，每个 Step 标注技能调用并提供完整代码和命令

**Tech Stack:** uni-app (Vue 3) + TypeScript + Pinia + 云开发

---

## 文件结构

```
miniprogram/
├── MiniProgram_App_2Day_Development_SOP.md  # 主文档（约5000+行）
├── UNI_APP_GUIDE.md                          # uni-app 开发指南（参考文档）
└── CLOUD_DEVELOPMENT_GUIDE.md                # 云开发指南（参考文档）
```

---

## Task 1: 创建 SOP 文档框架和元信息

**Files:**
- Create: `miniprogram/MiniProgram_App_2Day_Development_SOP.md`

- [ ] **Step 1.1: 创建文档头部和元信息**

```markdown
# 📱 Claude Code + HBuilderX 两天全自动小程序 App 开发 SOP

> **适用对象**: Claude Code (AI Agent) 全自动执行
> **目标**: 从零到三端小程序上架，两天 (16 小时) 完成
> **技术栈**: uni-app (Vue 3) + TypeScript + Pinia + 云开发
> **目标平台**: 微信小程序、支付宝小程序、抖音小程序
> **最低要求**: Node.js 18+, HBuilderX 4.0+ 或 VS Code, 微信开发者工具

---

## 🧠 Claude Code 技能调用矩阵

> **每个步骤标注了 Claude Code 需要调用的核心技能。Claude Code 在读到该步骤时自动切换对应模式。**

| 技能标识 | 技能名称 | 说明 | Claude Code 工具 |
|---------|---------|------|-----------------|
| `[SHELL]` | Shell 执行 | 执行 bash 命令、npx、git | `Bash` |
| `[WRITE]` | 文件写入 | 创建/修改 .vue .ts .json 文件 | `Edit` / `Write` |
| `[READ]` | 文件读取 | 读取现有代码、日志、配置 | `Read` |
| `[DIALOG]` | 用户交互 | 向用户提问、请求确认、展示结果 | 对话输出 |
| `[GENERATE]` | 代码生成 | 生成页面、组件、云函数代码 | `Edit` / `Write` |
| `[REVIEW]` | 代码审查 | 审查生成代码的质量、性能、安全性 | `Read` → 分析 |
| `[DEBUG]` | 调试分析 | 分析编译错误/运行时错误并修复 | `Read` + `Bash` + `Edit` |
| `[RESEARCH]` | 知识检索 | 查阅 uni-app 文档、各平台 API 差异 | `Read` 参考文档 |
| `[VALIDATE]` | 验证检查 | 对照清单逐项验证 | `Bash` + `Read` + 分析 |
| `[GIT]` | 版本控制 | git add/commit/tag/push | `Bash` |

### 技能调用原则

\`\`\`
1. 每个 Phase 开始前 → [RESEARCH] 查阅本 SOP 的 uni-app 约束和平台差异
2. 每次生成代码后 → [REVIEW] 对照前后步骤自检代码质量
3. 每次编译后 → [DEBUG] 分析输出、自动修复、不把错误抛给用户
4. 每个 Phase 结束前 → [VALIDATE] 对照该 Phase 的验收条件逐项确认
5. 每个 Phase 结束后 → [GIT] 提交代码
6. 涉及不可逆操作 → [DIALOG] 必须获得用户确认
\`\`\`
```

- [ ] **Step 1.2: 添加总览时间线**

```markdown
---

## 📋 总览时间线

\`\`\`
Day 1 (8h)                              Day 2 (8h)
├─ [0.0h] 环境检查 & 项目初始化          ├─ [0.0h] 云函数开发
├─ [0.5h] 产品需求确认                   ├─ [1.0h] 支付集成
├─ [1.0h] PRD 产品需求文档                ├─ [2.0h] 平台适配
├─ [1.5h] 页面路由配置                   ├─ [3.0h] 功能完善
├─ [2.0h] 云开发环境配置                 ├─ [4.0h] 测试与调试
├─ [2.5h] 基础组件封装                   ├─ [5.5h] 审核上架
├─ [4.0h] 首页开发                       ├─ [6.5h] 发布管理
├─ [5.5h] 用户系统开发                   └─ [7.5h] 运营迭代
├─ [7.0h] 数据模型设计
└─ [8.0h] 自测 & 代码 Review
\`\`\`
```

- [ ] **Step 1.3: 添加执行前提检查**

```markdown
---

## ⚠️ 执行前提 (Claude Code 自动检查)

在执行本 SOP 前，Claude Code 必须逐条确认以下条件：

\`\`\`bash
# [SHELL] 1. 确认 Node.js 版本 >= 18.0
NODE_VERSION=$(node -v | sed 's/v//' | cut -d. -f1)
if [ "$NODE_VERSION" -lt 18 ]; then
    echo "❌ 需要 Node.js 18.0+，当前: $(node -v)"
    exit 1
fi
echo "✅ Node.js: $(node -v)"

# [SHELL] 2. 确认 npm/pnpm 可用
which npm >/dev/null 2>&1 && echo "✅ npm: $(npm -v)" || { echo "❌ npm 未安装"; exit 1; }

# [SHELL] 3. 确认 Git 已配置
git --version || { echo "❌ Git 未安装"; exit 1; }
git config user.name || echo "⚠️ Git user.name 未配置"
git config user.email || echo "⚠️ Git user.email 未配置"

# [SHELL] 4. 检查微信开发者工具 CLI (可选)
WX_CLI="/Applications/wechatwebdevtools.app/Contents/MacOS/cli"
if [ -f "$WX_CLI" ]; then
    echo "✅ 微信开发者工具 CLI 可用"
else
    echo "⚠️ 微信开发者工具 CLI 未找到，需要手动操作"
fi

# [SHELL] 5. 检查 HBuilderX (可选)
if [ -d "/Applications/HBuilderX.app" ]; then
    echo "✅ HBuilderX 已安装"
else
    echo "⚠️ HBuilderX 未安装，将使用 VS Code + CLI"
fi

# [SHELL] 6. 保存环境变量到临时文件
cat > /tmp/sop_miniprogram.env << 'EOF'
PROJECT_NAME="MyMiniApp"
PROJECT_DIR="/Users/xurui/Projects/SOP/MyMiniApp"
EOF
echo "✅ 环境变量已保存到 /tmp/sop_miniprogram.env"
\`\`\`

> **如果任一检查失败**: 立即停止，向用户报告缺失项和修复方法，不要继续执行。
```

- [ ] **Step 1.4: 提交框架代码**

```bash
git add miniprogram/MiniProgram_App_2Day_Development_SOP.md
git commit -m "docs: add mini program SOP framework

- Add document header and metadata
- Add skill matrix and timeline
- Add prerequisites checklist

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 2: 编写 Phase 0-2 (环境初始化和项目配置)

**Files:**
- Modify: `miniprogram/MiniProgram_App_2Day_Development_SOP.md`

- [ ] **Step 2.1: 编写 Phase 0 环境初始化**

```markdown
---

# 🗓️ DAY 1 — 从零到可运行 MVP

---

## Phase 0: 环境初始化 (0:00 - 0:30)

> **[SHELL]** + **[DIALOG]** 环境检查和项目骨架创建

### Step 0.0 — 环境自检 & 工具确认

\`\`\`bash
# [SHELL] 加载环境变量
source /tmp/sop_miniprogram.env

# [SHELL] 检查并安装全局工具
if ! command -v degit &> /dev/null; then
    echo "正在安装 degit (项目模板工具)..."
    npm install -g degit
fi

# [SHELL] 检查 pnpm (推荐)
if ! command -v pnpm &> /dev/null; then
    echo "正在安装 pnpm..."
    npm install -g pnpm
fi

echo "✅ 环境检查完成"
\`\`\`

### Step 0.1 — 创建项目目录结构

> **[SHELL]** + **[WRITE]** 创建完整的项目目录

\`\`\`bash
# [SHELL] 定义项目变量
source /tmp/sop_miniprogram.env
PROJECT_NAME="MyMiniApp"
APP_ID="wx1234567890abcdef"  # 用户需要替换为实际AppID

# 更新环境变量
cat >> /tmp/sop_miniprogram.env << EOF
PROJECT_NAME="$PROJECT_NAME"
APP_ID="$APP_ID"
EOF

# [SHELL] 创建项目目录
mkdir -p $PROJECT_NAME
cd $PROJECT_NAME

# 创建源码目录结构
mkdir -p src/pages/index
mkdir -p src/pages/list
mkdir -p src/pages/detail
mkdir -p src/pages/profile
mkdir -p src/components/common
mkdir -p src/components/business
mkdir -p src/api
mkdir -p src/store
mkdir -p src/utils
mkdir -p src/static/images
mkdir -p src/static/icons
mkdir -p cloudfunctions/user
mkdir -p cloudfunctions/data
mkdir -p cloudfunctions/payment
mkdir -p tests/unit
mkdir -p tests/integration

echo "✅ 项目目录结构创建完成"
\`\`\`

### Step 0.2 — 初始化 uni-app 项目

> **[SHELL]** 使用官方模板创建项目

\`\`\`bash
# [SHELL] 使用 degit 克隆 uni-app Vue3 + TypeScript 模板
cd /Users/xurui/Projects/SOP
npx degit dcloudio/uni-preset-vue#vite-ts $PROJECT_NAME

# [SHELL] 进入项目目录并安装依赖
cd $PROJECT_NAME
pnpm install

# [SHELL] 验证项目结构
ls -la src/
\`\`\`

### Step 0.3 — 配置 TypeScript 和 ESLint

> **[WRITE]** 创建配置文件

\`\`\`typescript
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "strict": true,
    "jsx": "preserve",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "esModuleInterop": true,
    "lib": ["ESNext", "DOM"],
    "skipLibCheck": true,
    "noEmit": true,
    "paths": {
      "@/*": ["./src/*"]
    },
    "types": [
      "@dcloudio/types",
      "miniprogram-api-typings"
    ]
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"],
  "exclude": ["node_modules", "dist", "unpackage"]
}
\`\`\`

\`\`\`bash
# [SHELL] 提交 Phase 0
git add .
git commit -m "chore: initialize uni-app project with TypeScript

- Create project structure
- Configure TypeScript

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
\`\`\`

### ✅ Phase 0 验收条件

- [ ] Node.js 18+ 已安装
- [ ] pnpm 已安装
- [ ] 项目目录结构创建完成
- [ ] uni-app 模板项目初始化成功
- [ ] TypeScript 配置完成
```

- [ ] **Step 2.2: 编写 Phase 1 页面路由配置**

```markdown
---

## Phase 1: 页面路由配置 (0:30 - 1:00)

> **[WRITE]** + **[GENERATE]** 配置 pages.json 和 TabBar

### Step 1.0 — 配置 pages.json

> **[WRITE]** 定义页面路由和全局配置

\`\`\`json
{
  "pages": [
    {
      "path": "pages/index/index",
      "style": {
        "navigationBarTitleText": "首页",
        "enablePullDownRefresh": true
      }
    },
    {
      "path": "pages/list/list",
      "style": {
        "navigationBarTitleText": "列表",
        "enablePullDownRefresh": true
      }
    },
    {
      "path": "pages/detail/detail",
      "style": {
        "navigationBarTitleText": "详情"
      }
    },
    {
      "path": "pages/profile/profile",
      "style": {
        "navigationBarTitleText": "我的"
      }
    }
  ],
  "globalStyle": {
    "navigationBarTextStyle": "black",
    "navigationBarTitleText": "MyMiniApp",
    "navigationBarBackgroundColor": "#ffffff",
    "backgroundColor": "#f8f8f8"
  },
  "tabBar": {
    "color": "#999999",
    "selectedColor": "#007AFF",
    "borderStyle": "black",
    "backgroundColor": "#ffffff",
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "首页",
        "iconPath": "static/icons/home.png",
        "selectedIconPath": "static/icons/home-active.png"
      },
      {
        "pagePath": "pages/list/list",
        "text": "列表",
        "iconPath": "static/icons/list.png",
        "selectedIconPath": "static/icons/list-active.png"
      },
      {
        "pagePath": "pages/profile/profile",
        "text": "我的",
        "iconPath": "static/icons/profile.png",
        "selectedIconPath": "static/icons/profile-active.png"
      }
    ]
  }
}
\`\`\`

### Step 1.1 — 创建占位页面

> **[GENERATE]** 创建基础页面组件

\`\`\`vue
<!-- src/pages/index/index.vue -->
<template>
  <view class="container">
    <text>首页 - 开发中</text>
  </view>
</template>

<script setup lang="ts">
import { onLoad } from '@dcloudio/uni-app'

onLoad(() => {
  console.log('首页加载')
})
</script>

<style scoped>
.container {
  padding: 20rpx;
}
</style>
\`\`\`

\`\`\`vue
<!-- src/pages/list/list.vue -->
<template>
  <view class="container">
    <text>列表页 - 开发中</text>
  </view>
</template>

<script setup lang="ts">
</script>

<style scoped>
.container {
  padding: 20rpx;
}
</style>
\`\`\`

\`\`\`vue
<!-- src/pages/detail/detail.vue -->
<template>
  <view class="container">
    <text>详情页 - 开发中</text>
  </view>
</template>

<script setup lang="ts">
</script>

<style scoped>
.container {
  padding: 20rpx;
}
</style>
\`\`\`

\`\`\`vue
<!-- src/pages/profile/profile.vue -->
<template>
  <view class="container">
    <text>个人中心 - 开发中</text>
  </view>
</template>

<script setup lang="ts">
</script>

<style scoped>
.container {
  padding: 20rpx;
}
</style>
\`\`\`

### Step 1.2 — 配置 manifest.json

> **[WRITE]** 配置应用基本信息

\`\`\`json
{
  "name": "MyMiniApp",
  "appid": "__UNI__XXXXXXX",
  "description": "通用小程序模板",
  "versionName": "1.0.0",
  "versionCode": "100",
  "transformPx": false,
  "mp-weixin": {
    "appid": "wx1234567890abcdef",
    "setting": {
      "urlCheck": false,
      "es6": true,
      "postcss": true,
      "minified": true
    },
    "usingComponents": true,
    "permission": {
      "scope.userLocation": {
        "desc": "您的位置信息将用于..."
      }
    }
  },
  "mp-alipay": {
    "appid": "2021XXXXXXXXXXXX",
    "usingComponents": true
  },
  "mp-toutiao": {
    "appid": "tt1234567890abcdef",
    "usingComponents": true
  }
}
\`\`\`

### Step 1.3 — 验证页面路由

\`\`\`bash
# [SHELL] 编译微信小程序
pnpm dev:mp-weixin

# [SHELL] 检查编译输出
ls -la dist/dev/mp-weixin/

# [SHELL] 提交 Phase 1
git add .
git commit -m "feat: configure pages and tabBar

- Add pages.json with 4 pages
- Create placeholder pages
- Configure manifest.json for 3 platforms

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
\`\`\`

### ✅ Phase 1 验收条件

- [ ] pages.json 配置正确，包含4个页面
- [ ] TabBar 配置正确，包含3个标签
- [ ] 占位页面创建完成
- [ ] 编译成功，无错误
```

- [ ] **Step 2.3: 编写 Phase 2 云开发环境配置**

```markdown
---

## Phase 2: 云开发环境配置 (1:00 - 1:30)

> **[DIALOG]** + **[WRITE]** 配置云开发环境

### Step 2.0 — 创建云开发环境说明

> **[DIALOG]** 向用户说明云开发配置步骤

```
⚠️ 此步骤需要用户在微信开发者工具中手动操作：

1. 打开微信开发者工具
2. 导入项目：dist/dev/mp-weixin
3. 点击"云开发"按钮
4. 创建云开发环境（选择免费额度即可）
5. 记录环境 ID (如：cloud1-xxx)

完成后，请输入您的云开发环境 ID：
```

### Step 2.1 — 配置云开发环境 ID

> **[WRITE]** 保存云开发配置

\`\`\`bash
# [SHELL] 保存云开发环境 ID
# 用户输入后执行
read -p "请输入云开发环境 ID: " CLOUD_ENV_ID
echo "CLOUD_ENV_ID=\"$CLOUD_ENV_ID\"" >> /tmp/sop_miniprogram.env
\`\`\`

\`\`\`javascript
// src/utils/cloud.ts
const CLOUD_ENV_ID = import.meta.env.VITE_CLOUD_ENV_ID || ''

export function initCloud() {
  // #ifdef MP-WEIXIN
  if (!wx.cloud) {
    console.error('请使用 2.2.3 或以上的基础库以使用云能力')
    return
  }
  wx.cloud.init({
    env: CLOUD_ENV_ID,
    traceUser: true
  })
  // #endif
}

export function getCloudDB() {
  // #ifdef MP-WEIXIN
  return wx.cloud.database()
  // #endif
  return null
}

export function getCloudFunction(name: string) {
  // #ifdef MP-WEIXIN
  return wx.cloud.callFunction({ name })
  // #endif
  return Promise.reject('当前平台不支持云函数')
}
\`\`\`

### Step 2.2 — 创建数据库集合

> **[DIALOG]** 引导用户创建数据库集合

```
请在微信开发者工具云开发控制台中创建以下数据库集合：

1. users - 用户信息表
2. contents - 内容表
3. orders - 订单表

权限设置：
- users: 仅创建者可写，所有人可读
- contents: 仅创建者可写，所有人可读
- orders: 仅创建者可写，仅创建者可读
```

### Step 2.3 — 创建云函数目录结构

> **[WRITE]** 创建云函数模板

\`\`\`javascript
// cloudfunctions/user/index.js
const cloud = require('wx-server-sdk')

cloud.init({
  env: cloud.DYNAMIC_CURRENT_ENV
})

const db = cloud.database()

exports.main = async (event, context) => {
  const { action, data } = event
  const wxContext = cloud.getWXContext()
  
  switch (action) {
    case 'login':
      // 用户登录
      const user = await db.collection('users').where({
        openid: wxContext.OPENID
      }).get()
      
      if (user.data.length === 0) {
        // 新用户注册
        const result = await db.collection('users').add({
          data: {
            openid: wxContext.OPENID,
            createTime: db.serverDate(),
            updateTime: db.serverDate()
          }
        })
        return { success: true, userId: result._id }
      }
      return { success: true, userId: user.data[0]._id }
      
    default:
      return { success: false, message: '未知操作' }
  }
}
\`\`\`

\`\`\`json
// cloudfunctions/user/package.json
{
  "name": "user",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "wx-server-sdk": "~2.6.3"
  }
}
\`\`\`

### Step 2.4 — 提交 Phase 2

\`\`\`bash
git add .
git commit -m "feat: configure cloud development

- Add cloud utilities
- Create user cloud function template
- Add database collection guide

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
\`\`\`

### ✅ Phase 2 验收条件

- [ ] 云开发环境创建成功
- [ ] 环境ID已配置
- [ ] 数据库集合创建完成
- [ ] 云函数目录结构创建完成
```

- [ ] **Step 2.4: 提交 Phase 0-2 代码**

```bash
git add miniprogram/MiniProgram_App_2Day_Development_SOP.md
git commit -m "docs: add mini program SOP Phase 0-2

- Phase 0: Environment initialization
- Phase 1: Pages and routing configuration
- Phase 2: Cloud development setup

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 3: 编写 Phase 3-6 (基础组件和首页开发)

**Files:**
- Modify: `miniprogram/MiniProgram_App_2Day_Development_SOP.md`

- [ ] **Step 3.1: 编写 Phase 3 基础组件封装**

添加 Phase 3 内容，包含：
- 导航栏组件
- 列表项组件
- 空状态组件
- 加载状态组件
- 网络请求封装

- [ ] **Step 3.2: 编写 Phase 4 首页开发**

添加 Phase 4 内容，包含：
- Banner 轮播组件
- 功能入口网格
- 推荐列表组件
- 首页完整实现

- [ ] **Step 3.3: 编写 Phase 5 数据模型设计**

添加 Phase 5 内容，包含：
- 云数据库 Schema 设计
- 数据模型 TypeScript 接口
- 初始化数据脚本

- [ ] **Step 3.4: 编写 Phase 6 用户系统开发**

添加 Phase 6 内容，包含：
- 用户登录授权流程
- Pinia Store 用户状态管理
- 用户信息页面

- [ ] **Step 3.5: 提交 Phase 3-6 代码**

```bash
git add miniprogram/MiniProgram_App_2Day_Development_SOP.md
git commit -m "docs: add mini program SOP Phase 3-6

- Phase 3: Base components
- Phase 4: Home page development
- Phase 5: Data models
- Phase 6: User system

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 4: 编写 Phase 7-10 (云函数、支付、平台适配)

**Files:**
- Modify: `miniprogram/MiniProgram_App_2Day_Development_SOP.md`

- [ ] **Step 4.1: 编写 Phase 7 云函数开发**

添加 Phase 7 内容，包含：
- 用户云函数 (login, updateProfile)
- 数据云函数 (CRUD operations)
- 权限校验中间件

- [ ] **Step 4.2: 编写 Phase 8 支付集成**

添加 Phase 8 内容，包含：
- 微信支付流程
- 支付宝支付流程
- 订单管理系统
- 支付回调处理

- [ ] **Step 4.3: 编写 Phase 9 平台适配**

添加 Phase 9 内容，包含：
- 条件编译语法说明
- 微信/支付宝/抖音 平台差异处理
- 样式适配
- API 差异封装

- [ ] **Step 4.4: 编写 Phase 10 功能完善**

添加 Phase 10 内容，包含：
- 收藏功能
- 评论系统
- 分享功能
- 搜索功能

- [ ] **Step 4.5: 提交 Phase 7-10 代码**

```bash
git add miniprogram/MiniProgram_App_2Day_Development_SOP.md
git commit -m "docs: add mini program SOP Phase 7-10

- Phase 7: Cloud functions
- Phase 8: Payment integration
- Phase 9: Platform adaptation
- Phase 10: Feature completion

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 5: 编写 Phase 11-13 (测试、审核、发布)

**Files:**
- Modify: `miniprogram/MiniProgram_App_2Day_Development_SOP.md`

- [ ] **Step 5.1: 编写 Phase 11 测试与调试**

添加 Phase 11 内容，包含：
- 单元测试配置
- 组件测试
- 真机调试指南
- 性能优化建议

- [ ] **Step 5.2: 编写 Phase 12 审核上架**

添加 Phase 12 内容，包含：
- 微信小程序审核流程
- 支付宝小程序审核流程
- 抖音小程序审核流程
- 隐私协议配置
- 资质准备清单

- [ ] **Step 5.3: 编写 Phase 13 发布管理**

添加 Phase 13 内容，包含：
- 版本号管理规范
- 灰度发布流程
- 回滚机制
- 更新日志格式

- [ ] **Step 5.4: 提交 Phase 11-13 代码**

```bash
git add miniprogram/MiniProgram_App_2Day_Development_SOP.md
git commit -m "docs: add mini program SOP Phase 11-13

- Phase 11: Testing and debugging
- Phase 12: Review and publishing
- Phase 13: Release management

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 6: 编写 Phase 14-16 和参考文档

**Files:**
- Modify: `miniprogram/MiniProgram_App_2Day_Development_SOP.md`
- Create: `miniprogram/UNI_APP_GUIDE.md`
- Create: `miniprogram/CLOUD_DEVELOPMENT_GUIDE.md`

- [ ] **Step 6.1: 编写 Phase 14-16**

添加剩余 Phase 内容：
- Phase 14: 版本管理与迭代
- Phase 15: 运营监控
- Phase 16: 文档归档

- [ ] **Step 6.2: 创建 uni-app 开发指南**

\`\`\`markdown
# uni-app 开发指南

## 条件编译

### 平台判断

\`\`\`vue
<!-- #ifdef MP-WEIXIN -->
<view>微信小程序专属内容</view>
<!-- #endif -->

<!-- #ifdef MP-ALIPAY -->
<view>支付宝小程序专属内容</view>
<!-- #endif -->

<!-- #ifdef MP-TOUTIAO -->
<view>抖音小程序专属内容</view>
<!-- #endif -->
\`\`\`

### API 差异处理

\`\`\`typescript
// 获取用户信息
export function getUserInfo() {
  // #ifdef MP-WEIXIN
  return wx.getUserProfile({ desc: '用于完善用户资料' })
  // #endif

  // #ifdef MP-ALIPAY
  return my.getOpenUserInfo()
  // #endif

  // #ifdef MP-TOUTIAO
  return tt.getUserInfo()
  // #endif
}
\`\`\`

## 常见问题

### 1. 样式单位

使用 rpx 作为响应式单位，设计稿以 750px 为基准。

### 2. 生命周期

- onLoad: 页面加载时
- onShow: 页面显示时
- onReady: 页面渲染完成时
- onHide: 页面隐藏时
- onUnload: 页面卸载时
\`\`\`

- [ ] **Step 6.3: 创建云开发指南**

\`\`\`markdown
# 微信云开发指南

## 数据库操作

### 查询数据

\`\`\`javascript
const db = wx.cloud.database()
const result = await db.collection('users').where({
  age: _.gt(18)
}).get()
\`\`\`

### 添加数据

\`\`\`javascript
const result = await db.collection('users').add({
  data: {
    name: '张三',
    age: 25
  }
})
\`\`\`

## 云函数

### 调用云函数

\`\`\`javascript
const result = await wx.cloud.callFunction({
  name: 'user',
  data: {
    action: 'login',
    data: { nickname: '用户' }
  }
})
\`\`\`

## 云存储

### 上传文件

\`\`\`javascript
const result = await wx.cloud.uploadFile({
  cloudPath: 'images/test.jpg',
  filePath: localPath
})
\`\`\`
\`\`\`

- [ ] **Step 6.4: 最终提交**

```bash
git add .
git commit -m "docs: complete mini program SOP documentation

- Add Phase 14-16
- Add uni-app development guide
- Add cloud development guide

Total: 16 Phases, complete documentation

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 7: 更新 README 和项目文档

**Files:**
- Modify: `README.md`

- [ ] **Step 7.1: 更新 README 添加小程序 SOP 条目**

在 README.md 的平台开发表格中添加：

```markdown
### 小程序开发

| 文档 | 说明 | 行数 |
|------|------|------|
| [小程序 App 2 天开发 SOP](miniprogram/MiniProgram_App_2Day_Development_SOP.md) | uni-app + Vue3 + TypeScript，16 Phase 全自动执行 | 5,000+ |
| [uni-app 开发指南](miniprogram/UNI_APP_GUIDE.md) | 条件编译、平台差异、最佳实践 | 300+ |
| [云开发指南](miniprogram/CLOUD_DEVELOPMENT_GUIDE.md) | 云数据库、云函数、云存储 | 200+ |
```

- [ ] **Step 7.2: 更新项目统计**

```markdown
| 指标 | 数值 |
|------|------|
| SOP 文档数 | 17 |
| 总行数 | 23,000+ |
| Phase/Step 总数 | 110+ Phase，280+ Step |
```

- [ ] **Step 7.3: 提交 README 更新**

```bash
git add README.md
git commit -m "docs: add mini program SOP to README

- Add mini program development section
- Update project statistics

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Self-Review Checklist

- [ ] **Spec Coverage**: 所有16个Phase都已实现
- [ ] **Placeholder Scan**: 无 "TBD", "TODO", "implement later" 等
- [ ] **Type Consistency**: TypeScript 接口在各 Phase 中保持一致
- [ ] **Code Completeness**: 所有代码步骤都有完整代码
- [ ] **Command Accuracy**: 所有 bash 命令可直接执行
