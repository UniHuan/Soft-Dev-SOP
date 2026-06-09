# SOP Hub — 全自动 App 开发标准操作流程

> Claude Code + IDE 从零到上架的完整自动化流水线

→ **[🚀 30 秒快速开始](QUICK_START.md)** ← 不知道该用哪个 SOP？先看这里。

---

## 📂 文档导航

### iOS 开发

| 文档 | 说明 | 行数 |
|------|------|------|
| [iOS App 2 天开发 SOP](ios/iOS_App_2Day_Development_SOP.md) | SwiftUI + Xcode，16 Phase 全自动执行 | 5,410 |
| [HIG 知识库](ios/HIG_KNOWLEDGE_BASE.md) | Apple Human Interface Guidelines 结构化总结 | 493 |

### 鸿蒙开发

| 文档 | 说明 | 行数 |
|------|------|------|
| [鸿蒙 App 2 天开发 SOP](harmonyos/HarmonyOS_App_2Day_Development_SOP.md) | ArkTS + DevEco Studio，13 Phase 全自动执行 | 2,449 |
| [鸿蒙开发规范](harmonyos/HarmonyOS_Development_Guide.md) | ArkTS/ArkUI/Stage 模型完整规范 + iOS 概念映射 | 1,883 |

### Android 开发

| 文档 | 说明 | 行数 |
|------|------|------|
| [Android App 2 天开发 SOP](android/Android_App_2Day_Development_SOP.md) | Kotlin + Jetpack Compose，14 Phase 全自动执行 | 398 |

### Web 开发

| 文档 | 说明 | 行数 |
|------|------|------|
| [Web App 2 天开发 SOP](web/Web_App_2Day_Development_SOP.md) | Next.js 14+ + TypeScript + Prisma + Tailwind，前端 | 571 |
| [后端服务 2 天开发 SOP](backend/Backend_Service_2Day_Development_SOP.md) | Hono + Prisma + JWT + Docker，前后端联调 | 807 |

### CI/CD

| 文档 | 说明 | 行数 |
|------|------|------|
| [CI/CD 集成 SOP](cicd/CI_CD_Integration_SOP.md) | GitHub Actions + hvigor CI 双平台自动化 | 240 |

### 开发流程 & 质量

| 文档 | 说明 | 行数 |
|------|------|------|
| [Git 工作流 & 代码审查](process/Git_Workflow_Code_Review_SOP.md) | 分支策略、Conventional Commits、PR 审查清单 | 182 |
| [测试策略](process/Testing_Strategy_SOP.md) | 测试金字塔、覆盖率目标、Bug 报告模板 | 137 |
| [发布管理](process/Release_Management_SOP.md) | 语义化版本、发布检查清单、热修复流程 | 108 |
| [ASO & 用户增长](process/ASO_Growth_SOP.md) | 关键词优化、视觉优化、评分管理、增长渠道 | 131 |

### 软著 & 全流程

| 文档 | 说明 | 行数 |
|------|------|------|
| [软著申请 SOP](copyright/Software_Copyright_Application_SOP.md) | 从材料准备到领证，8 Phase | 1,269 |
| [开发→上架→软著流水线](pipeline/App_Development_to_Copyright_Full_Pipeline.md) | iOS 开发与软著申请交叉复用指南 | 709 |

---

## 🚀 快速开始

```bash
# 克隆仓库
git clone https://github.com/UniHuan/IOS-App-Dev-SOP.git
cd IOS-App-Dev-SOP

# 告诉 Claude Code 你想做什么：
# "按照 SOP 开发一个 iOS App"
# "按照 SOP 开发一个鸿蒙 App"
# "帮我申请软件著作权"
```

Claude Code 会读取对应的 SOP 文件，按 Phase 顺序全自动执行。

---

## 🧠 SOP 设计理念

```
每条 SOP 包含:
├── 技能标注 ([SHELL] [WRITE] [GENERATE] [DEBUG] [DIALOG] [REVIEW] [VALIDATE] [RESEARCH] [GIT])
├── 编译验证门禁 (每个 Phase 结束自动检查)
├── [DEBUG] 自动修复循环 (编译失败 → 读日志 → 修复 → 重试)
├── [DIALOG] 用户确认点 (不可逆操作前暂停)
└── Git 增量提交 (每 Phase 一个 commit，可随时回滚)
```

---

## 📊 项目统计

| 指标 | 数值 |
|------|------|
| SOP 文档数 | 12 |
| 总行数 | 14,600+ |
| Phase/Step 总数 | 80+ Phase，200+ Step |
| 技能标注密度 | 每个 Step 1-3 个技能标签 |
| 编译验证点 | 30+ 处 |
| 覆盖平台 | iOS、Android、HarmonyOS、Web、后端、软著、CI/CD、运营、流程 |

---

## 🔗 相关链接

- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
- [华为开发者联盟](https://developer.huawei.com/consumer/cn/)
- [中国版权保护中心](https://www.ccopyright.com.cn)
- [HarmonyOS 开发文档](https://developer.huawei.com/consumer/cn/doc/)

---

> **维护**: 持续审查中 | **最后更新**: 2026-06-08
