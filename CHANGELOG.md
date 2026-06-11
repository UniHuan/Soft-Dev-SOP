# Changelog

---

## [2.6.0] — 2026-06-11

### Added
- **📖 小说创作 & 短剧制作 SOP** (1,598行) — Claude Code AI 写10万字小说 + 小云雀2.0制作短剧视频，10 Phase 全流程
  - Phase 0: 项目初始化 (目录/元信息/进度追踪/Git)
  - Phase 1-3: 小说规划 (题材定位/三幕结构/人物设计/世界观)
  - Phase 4-6: 逐章创作 (第一幕/第二幕/第三幕，每章自动审查)
  - Phase 7: 全局编辑润色 (一致性审查/文笔润色)
  - Phase 8: 短剧改编 (分集剧本/小云雀导入文件)
  - Phase 9: 小云雀 2.0 视频制作 (AI生成/审核迭代)
  - Phase 10: 发布归档 (交付包/平台指南/数据报告)
- 附录: 命令速查/常见问题速查/短剧剧本格式示例/小云雀提示词指南/发布排期模板

### Changed
- README.md: 新增「创意内容创作」章节
- CLAUDE.md: 新增 creative SOP 条目
- 目录结构: 新增 `creative/` 目录

---

## [2.5.0] — 2026-06-10

### Added
- **小程序 App 2 天开发 SOP 设计文档** (310行) — uni-app + Vue 3 + TypeScript，微信/支付宝/抖音三端，云开发
- **Cocos Creator 游戏开发 SOP 设计文档** (385行) — 5阶段完整流程，微信/抖音/快手小游戏，单机优先
- **sop-test 测试自动化工具设计文档** (422行) — 统一测试CLI，核心框架+平台适配器，支持7平台
- **小程序 SOP 实现计划** (1,051行) — 7 Tasks，16 Phases 详细步骤
- **Cocos Creator 游戏 SOP 实现计划** (533行) — 7 Tasks，5 阶段详细步骤
- **sop-test 工具实现计划** (993行) — 7 Tasks，CLI 开发完整流程

### Changed
- README.md: 新增"规划中"章节、更新项目统计、添加目录结构图
- 项目总行数: 17,900+ → 22,000+

### Directory Structure
```
新增目录:
├── docs/superpowers/
│   ├── specs/          # 设计文档 (3个)
│   └── plans/          # 实现计划 (3个)
```

---

## [2.4.0] — 2026-06-08

### Added
- **ArkTS 代码质量审计** (鸿蒙 SOP Phase 8.2): hvigor lint、大文件检查、硬编码扫描、6 维度清单
- **Swift 代码质量审计** (iOS SOP Step 8.3): SwiftLint、强制解包检查、Keychain、性能规范
- **五平台统一质量门禁**: build+lint+secret+filesize → Phase 9 准入

### Changed
- CLAUDE.md: 完整 Key Files 表 (17 文档) + Android/Web/Backend 平台规则
- README: 行数/Phase/SOP 数更新至实际值

---

## [2.3.0] — 2026-06-08
### Added
- 安全/合规/质量检查嵌入 Android、Web、Backend SOP 的 Phase 8

## [2.2.0] — 2026-06-08
### Added
- Security Best Practices SOP (153行)、API Design SOP (107行)

## [2.1.0] — 2026-06-08
### Added
- Git Workflow、Testing Strategy、Release Management、ASO & Growth (4 份流程 SOP, 558行)

## [2.0.0] — 2026-06-08
### Added
- Web SOP 扩展 (测试/SEO/E2E/Vercel Postgres/后端联调)、CI/CD 扩展 (Web/Backend/Android workflows)

## [1.9.0] — 2026-06-08
### Added
- Android SOP 扩展 (398→675行): Hilt DI、DetailScreen、Play Billing、26项提交清单

## [1.8.0] — 2026-06-08
### Added
- App Operations & Monitoring SOP (339行)

## [1.7.0] — 2026-06-08
### Added
- Backend Service SOP (807行): Hono + Prisma + JWT + Docker

## [1.6.0] — 2026-06-08
### Added
- Web App SOP (571行): Next.js 14+ + Prisma + shadcn/ui + Vercel

## [1.5.0] — 2026-06-08
### Added
- CI/CD SOP (240行) + Android SOP (398行)

## [1.4.0] — 2026-06-08

### Added
- **Phase 1.2: PRD 产品需求文档** — iOS + 鸿蒙 SOP 均新增 PRD 阶段
  - 规格→PRD→原型→开发 四阶段流水线
  - PRD 含 7 章节: 产品概述/功能规格(MoSCoW)/用户流程/数据模型/屏幕规格/非功能性需求/验收标准
  - 屏幕规格(第5章)直接作为 Phase 1.5 原型设计的输入
  - 新增门禁: "PRD 确认" 通过后方可进入原型设计

