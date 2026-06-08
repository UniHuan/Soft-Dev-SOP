# Changelog

All notable changes to the SOP Hub will be documented in this file.

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
