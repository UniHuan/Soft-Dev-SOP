# 📊 应用运营 & 监控 SOP — 全平台覆盖

> **适用对象**: Claude Code (AI Agent) + 用户协作
> **目标**: App 上架后的持续运营：崩溃监控、数据分析、评分管理、版本迭代
> **覆盖平台**: iOS · Android · HarmonyOS · Web
> **前置条件**: App 已上架 (完成对应平台 SOP 的 Phase 13)

---

## 🧠 技能调用

| 技能 | 说明 |
|------|------|
| `[SHELL]` | SDK 集成命令 |
| `[WRITE]` | 配置文件、隐私政策 |
| `[DIALOG]` | 确认运营策略 |
| `[VALIDATE]` | 验证监控数据 |

---

## 📋 运营全生命周期

```
上架后 Day 1-7     上架后 Week 2-4     持续运营
├─ 崩溃监控接入     ├─ 数据分析        ├─ 版本迭代
├─ 分析 SDK 集成    ├─ 用户反馈收集    ├─ ASO 优化
├─ 评分引导         ├─ 首版数据复盘    ├─ 用户增长
└─ 隐私合规检查     └─ 紧急修复流程    └─ 变现优化
```

---

## Phase 1: 崩溃 & 性能监控 (Day 1)

### iOS — Firebase Crashlytics

```swift
// [WRITE] 通过 SPM 添加: https://github.com/firebase/firebase-ios-sdk
// AppDelegate.swift 或 ${PROJECT_NAME}App.swift
import FirebaseCore
import FirebaseCrashlytics

// 在 App init 中:
FirebaseApp.configure()
Crashlytics.crashlytics().setCrashlyticsCollectionEnabled(true)
```

> GoogleService-Info.plist 从 Firebase Console 下载 → 拖入 Xcode 项目

### Android — Firebase Crashlytics

```kotlin
// [WRITE] app/build.gradle.kts
dependencies {
    implementation(platform("com.google.firebase:firebase-bom:33.0.0"))
    implementation("com.google.firebase:firebase-crashlytics-ktx")
    implementation("com.google.firebase:firebase-analytics-ktx")
}
// google-services.json 从 Firebase Console → app/ 目录
```

### HarmonyOS — AGC 崩溃服务

```json5
// [WRITE] entry/src/main/module.json5 — 已在 Phase 11 涉及
// AGC → 质量 → 崩溃 → 自动采集
// 无需额外 SDK, AGC 内置崩溃监控
```

### Web — Sentry

```bash
# [SHELL]
pnpm add @sentry/nextjs    # Next.js
# 或
pnpm add @sentry/react     # React

# 自动配置:
npx @sentry/wizard -i nextjs
```

---

## Phase 2: 数据分析 (Day 1-2)

### 核心指标定义

```
必采指标 (所有平台):
□ DAU (日活跃用户) / MAU (月活跃用户)
□ 留存率 (Day 1 / Day 7 / Day 30)
□ 会话时长 (平均使用时长)
□ 页面浏览 (每个屏幕的 PV)
□ 转化率 (注册率 / 付费率)
□ 崩溃率 (崩溃用户占比 < 1%)

关键事件 (按功能定义):
□ 完成核心操作 (如 "任务创建")
□ 触发付费 (如 "点击订阅按钮")
□ 分享行为 (如 "分享任务")
```

### iOS — Firebase Analytics

```swift
// 自动采集: 页面浏览、首次打开、应用内购买
// 自定义事件:
Analytics.logEvent("task_created", parameters: [
    "priority": task.priority,
    "has_due_date": task.dueDate != nil
])
```

### Android — Firebase Analytics

```kotlin
// 自动采集 + 自定义事件
firebaseAnalytics.logEvent("task_created") {
    param("priority", task.priority.toString())
    param("has_due_date", (task.dueDate > 0).toString())
}
```

### HarmonyOS — AGC 分析

```typescript
// 自动采集 (AGC 分析服务)

// 自定义事件 (通过 AGC SDK)
import { analytics } from '@kit.AnalyticsKit'
analytics.onEvent('task_created', { priority: task.priority })
```

### Web — Plausible / Umami (隐私友好)

```html
<!-- [WRITE] 在 app/layout.tsx <head> 中 -->
<script defer data-domain="yourdomain.com" 
  src="https://plausible.io/js/script.js"></script>
```

---

## Phase 3: 评分 & 评价管理 (Day 2-3)

### 评分引导策略

```
✅ 推荐时机 (在用户最满意时引导):
  - 完成第 5 个任务后 → 弹出评分引导
  - 连续使用 7 天后 → 弹出评分引导
  - 完成付费后 → 弹出评分引导

❌ 避免时机 (会被差评):
  - App 刚打开就弹 (打断用户)
  - 崩溃后弹 (用户正在气头上)
  - 频繁弹 (每周 > 1 次)
```

### iOS — SKStoreReviewController

