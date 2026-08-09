# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repo contains SOP (Standard Operating Procedure) documentation for fully automated app development using Claude Code + IDE, plus creative content workflows (AI novel writing + short drama video production). Covers iOS (SwiftUI+Xcode), HarmonyOS (ArkTS+DevEco Studio), Android (Kotlin+Compose), Web (Next.js), Backend (Hono), Fullstack multi-platform (Huawei Cloud), software copyright, and creative content.

## Key Files

| File | Lines | Purpose |
|------|-------|---------|
| **Platform SOPs** | | |
| `ios/iOS_App_2Day_Development_SOP.md` | 5,665 | iOS (SwiftUI+Xcode) — 17 Phases |
| `harmonyos/HarmonyOS_App_2Day_Development_SOP.md` | 2,754 | HarmonyOS (ArkTS+DevEco) — 16 Phases |
| `android/Android_App_2Day_Development_SOP.md` | 731 | Android (Kotlin+Compose) — 14 Phases |
| `web/Web_App_2Day_Development_SOP.md` | 787 | Web (Next.js+Prisma) — 9 Phases |
| `backend/Backend_Service_2Day_Development_SOP.md` | 878 | Backend (Hono+Prisma+JWT) — 6 Phases |
| `fullstack/Fullstack_MultiPlatform_Product_SOP.md` | 7,000+ | Fullstack (Backend+CMS+Web+iOS+Android+Huawei Cloud+Ops+Audit+Ecosystem) — 21 Phases |
| **Specialty SOPs** | | |
| `copyright/Software_Copyright_Application_SOP.md` | 1,269 | Software copyright — 8 Phases |
| `pipeline/App_Development_to_Copyright_Full_Pipeline.md` | 917 | Dev→App Store→Copyright pipeline |
| `cicd/CI_CD_Integration_SOP.md` | 295 | CI/CD (GitHub Actions+hvigor) |
| `operations/App_Operations_SOP.md` | 339 | Operations & monitoring |
| **Process SOPs** | | |
| `process/Git_Workflow_Code_Review_SOP.md` | 182 | Git flow + PR reviews |
| `process/Testing_Strategy_SOP.md` | 137 | Test pyramid + coverage targets |
| `process/Release_Management_SOP.md` | 108 | Semantic versioning + hotfix |
| `process/ASO_Growth_SOP.md` | 131 | App Store Optimization |
| `process/API_Design_SOP.md` | 107 | RESTful API standards |
| `process/Security_Best_Practices_SOP.md` | 153 | OWASP + encryption + auth |
| `creative/Novel_to_Short_Drama_SOP.md` | 1,598 | Novel writing + short drama video — 10 Phases |
| **Reference** | | |
| `ios/HIG_KNOWLEDGE_BASE.md` | 493 | Apple HIG for `[RESEARCH]` |
| `harmonyos/HarmonyOS_Development_Guide.md` | 1,883 | HarmonyOS dev specs for `[RESEARCH]` |

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

### Android SOP (`android/Android_App_2Day_Development_SOP.md`)
- `source /tmp/sop_android.env` before every bash block
- Build: `./gradlew assembleDebug` (NOT `./hvigorw`)
- Phase 2: Hilt DI requires `@HiltAndroidApp` Application + `@Module @InstallIn`
- Phase 8: Security checks — EncryptedSharedPreferences, R8 `isMinifyEnabled`, BuildConfig
- Phase 11: `./gradlew bundleRelease` for Google Play (`.aab`), NOT APK
- Phase 12: 26-item Google Play submission checklist

### Web SOP (`web/Web_App_2Day_Development_SOP.md`)
- Build: `pnpm build` + `pnpm tsc --noEmit` (dual check)
- Phase 2: `npx prisma migrate dev` before any data operations
- Phase 6: Security — `pnpm audit`, CSP headers, `.env` gitignored
- Phase 8: Vercel auto-deploys on `git push main`
- Phase 9: Backend integration via `fetchAPI()` with JWT token

### Backend SOP (`backend/Backend_Service_2Day_Development_SOP.md`)
- Build: `pnpm tsc` + `node dist/index.js`
- Phase 2: JWT `signToken`/`verifyToken` with `JWT_SECRET` env var
- Phase 4: Security — Rate Limiting, Helmet headers, `pnpm audit`, Secret scan
- Phase 5: Docker `docker compose up -d` for local, Railway/Render for production
- CORS: Must configure frontend origin (not `*`)

### Quality Gates (all platforms)
- Each SOP Phase 8 includes: code quality scan + security audit + quality gate
- Gate: no build errors, lint clean, no hardcoded secrets, max file size checks
- User must confirm "code audit passed" before progressing to Phase 9

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
