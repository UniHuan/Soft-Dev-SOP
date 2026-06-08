# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repo contains SOP (Standard Operating Procedure) documentation for fully automated app development using Claude Code + IDE. Covers iOS (SwiftUI+Xcode), HarmonyOS (ArkTS+DevEco Studio), and software copyright application.

## Key Files

| File | Lines | Purpose |
|------|-------|---------|
| `ios/iOS_App_2Day_Development_SOP.md` | 5,410 | iOS App 2-day dev SOP (SwiftUI+Xcode, 16 Phases) |
| `ios/HIG_KNOWLEDGE_BASE.md` | 493 | Apple HIG knowledge base for `[RESEARCH]` |
| `harmonyos/HarmonyOS_App_2Day_Development_SOP.md` | 1,702 | HarmonyOS App 2-day dev SOP (ArkTS+DevEco, 13 Phases) |
| `harmonyos/HarmonyOS_Development_Guide.md` | 1,883 | HarmonyOS dev spec reference for `[RESEARCH]` |
| `copyright/Software_Copyright_Application_SOP.md` | 1,269 | Software copyright application SOP (8 Phases) |
| `pipeline/App_Development_to_Copyright_Full_Pipeline.md` | 709 | Combined iOS dev→App Store→copyright pipeline |

## Skill Annotation System

All SOPs use 9 skill tags. When Claude Code encounters a tag, switch to the corresponding mode:

| Tag | Mode | Tools |
|-----|------|-------|
| `[SHELL]` | Execute bash commands | `Bash` |
| `[WRITE]` | Create/modify files | `Edit` |
| `[GENERATE]` | Generate code | `Edit` |
| `[DIALOG]` | Interact with user | Conversation |
| `[DEBUG]` | Analyze & fix build errors | `Read` + `Bash` + `Edit` |
| `[REVIEW]` | Code quality & spec audit | `Read` → analysis |
| `[VALIDATE]` | Checklist verification | `Bash` + `Read` |
| `[RESEARCH]` | Look up specs/API docs | `Read` reference docs |
| `[GIT]` | Version control | `Bash` (git) |

## SOP Execution Rules (all platforms)

1. Read the target SOP sequentially, executing each Phase in order
2. After every build command (`xcodebuild`, `./hvigorw`), read output and auto-fix errors (`[DEBUG]`) — never pass compile errors to user
3. Pause at `[DIALOG]` points, wait for user confirmation
4. Git-commit after each Phase completes
5. **Track execution**: Copy `.sop_execution_log_template.md` to `SOP_EXECUTION_LOG.md` and update after each Phase

## Platform-Specific Rules

### iOS SOP (`ios/iOS_App_2Day_Development_SOP.md`)
- `source /tmp/sop_project.env` before every bash block
- `source /tmp/sop_simulator.env` for `$SIMULATOR_ID`
- Phase 2 stub views: `HomeView/DiscoverView/ProfileView` are stubs, deleted at Phase 5 Step 5.0
- Phase 3: add `.modelContainer(for:)` to App entry after SwiftData models
- Phase 5: visual verification via `xcrun simctl io screenshot`
- Phase 11: `ExportOptions.plist` required; ASC API Key guide if env vars missing

### HarmonyOS SOP (`harmonyos/HarmonyOS_App_2Day_Development_SOP.md`)
- `source /tmp/sop_harmony.env` before every bash block
- ALL build commands use `./hvigorw` (NOT `hvigorw` — it's a project-local script)
- Phase 3.4: replace entire `EntryAbility.ets` (not partial edit)
- Phase 7.2: precise search-replace edits (marked "替换 1", "替换 2")
- Phase 8: LongPressGesture on `ListItem()`, not on `TaskCard`
- Phase 10: signing requires user to generate .p12/.csr via DevEco Studio GUI

### Copyright SOP (`copyright/Software_Copyright_Application_SOP.md`)
- Phase 2: source code extraction uses `while IFS= read -r` (not `for f in $var`)
- Phase 2.3: Python `reportlab` for PDF; manual Word/Pages fallback
- Phase 5: five-dimension validation (一致性/格式/内容/签章/交叉)
- Phase 6: ASC account real-name verification must complete first

## Maintaining SOPs

When updating any SOP:
- Every Phase header must have a `> [SKILL]` annotation line
- Every build call must use the correct tool path (`xcodebuild`, `./hvigorw`)
- Every Phase end must include: compile verification → `[DEBUG]` → `[GIT]`
- Reference docs (`ios/HIG_KNOWLEDGE_BASE.md`, `harmonyos/HarmonyOS_Development_Guide.md`) must stay in sync
- Update `README.md` line counts if adding/removing content