```swift
import StoreKit

func requestReview() {
    // App Store 限制: 每年最多弹 3 次, 系统自动控制频率
    if let scene = UIApplication.shared.connectedScenes.first as? UIWindowScene {
        SKStoreReviewController.requestReview(in: scene)
    }
}
```

### Android — In-App Review API

```kotlin
// Play Core library
val manager = ReviewManagerFactory.create(context)
val request = manager.requestReviewFlow()
request.addOnCompleteListener { task ->
    if (task.isSuccessful) {
        manager.launchReviewFlow(activity, task.result)
    }
}
```

### 差评处理

```
□ 每天检查 App Store Connect / Google Play Console 评价
□ 24 小时内回复差评 (公开回复, 展示服务态度)
□ 根据反馈修复关键问题 → 发布更新 → 联系用户告知已修复
□ 回复模板:
  "感谢反馈! 我们已记录此问题, 将在下个版本修复。
   如有更多建议, 请联系 support@example.com"
```

---

## Phase 4: 用户反馈 & 迭代 (持续)

### 反馈渠道搭建

```
□ 应用内反馈入口 (设置页 → "意见反馈")
□ 支持邮箱 (support@example.com)
□ 应用商店评价回复
□ 社交媒体监控 (微博/小红书/知乎)
```

### 版本迭代节奏

```
热修复 (紧急):
  - 崩溃率 > 1% → 24 小时内修复 → 发布
  - 严重功能不可用 → 24 小时内修复 → 发布

小版本 (2-4 周):
  - Bug 修复 + 小优化
  - versionName: 1.0.1, 1.0.2...

中版本 (1-2 月):
  - 新功能 + 体验优化
  - versionName: 1.1.0, 1.2.0...

大版本 (3-6 月):
  - 重大功能 + UI 改版
  - versionName: 2.0.0
```

---

## Phase 5: 隐私合规 (上线前)

### 隐私政策模板

```markdown
# ${App名称} 隐私政策

**最后更新**: yyyy-mm-dd

## 1. 信息收集
我们收集以下信息以提供服务:
- 账号信息: 邮箱 (用于登录)
- 使用数据: 功能使用频率 (用于优化体验)
- 设备信息: 设备型号、系统版本 (用于兼容性分析)

## 2. 信息使用
- 提供核心功能服务
- 改进产品体验
- 不用于广告定向
- 不与第三方共享个人数据

## 3. 数据存储与安全
- 数据存储于 [服务器位置]
- 传输使用 HTTPS 加密
- 密码使用 bcrypt 哈希存储

## 4. 用户权利
- 可随时删除账号及关联数据
- 可导出个人数据
- 可撤回授权

## 5. 联系我们
- 邮箱: support@example.com
```

### 各平台合规要求

```
iOS (App Store):
  □ PrivacyInfo.xcprivacy (2024+ 强制)
  □ App Store Connect 隐私标签
  □ ATT 弹窗 (如使用广告标识符)

Android (Google Play):
  □ Data safety section (2022+ 强制)
  □ 敏感权限声明

HarmonyOS (AppGallery):
  □ AGC 隐私声明
  □ 最小权限原则

Web:
  □ GDPR 合规 (欧盟用户)
  □ Cookie 同意弹窗
  □ CCPA 合规 (加州用户)
```

---

## Phase 6: 变现策略 (可选)

```
变现模式选择:
├── 免费 + 广告 → AdMob (iOS/Android)
├── 免费 + 内购 → StoreKit / Play Billing / Huawei IAP
├── 订阅制 → Auto-Renewable Subscription
├── 付费下载 → 一次购买
└── 企业服务 → B2B API 付费

指标监控:
□ ARPU (每用户平均收入)
□ LTV (用户生命周期价值)
□ 付费转化率
□ 退款率 (< 2%)
```

---

## 📚 附录

### A. 运营工具推荐

| 工具 | 用途 | 平台 | 费用 |
|------|------|------|------|
| Firebase | 崩溃+分析 | iOS/Android/Web | 免费 |
| Sentry | 错误追踪 | 全平台 | 免费层 5K events |
| AGC 分析 | 数据+崩溃 | HarmonyOS | 免费 |
| Plausible | Web 分析 | Web | 自建免费 |
| AppFollow | 评价管理 | iOS/Android | 免费层 |
| UptimeRobot | 可用性监控 | 后端 | 免费 50 monitors |

### B. 运营指标看板

```
┌─────────────────────────────────────────┐
│              运营仪表板                    │
├─────────────────────────────────────────┤
│ DAU: 1,234 ↑12%       崩溃率: 0.3% ✅   │
│ MAU: 8,901 ↑5%        评分: 4.7 ★★★★★  │
│ 留存 D7: 45%          付费率: 3.2%       │
│ 会话时长: 8.5min       ARPU: ¥2.5        │
└─────────────────────────────────────────┘
```

---

> **SOP 版本**: 1.0.0 | **最后更新**: 2026-06-08
> **适用**: 所有已上架 App 的持续运营