### Changed
- iOS SOP: 5,410 → 5,590 行 (17 Phase, 64 Step)
- 鸿蒙 SOP: 2,602 → 2,640 行 (16 Phase, 34 Step)
- 项目总行数: 13,009 → 13,227

---

## [1.3.0] — 2026-06-08

### Added
- **`SOP_TEMPLATE.md`** — 可复用的 SOP 编写模板 (161 行，含完整 Phase 结构)
- **`.sop_execution_log_template.md`** — Claude Code 执行追踪日志模板
- **鸿蒙 SOP**: AGC 审核防拒指南 (Phase 13.1)、无障碍自动扫描脚本 (Phase 8.1)
- **CONTRIBUTING**: 完整 Phase 模板 + 新增 SOP 10 项 Checklist + 项目覆盖度表

### Changed
- 鸿蒙 SOP: 2,544 → 2,602 行
- HarmonyOS Dev Guide: 版本号 1.0.0 → 1.1.0
- CLAUDE.md: 新增执行日志追踪规则

---

## [1.2.0] — 2026-06-08

### Added
- **`QUICK_START.md`** — 30 秒场景决策 + 一句话触发词
- **鸿蒙 SOP**: 导航模式(5.3)、模态&反馈(5.4)、引导页&空状态(5.5)、单元测试、付费墙UI、无障碍代码示例、可选 Phase 跳过指南

### Changed
- 鸿蒙 SOP: 2,127 → 2,544 行
- Phase 0.2: 新增 app.json5 模板

---

## [1.1.0] — 2026-06-08

### Added
- **鸿蒙 SOP**: NetworkService、DI Container、无障碍验证、华为 IAP、24项AGC清单、原型映射、华为账号登录、元数据自动生成、FAQ(8问)、自测(10项)
- **Pipeline**: 鸿蒙→软著流水线覆盖
- **CONTRIBUTING.md** + **CHANGELOG.md**
- **Git tag v1.0.0**

### Changed
- 鸿蒙 SOP: 1,147 → **2,127** 行
- Pipeline: 709 → **748** 行
- 项目总行数: **12,177**
- 项目文件按平台分目录 (ios/harmonyos/copyright/pipeline)
- `hvigorw` → `./hvigorw` (29处)
- EntryAbility Step 3.4: 片段 → 完整文件

### Fixed
- Pipeline 交叉引用路径 (8处 iOS + 4处 软著)
- Step 7.2 模糊编辑指令 → 精确替换
- LongPressGesture 位置 (ListItem 而非 TaskCard)

---

## [1.0.0] — 2026-06-08

### Added
- **`ios/iOS_App_2Day_Development_SOP.md`** — iOS App 2 天全自动开发 SOP (5,410 行, 16 Phase, 60 Step)
- **`ios/HIG_KNOWLEDGE_BASE.md`** — Apple Human Interface Guidelines 知识库 (493 行)
- **`harmonyos/HarmonyOS_App_2Day_Development_SOP.md`** — 鸿蒙 App 2 天全自动开发 SOP (1,702 行, 13 Phase)
- **`harmonyos/HarmonyOS_Development_Guide.md`** — 鸿蒙 ArkTS/ArkUI/Stage 模型开发规范 (1,883 行)
- **`copyright/Software_Copyright_Application_SOP.md`** — 软件著作权申请全流程 SOP (1,269 行, 8 Phase)
- **`pipeline/App_Development_to_Copyright_Full_Pipeline.md`** — iOS 开发→上架→软著一体化流水线 (709 行)
- **`CLAUDE.md`** — Claude Code 项目指引 (74 行)
- **`README.md`** — 项目导航与快速开始 (84 行)

### Design
- 9 种技能标注体系 (`[SHELL]` `[WRITE]` `[GENERATE]` `[DEBUG]` `[DIALOG]` `[REVIEW]` `[VALIDATE]` `[RESEARCH]` `[GIT]`)
- 编译验证门禁模式 (每个 Phase 结束自动检查 + [DEBUG] 自动修复循环)
- 增量 Git 提交策略 (每 Phase 一个 commit)
- 双平台覆盖 (iOS + HarmonyOS) + 软著 + 流水线

### Directory Structure
```
SOP/
├── ios/         # iOS 开发
├── harmonyos/   # 鸿蒙开发
├── copyright/   # 软著
├── pipeline/    # 流水线
├── README.md
├── CLAUDE.md
└── CHANGELOG.md
```
