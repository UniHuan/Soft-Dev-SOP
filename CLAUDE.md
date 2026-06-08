# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repo contains SOP (Standard Operating Procedure) documentation for fully automated iOS app development using Claude Code + Xcode. The SOP enables Claude Code to build an iOS app from zero to App Store publication in 2 days (16 hours).

## Key Files

- **`iOS_App_2Day_Development_SOP.md`** (5410 lines) — The master SOP with 16 Phases and 64 Steps covering the full lifecycle: environment setup → requirements → high-fidelity prototypes → architecture → data layer → ViewModels → UI → testing → accessibility → App Store assets → IAP → archive → submission.
- **`HIG_KNOWLEDGE_BASE.md`** — Structured summary of Apple's Human Interface Guidelines. Used by Claude Code as a `[RESEARCH]` reference during SOP execution to ensure design compliance.

## How Claude Code Executes the SOP

When a user says "按照 SOP 开发一个 iOS App" or "两天开发一个 App", Claude Code must:

1. Read `iOS_App_2Day_Development_SOP.md` sequentially
2. Execute each Phase in order, following skill annotations (`[SHELL]`, `[WRITE]`, `[GENERATE]`, `[DEBUG]`, etc.)
3. After every `xcodebuild` command, read output and auto-fix errors (`[DEBUG]`) — never pass compile errors to the user
4. Pause at `[DIALOG]` points and wait for user confirmation before continuing
5. Git-commit after each Phase completes

## Skill Annotation System

The SOP uses 9 skill tags throughout. When Claude Code encounters a tag, it switches to the corresponding mode:

| Tag | Mode | Primary Tools |
|-----|------|--------------|
| `[SHELL]` | Execute bash commands | `Bash` |
| `[WRITE]` | Create/modify files | `Edit` |
| `[GENERATE]` | Generate code blocks | `Edit` |
| `[DIALOG]` | Interact with user | Conversation output |
| `[DEBUG]` | Analyze & fix build errors | `Read` + `Bash` + `Edit` |
| `[REVIEW]` | Code quality & HIG audit | `Read` → analysis |
| `[VALIDATE]` | Checklist verification | `Bash` + `Read` + analysis |
| `[RESEARCH]` | Look up HIG/API docs | `Read` HIG_KNOWLEDGE_BASE.md |
| `[GIT]` | Version control | `Bash` (git) |

## SOP Structure

```
Phase 0   — 环境初始化 (环境检查、XcodeGen、项目骨架、Git)
Phase 1   — 产品需求澄清 (问卷 → SPECS.md)
Phase 1.5 — 高保真原型设计 (DESIGN_SPECS.md + SwiftUI Previews)
Phase 2   — 架构搭建 (App入口、Stub视图、Protocol、Info.plist)
Phase 3   — 数据层 (SwiftData Model、StorageService、modelContainer)
Phase 4   — ViewModel (BaseViewModel + 业务ViewModel)
Phase X   — DI容器 (可选，Service≥3时引入)
Phase 5   — UI层 (删除Stub → 按原型1:1实现 + HIG导航/模态/启动)
Phase 6   — 自测 & Day 1收尾 (单元测试、SwiftLint、Day1报告)
Phase 7   — 集成测试 (UI测试、边界情况、无障碍基础扫描)
Phase 8   — 性能 & 无障碍 (WCAG AA、VoiceOver、Dynamic Type、隐私清单)
Phase 9   — App Store素材 (ASC App创建、图标、截图、元数据、文案审查)
Phase 10  — 内购 (StoreKit 2、付费墙、恢复购买)
Phase 11  — Archive & TestFlight (签名、ExportOptions.plist、API Key)
Phase 12  — ASC配置 & 提交审核 (元数据、截图上传、56项HIG清单)
Phase 13  — 归档 (Git tag、README、执行报告、审核防拒、Sign in with Apple)
```

## Critical Execution Rules

1. **Dynamic simulator detection** — All `xcodebuild` commands use `$SIMULATOR_ID` from `/tmp/sop_simulator.env`, never hardcoded device names.
2. **Project variable persistence** — `source /tmp/sop_project.env` at the start of every bash block to access `$PROJECT_NAME`, `$PROJECT_DIR`, `$BUNDLE_ID`.
3. **Phase 2 Stub views** — `ContentView.swift` contains stub `HomeView/DiscoverView/ProfileView` to ensure compile succeeds between Phase 2 and Phase 5. Phase 5 Step 5.0 must delete stubs before creating real views.
4. **Phase 3 modelContainer** — After creating SwiftData models, update the App entry to add `.modelContainer(for:)` on `WindowGroup`.
5. **Phase 5 visual verification** — Build and launch in simulator, capture screenshot for user confirmation before Day 1 ends.
6. **Phase 9 ASC App creation** — Must create the App Record in App Store Connect before Phase 11 can upload builds.

## Maintaining the SOP

When updating the SOP:
- Every Phase header must have a `> [SKILL]` annotation line
- Every `xcodebuild` call must use `$SIMULATOR_ID` from the env file
- Every Phase end must include a compile verification → [DEBUG] → [GIT] flow
- New Phases must be reflected in the timeline diagram at the top and the execution report in Phase 13
- HIG references must be cross-checked against `HIG_KNOWLEDGE_BASE.md`
- Stub views must remain in Phase 2 and be removed in Phase 5 Step 5.0
