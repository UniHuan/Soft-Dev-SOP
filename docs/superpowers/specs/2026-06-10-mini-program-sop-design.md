# 小程序 App 2天开发 SOP 设计文档

> 创建日期：2026-06-10
> 状态：已批准

---

## 1. 概述

### 1.1 基本信息

| 项目 | 内容 |
|------|------|
| 名称 | 小程序 App 2天开发 SOP |
| 技术栈 | uni-app (Vue 3) + TypeScript + Pinia |
| 目标平台 | 微信、支付宝、抖音小程序 |
| 后端方案 | 云开发（微信云开发 / 支付宝云开发） |
| 开发周期 | 2天（16 Phases） |

### 1.2 设计目标

- 填补现有SOP体系在小程序领域的空白
- 提供一套代码编译三端的能力
- 降低后端门槛，使用云开发零服务器方案
- 复用现有SOP框架，保持一致性

---

## 2. 架构设计

### 2.1 项目结构

```
项目结构
├── src/
│   ├── pages/              # 页面目录
│   │   ├── index/          # 首页
│   │   ├── list/           # 列表页
│   │   ├── detail/         # 详情页
│   │   └── profile/        # 个人中心
│   ├── components/         # 公共组件
│   ├── api/                # 云函数调用封装
│   ├── store/              # Pinia状态管理
│   ├── utils/              # 工具函数
│   └── static/             # 静态资源
├── cloudfunctions/         # 云函数目录
│   ├── user/               # 用户相关
│   ├── data/               # 数据操作
│   └── payment/            # 支付处理
├── manifest.json           # 应用配置
├── pages.json              # 页面路由配置
└── uni.scss                # 全局样式变量
```

### 2.2 技术选型理由

| 技术 | 选型理由 |
|------|----------|
| Vue 3 + Composition API | uni-app生态Vue更成熟，社区资源丰富 |
| Pinia | uni-app官方推荐，比Vuex更轻量 |
| TypeScript | 与现有SOP保持一致，类型安全 |
| uni-app | 国内最流行的跨端小程序框架，支持三端编译 |

---

## 3. 核心组件设计

### 3.1 示例项目：通用小程序模板

| 模块 | 组件/功能 | 说明 |
|------|-----------|------|
| **用户系统** | 登录授权、用户信息、权限管理 | 微信/支付宝/抖音一键登录，自动适配 |
| **首页模块** | Banner轮播、功能入口、推荐列表 | 可配置化，支持运营位 |
| **列表模块** | 下拉刷新、上拉加载、筛选排序 | 复用性高，支持虚拟列表优化 |
| **详情模块** | 富文本渲染、评论系统、收藏分享 | 支持markdown/富文本解析 |
| **个人中心** | 用户资料、我的收藏、设置反馈 | 标准用户中心模板 |
| **支付模块** | 微信支付、支付宝支付 | 订单创建→支付→回调通知完整流程 |
| **云开发模块** | 云数据库、云函数、云存储 | 数据模型设计、文件上传下载 |

### 3.2 技能标注映射

| 标签 | 小程序SOP应用场景 |
|------|-------------------|
| `[SHELL]` | uni-cli 创建项目、安装依赖、编译运行 |
| `[WRITE]` | 创建页面、组件、配置文件 |
| `[GENERATE]` | 生成云函数模板、API封装代码 |
| `[DEBUG]` | 微信开发者工具调试、真机调试 |
| `[DIALOG]` | 云开发环境创建、支付配置确认 |
| `[REVIEW]` | 代码审查、性能检查 |
| `[VALIDATE]` | 多端编译验证、真机测试 |
| `[RESEARCH]` | uni-app文档、各平台API差异 |
| `[GIT]` | 版本提交 |

---

## 4. 数据流设计

### 4.1 数据流向

```
用户操作 → 页面组件 → Pinia Store → 云函数 → 云数据库
                ↓
            云存储（文件）
                ↓
            第三方API（可选）
```

### 4.2 数据模型

#### 用户表

```typescript
interface User {
  _id: string
  openid: string           // 微信openid / 支付宝user_id / 抖音open_id
  platform: 'wechat' | 'alipay' | 'douyin'
  nickname: string
  avatar: string
  createTime: Date
  updateTime: Date
}
```

#### 内容表（通用）

```typescript
interface Content {
  _id: string
  title: string
  cover: string
  content: string          // 富文本/Markdown
  authorId: string
  viewCount: number
  likeCount: number
  tags: string[]
  status: 'draft' | 'published'
  createTime: Date
  updateTime: Date
}
```

#### 订单表

```typescript
interface Order {
  _id: string
  orderId: string          // 商户订单号
  userId: string
  amount: number
  status: 'pending' | 'paid' | 'refunded'
  platform: 'wechat' | 'alipay'
  transactionId: string    // 平台交易号
  createTime: Date
  payTime?: Date
}
```

### 4.3 云函数设计原则

- **单一职责**：每个云函数只做一件事
- **权限校验**：所有写操作验证用户身份
- **错误处理**：统一错误码规范
- **日志记录**：关键操作记录日志便于排查

---

## 5. 错误处理设计

### 5.1 错误分类

| 类型 | 场景 | 处理方式 |
|------|------|----------|
| **网络错误** | 请求超时、服务器无响应 | Toast提示 + 重试按钮 |
| **业务错误** | 参数校验失败、权限不足 | Toast提示具体原因 |
| **授权错误** | 用户拒绝授权、登录过期 | 引导重新授权/登录 |
| **支付错误** | 支付取消、余额不足 | 明确提示 + 订单状态更新 |
| **云函数错误** | 数据库操作失败、逻辑异常 | 统一错误码 + 日志记录 |

