# 📖 SOP Hub 使用手册

> **适用**: 所有开发者、产品经理、Claude Code 用户
> **目标**: 30 分钟内掌握如何用本项目完成软件产品全生命周期

---

## 🚀 5 分钟快速上手

### 第一步: 告诉 Claude Code 你想做什么

```
"帮我挖掘未来半年软件需求"              → 需求调研
"按照 SOP 开发一个 iOS App"            → iOS 开发
"帮我从开发到上架再到软著全流程"        → 完整流水线
"帮我做变现规划"                       → 变现优化
```

### 第二步: Claude Code 自动执行

```
Claude Code 读取对应 SOP
    ↓
逐 Phase 执行 (自动编译检查)
    ↓
遇到 [DIALOG] 暂停 → 等待你确认
    ↓
每 Phase 结束自动 Git 提交
    ↓
完成后输出报告
```

---

## 📂 项目结构速览

```
SOP/
├── ios/           ← iOS 开发 (SwiftUI+Xcode)
├── android/       ← Android 开发 (Kotlin+Compose)
├── harmonyos/     ← 鸿蒙开发 (ArkTS+DevEco)
├── web/           ← Web 开发 (Next.js+Prisma)
├── backend/       ← 后端开发 (Hono+JWT+Docker)
├── cicd/          ← CI/CD 自动化
├── copyright/     ← 软件著作权申请
├── pipeline/      ← 开发→上架→软著 流水线
├── operations/    ← 运营监控
├── process/       ← 流程规范 (需求/变现/测试/发布/...)
│
├── README.md          ← 项目概览
├── QUICK_START.md     ← 30 秒场景决策
├── CLAUDE.md          ← Claude Code 执行规则
├── CONTRIBUTING.md    ← 贡献指南
├── SOP_TEMPLATE.md    ← 创建新 SOP 模板
└── USER_MANUAL.md     ← 本文件
```

---

## 🎯 典型使用场景

### 场景 1: 我有一个想法，不知道值不值得做

```
1. 对 Claude Code 说: "帮我挖掘软件需求"
2. Claude Code 执行 Requirement_Discovery_Research_SOP
   → 深度访谈了解你的背景/资源/兴趣
   → 分析市场趋势 + 竞品
   → 输出 3 套完整方案 + SWOT + ROI 对比
   → 推荐最优方案
3. 你的决策: 选定方案 → 进入场景 2
```

### 场景 2: 我要从零开发一个 App 并上架

```
1. 对 Claude Code 说: "按照 SOP 开发一个 iOS App"
   (或 Android / 鸿蒙 / Web / 后端)
2. Claude Code 执行 13-17 个 Phase:
   Day 1: 需求→PRD→原型→架构→数据→ViewModel→UI→自测
   Day 2: 测试→安全审计→素材→构建→签名→提交审核
3. 你只需在 [DIALOG] 标记处确认关键决策
4. 每个 Phase 自动编译验证 + Git 提交
5. 产出: 可上架的 App + 完整代码仓库
```

### 场景 3: 我要把 App 的月收入从 1000 提升到 5000

```
1. 对 Claude Code 说: "帮我做变现规划"
2. Claude Code 执行 Product_Monetization_SOP
   → 竞品定价分析
   → 转化漏斗诊断
   → A/B 测试计划
   → 月度定价实验
3. 配合 Operations SOP (监控数据) + ASO SOP (增长)
```

### 场景 4: App 上架了，我要申请软著

```
1. 对 Claude Code 说: "帮我申请软件著作权"
2. Claude Code 执行 Copyright SOP
   → 自动从项目提取源代码
   → 生成 PDF (页眉/页码/宋体五号)
   → 生成用户手册 (复用 App Store 截图)
   → 在线提交指导
3. 等待 ~30 工作日 → 领取电子证书
```

### 场景 5: 全流程一站式

```
对 Claude Code 说: "从开发到上架到软著全流程"
→ Claude Code 按 Pipeline 执行:
   Part 1: iOS 开发 & 上架 (2 天)
   Part 2: 软著申请 (上架后)
   (中间材料自动复用, 省 68% 时间)
```

---

## 🤖 Claude Code 执行机制

### 技能标注系统

每个 SOP 的每个 Phase/Step 都标注了技能标签:

| 标签 | 含义 | Claude Code 做什么 |
|------|------|-------------------|
| `[SHELL]` | 执行命令 | 运行 bash、xcodebuild、./hvigorw、git |
| `[WRITE]` | 创建文件 | Edit 工具写代码/配置/文档 |
| `[GENERATE]` | 生成代码 | 生成业务逻辑/UI/测试代码 |
| `[DIALOG]` | 用户确认 | **暂停等用户回复后再继续** |
| `[DEBUG]` | 修复错误 | 编译失败 → 读日志 → 修复 → 重编译 |
| `[VALIDATE]` | 验证清单 | 逐项检查是否满足条件 |
| `[RESEARCH]` | 查阅规范 | 读参考文档 (HIG KB/HOS Guide) |
| `[GIT]` | 版本控制 | git add + git commit |

### 编译验证循环

```
每次代码写入后:
  → 运行构建命令
  → 成功 ✅ → 继续下一步
  → 失败 ❌ → [DEBUG] 自动分析错误
            → 定位文件+行号
            → Edit 修复
            → 重编译
            → (最多循环 5 次)
            → 仍失败 → [DIALOG] 报告用户
```

### 用户确认点

```
Claude Code 在以下情况暂停:
□ [DIALOG] 标记处 (如需求确认/PRD 确认/原型确认)
□ 不可逆操作 (如删除数据/提交审核/付费)
□ 编译失败 5 次仍无法修复
□ 需要用户手动操作 (如 App Store Connect/AGC 配置)
```

