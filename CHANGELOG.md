# Changelog

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