### 5.2 错误码规范

```typescript
const ERROR_CODES = {
  // 通用错误 1xxx
  1001: '参数错误',
  1002: '未授权，请先登录',
  1003: '权限不足',

  // 用户相关 2xxx
  2001: '用户不存在',
  2002: '登录已过期',

  // 业务相关 3xxx
  3001: '内容不存在',
  3002: '已收藏',

  // 支付相关 4xxx
  4001: '订单不存在',
  4002: '订单已支付',
  4003: '支付失败',
}
```

### 5.3 全局错误捕获

```typescript
// App.vue 生命周期
onError((error) => {
  console.error('全局错误:', error)
})

// 云函数统一错误处理
export function handleError(error: any) {
  const code = error.code || 500
  const message = ERROR_CODES[code] || '系统错误，请稍后重试'
  return { code, message, success: false }
}
```

---

## 6. 测试策略设计

### 6.1 测试层级

| 层级 | 工具 | 覆盖范围 |
|------|------|----------|
| **单元测试** | Vitest | 工具函数、Store逻辑、数据转换 |
| **组件测试** | @vue/test-utils | 组件渲染、交互事件、Props验证 |
| **云函数测试** | 云开发本地调试 | 接口响应、数据库操作、权限校验 |
| **端到端测试** | 微信开发者工具自动化 | 完整用户流程、跨页面交互 |

### 6.2 测试覆盖率目标

| 类型 | 目标 | 说明 |
|------|------|------|
| 核心业务逻辑 | ≥80% | Store、工具函数、云函数 |
| 组件 | ≥60% | 关键交互组件 |
| 云函数 | 100% | 所有云函数接口 |

### 6.3 测试示例

```typescript
// utils/__tests__/format.test.ts
describe('formatDate', () => {
  it('应该正确格式化日期', () => {
    const date = new Date('2026-01-01')
    expect(formatDate(date)).toBe('2026-01-01')
  })
})

// store/__tests__/user.test.ts
describe('UserStore', () => {
  it('登录后应该更新用户信息', async () => {
    const store = useUserStore()
    await store.login()
    expect(store.userInfo).toBeDefined()
    expect(store.isLoggedIn).toBe(true)
  })
})
```

---

## 7. Phase 详细设计

| Phase | 名称 | 主要内容 | 技能标注 | 产出物 |
|-------|------|----------|----------|--------|
| **1** | 项目初始化 | 创建uni-app项目、配置TypeScript、安装依赖 | `[SHELL][WRITE]` | 项目骨架 |
| **2** | 环境配置 | 微信开发者工具、HBuilderX、云开发环境 | `[SHELL][DIALOG]` | 开发环境就绪 |
| **3** | 页面路由 | 配置pages.json、TabBar、页面跳转逻辑 | `[WRITE]` | 路由系统 |
| **4** | 云开发配置 | 创建云开发环境、配置数据库集合、权限设置 | `[DIALOG][WRITE]` | 云环境就绪 |
| **5** | 基础组件 | 封装通用组件（导航栏、列表项、空状态等） | `[GENERATE]` | 组件库 |
| **6** | 首页开发 | Banner、功能入口、推荐列表 | `[GENERATE]` | 首页完成 |
| **7** | 数据模型 | 设计云数据库schema、初始化数据 | `[WRITE][RESEARCH]` | 数据模型 |
| **8** | 用户系统 | 登录授权、用户信息存储、权限管理 | `[GENERATE][DEBUG]` | 用户模块 |
| **9** | 云函数开发 | 核心业务云函数（CRUD、权限校验） | `[GENERATE]` | 云函数集 |
| **10** | 支付集成 | 微信/支付宝支付、订单管理、回调处理 | `[GENERATE][DIALOG]` | 支付模块 |
| **11** | 平台适配 | 条件编译、平台差异处理、样式适配 | `[WRITE][VALIDATE]` | 多端兼容 |
| **12** | 功能完善 | 收藏、评论、分享、搜索等辅助功能 | `[GENERATE]` | 完整功能 |
| **13** | 测试与调试 | 单元测试、真机调试、性能优化 | `[DEBUG][VALIDATE]` | 测试报告 |
| **14** | 审核上架 | 小程序代码审核、隐私协议、资质准备 | `[DIALOG][VALIDATE]` | 提交审核 |
| **15** | 发布管理 | 版本号管理、灰度发布、回滚机制 | `[GIT][WRITE]` | 正式发布 |
| **16** | 运营迭代 | 数据分析、用户反馈、版本更新 | `[REVIEW]` | 运营指南 |

### 7.1 关键Phase说明

- **Phase 4 云开发配置**：需要用户在微信开发者工具手动创建云开发环境，SOP提供引导
- **Phase 10 支付集成**：需要商户资质，SOP提供模拟支付测试方案
- **Phase 14 审核上架**：各平台审核流程不同，SOP提供三端分别指南
- **Phase 11 平台适配**：uni-app条件编译核心，`#ifdef MP-WEIXIN` 等语法

---

## 8. 与现有SOP的关系

```
小程序SOP (独立)
├── 可复用: Git工作流、测试策略、发布管理
├── 可引用: 需求挖掘SOP（Phase 0前置）
└── 独立模块: 云开发、多端编译、小程序审核
```

---

## 9. 后续工作

- [ ] 创建实现计划（writing-plans）
- [ ] 编写小程序SOP文档
- [ ] 创建示例项目模板
- [ ] 编写云函数最佳实践指南