---

## 📊 选择正确的 SOP

### 按产品类型选

| 产品类型 | 推荐 SOP |
|---------|---------|
| iOS App | `ios/iOS_App_2Day_Development_SOP.md` |
| Android App | `android/Android_App_2Day_Development_SOP.md` |
| 鸿蒙 App | `harmonyos/HarmonyOS_App_2Day_Development_SOP.md` |
| Web 应用 (全栈) | `web/Web_App_2Day_Development_SOP.md` + `backend/Backend_Service_2Day_Development_SOP.md` |
| 纯后端 API | `backend/Backend_Service_2Day_Development_SOP.md` |
| 小程序 | `harmonyos/HarmonyOS_App_2Day_Development_SOP.md` (鸿蒙) 或 Web 技术栈 |

### 按阶段选

| 当前阶段 | 推荐 SOP |
|---------|---------|
| 还没有想法 | `process/Requirement_Discovery_Research_SOP.md` |
| 有想法要验证 | 对应平台 SOP → Phase 1 (需求确认) + Phase 1.2 (PRD) |
| 开始开发 | 对应平台 SOP → Phase 2-13 |
| 已上架要运营 | `operations/App_Operations_SOP.md` |
| 要提升收入 | `process/Product_Monetization_SOP.md` |
| 要提升下载 | `process/ASO_Growth_SOP.md` |
| 要申请软著 | `copyright/Software_Copyright_Application_SOP.md` |
| 团队协作 | `process/Git_Workflow_Code_Review_SOP.md` |
| 要写测试 | `process/Testing_Strategy_SOP.md` |
| 要发布版本 | `process/Release_Management_SOP.md` |
| 要做 CI/CD | `cicd/CI_CD_Integration_SOP.md` |

---

## ⚠️ 常见问题

### Q: Claude Code 执行到一半停了怎么办?

```
A: 检查是否是 [DIALOG] 标记:
  - 如果是 → 回复你的决定, Claude Code 继续
  - 如果是编译错误 → 检查错误信息, 回复 "继续修复"
  - 如果是卡住没有任何输出 → 回复 "继续执行 Phase N"
  - Claude Code 可以从任意 Phase 恢复执行
```

### Q: 我想跳过某个 Phase

```
A: 直接告诉 Claude Code: "跳过 Phase 3, 从 Phase 4 继续"
   Claude Code 会跳过但提醒你可能缺少依赖。
   
   可选 Phase (明确标注):
   - Phase 3.5 NetworkService (纯本地 App 可跳过)
   - Phase 3.6 AuthService (不需要登录可跳过)
   - Phase 4.1 DI 容器 (简单 App 可跳过)
   - Phase 10.3 IAP (免费 App 可跳过)
```

### Q: 编译一直失败怎么办?

```
A: 1. Claude Code 会自动尝试修复 5 次
   2. 5 次失败后会报告具体错误文件和行号
   3. 你可以:
      - 手动打开文件检查
      - 回复 "用另一种方式实现"
      - 跳过并后续手动修复
   4. 常见原因: API 版本不匹配 / 缺少依赖 / 类型错误
```

### Q: 我想用不同的技术栈

```
A: SOP 是模板化的, Claude Code 可以适配:
  - "用 UIKit 代替 SwiftUI 开发" → Claude Code 调整代码风格
  - "用 Express 代替 Hono" → 调整后端框架
  - "用 Vue 代替 React" → 调整前端框架
  但建议先用推荐技术栈跑通, 后续再替换。
```

### Q: 多个 SOP 之间怎么衔接?

```
A: SOP 之间有标准衔接点:
  需求调研 SOP → 各平台 SOP Phase 1 (SPECS.md 可复用)
  各平台 SOP Phase 13 → 软著 SOP (源代码/截图可复用)
  各平台 SOP Phase 12 → Operations SOP (上架后运营)
  各平台 SOP Phase 12 → Monetization SOP (变现规划)
  Pipeline SOP 提供一键自动化衔接脚本
```

---

## 🔧 高级用法

### 自定义 SOP

```bash
# 1. 复制模板
cp SOP_TEMPLATE.md my_custom_sop.md

# 2. 按模板编写
# - 技能标注 ([SHELL] [WRITE] ...)
# - Phase 结构 (Phase 0 → Phase N)
# - 编译验证 (每 Phase 结束)
# - [DIALOG] 确认点

# 3. 更新 CLAUDE.md + README.md 引用
```

### 批量执行

```
"按照 SOP 开发 iOS App, 完成后自动执行 CI/CD SOP"
→ Claude Code 依次执行两个 SOP
```

### 增量开发

```
"从 Phase 5 继续执行 iOS SOP"
→ Claude Code 读取 DAY1_REPORT.md 了解进度
→ 从 Phase 5 开始继续
```

---

## 📋 执行检查清单

```
开始前:
□ 安装了对应平台工具 (Xcode/DevEco/Android Studio/Node.js)
□ 注册了对应开发者账号
□ Git 已配置

执行中:
□ 每个 [DIALOG] 及时回复
□ 每个 Phase 结束检查 Git 提交记录
□ 遇到编译错误让 Claude Code 自动修复

完成后:
□ App 已上架 (或 API 已部署/软著已提交)
□ Git 仓库包含完整提交历史
□ 文档已归档 (SPECS/PRD/DESIGN_SPECS/Day1 报告)
```

---

> **手册版本**: 1.0.0 | **最后更新**: 2026-06-08
> **项目主页**: [README.md](README.md) | **快速决策**: [QUICK_START.md](QUICK_START.md)
