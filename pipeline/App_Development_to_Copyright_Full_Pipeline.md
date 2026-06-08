# 🚀 iOS App 开发 → 上架 → 软著申请 全流程 SOP

> **适用对象**: Claude Code (AI Agent) 全自动 + 用户协作
> **目标**: 从零代码 → App Store 上架 → 软著登记证书，一条流水线
> **总周期**: 开发 2 天 + App Store 审核 1-3 天 + 软著审查 ~30 工作日
> **核心理念**: 开发阶段产出的材料直接复用为软著申请材料，零重复劳动
> **前置条件**: 已阅读并理解 `ios/../ios/iOS_App_2Day_Development_SOP.md` 和 `copyright/../copyright/Software_Copyright_Application_SOP.md`

---

## 📋 全流程时间线

```
═══════════════════════════════════════════════════════════════════════
  PART 1: App 开发 (Day 1-2)  →  审核等待 (1-3天)  →  PART 2: 软著 (上架后)
═══════════════════════════════════════════════════════════════════════

Day 1 — 从零到可运行 MVP
├─ Phase 0   环境初始化 → 产出: .xcodeproj, Git, 项目变量
├─ Phase 1   需求确认 → 产出: SPECS.md
├─ Phase 1.2 PRD 文档  → 产出: PRD.md (功能规格/屏幕规格/验收标准)
├─ Phase 1.5 高保真原型 → 产出: DESIGN_SPECS.md, Prototype/ (基于 PRD)
├─ Phase 2   架构搭建 → Stub 视图 + App 入口
├─ Phase 3   数据层 → SwiftData Models + StorageService + modelContainer
├─ Phase 4   ViewModel + DI 容器
├─ Phase 5   UI 层 (删除 Stub → 按原型 1:1 实现)
└─ Phase 6   自测 + 模拟器视觉验证

Day 2 — 从 MVP 到可发布
├─ Phase 7   集成测试 + 边界情况 + 无障碍基础扫描
├─ Phase 8   性能优化 + 无障碍完整审计 + 隐私清单
├─ Phase 9   App Store 素材 ★★ 软著材料主要来源
├─ Phase 10  内购 (如适用)
├─ Phase 11  Archive + TestFlight 上传
├─ Phase 12  App Store Connect 配置 + 提交审核
└─ Phase 13  归档 + 审核防拒检查

═══════════════════ App Store 审核中 (1-3 天) ═══════════════════
                    │
                    ├─ 审核通过 → App 上架 → 记录上架日期 → 进入 PART 2
                    │
                    └─ 审核被拒 → 修复 → 重新提交 → 等待通过
                       (软著材料可在此期间准备, 与审核并行)

═══════════════════ PART 2: 软著申请 (上架后 ~30 工作日) ═══════════════════

软著 D-Day (材料准备, ~1 小时, 大量复用 PART 1)
├─ P2-Phase 1   信息收集 → 80% 从 PART 1 直接复制
├─ P2-Phase 2   源代码文档 → git checkout v1.0.0 → 提取 → PDF
├─ P2-Phase 3   用户手册 → 复用 App Store 素材 + 截图
├─ P2-Phase 4   权利归属 + 申请表预填
└─ P2-Phase 5   五维度核验

在线提交 (0.5 天)
└─ P2-Phase 6   实名认证 → 填写 → 上传 → 提交

审查等待 (~30 工作日)
└─ P2-Phase 7   受理 → 补正应对 → 公告 → 发证

领证 & 归档
└─ P2-Phase 8   证书下载 → 归档到 Git 仓库

---

## 🧠 Claude Code 技能调用矩阵

| 技能 | 说明 | 使用频率 |
|------|------|---------|
| `[SHELL]` | Bash 执行 (xcodebuild/git/代码提取/PDF生成) | ██████████ 最高 |
| `[WRITE]` | 文件写入 (代码/文档/模板) | ██████████ 最高 |
| `[GENERATE]` | 代码生成 (Swift/SwiftUI/Python) | ████████ |
| `[DIALOG]` | 用户交互 (确认/填写/上传) | ██████ |
| `[DEBUG]` | 编译错误自动修复 | ██████ |
| `[REVIEW]` | 代码质量 & HIG 审查 | █████ |
| `[VALIDATE]` | 材料合规验证 | █████ |
| `[GIT]` | 版本控制 | █████ |
| `[RESEARCH]` | HIG/政策查阅 | ███ |

---

# PART 1: iOS App 开发 & 上架

> **详细步骤**: 参见 `../ios/../ios/iOS_App_2Day_Development_SOP.md` (5410 行)

---

## P1-Phase 0: 环境初始化 (Day 1, 0:00-0:30)

> **[SHELL]** + **[DIALOG]** 检测环境 → 创建 Xcode 项目 → 初始化 Git

```
产出物:
□ .xcodeproj 项目文件
□ Assets.xcassets (AccentColor + LaunchBackground)
□ Git 仓库 (main 分支)
□ /tmp/sop_project.env (项目变量)
□ /tmp/sop_simulator.env (模拟器信息)

┌─ 📌 软著复用 ──────────────────────────────────┐
│ 项目名称 ${PROJECT_NAME} 和 Bundle ID          │
│ 将在 PART 2 直接作为 "软件全称" 和 "标识"       │
└───────────────────────────────────────────────┘
```

---

## P1-Phase 1: 产品需求确认 (Day 1, 0:30-1:00)

> **[DIALOG]** + **[WRITE]** 结构化问卷 → 输出 SPECS.md

```
产出物: SPECS.md

┌─ 📌 软著复用 ──────────────────────────────────┐
│ SPECS.md → PRD.md → 软著申请表:                 │
│  - App 名称 → 软件全称                         │
│  - 功能规格(MoSCoW) → 开发目的/功能列表          │
│  - 用户画像 → 面向领域                          │
│  - 技术栈 → 编程语言/开发环境                    │
│  - PRD 数据模型 → 软著设计说明书模块划分         │
└───────────────────────────────────────────────┘
```

---

## P1-Phase 1.5: 高保真原型设计 (Day 1, 1:00-2:00)

> **[GENERATE]** + **[WRITE]** SwiftUI Previews 原型 + DESIGN_SPECS.md

```
产出物:
□ DESIGN_SPECS.md (颜色/字体/间距/组件规范)
□ Prototype/ 目录 (每个屏幕的原型文件)
□ 每个原型的亮/暗模式 Preview

┌─ 📌 软著复用 ──────────────────────────────────┐
│ 1. 原型截图 → 软著用户手册的功能界面截图        │
│    (直接在 Xcode Canvas 中截取, 无需等 App 上架) │
│ 2. DESIGN_SPECS.md "功能列表" → 用户手册核心功能 │
│ 3. 原型中的 "运行环境" 信息 → 软著申请表        │
└───────────────────────────────────────────────┘
```

---

## P1-Phase 2-5: 开发 → 数据层 → ViewModel → UI (Day 1, 2:00-8:00)

> **[GENERATE]** + **[WRITE]** + **[DEBUG]** 6 小时密集开发，每阶段结束编译验证

```
Phase 2  架构搭建 (2:00-2:30)
  □ ${PROJECT_NAME}App.swift (@main 入口, scenePhase 管理)
  □ ContentView.swift (Stub 视图 — HomeView/DiscoverView/ProfileView 占位)
  □ Utilities/ (AppError, AppConstants, ServiceProtocol)
  □ Resources/Info.plist (UILaunchScreen + ITSAppUsesNonExemptEncryption)
  → 编译验证通过 ✅

Phase 3  数据层 (2:30-4:30)
  □ Models/ (SwiftData @Model — TaskItem 等)
  □ Services/StorageService.swift (ModelContainer + CRUD)
  □ Services/NetworkService.swift (URLSession async/await)
  → 编译验证通过 ✅ + App 入口添加 .modelContainer(for:)

Phase 4  ViewModel (4:30-6:00)
  □ ViewModels/BaseViewModel.swift (isLoading/error/toast)
  □ ViewModels/TaskListViewModel.swift (filter/sort/search CRUD)
  □ Utilities/DIContainer.swift (依赖注入, Preview Mock)
  → 编译验证通过 ✅

Phase 5  UI 层 (6:00-7:30)
  □ Step 5.0: 删除 ContentView 中的 Stub 视图
  □ Views/Components/DesignTokens.swift (HIG 合规颜色/字体/间距)
  □ Views/HomeView.swift (列表 + 筛选 + 排序 + 搜索)
  □ Views/DetailView.swift + SettingsView.swift
  □ HIG 导航模式 (TabBar/NavigationStack/Search/ContextMenu)
  □ HIG 模态 & 反馈 (Sheet/Alert/ConfirmationDialog/Toast/Haptic)
  □ HIG 引导/启动/设置 (Onboarding/UILaunchScreen/ContentUnavailableView)
  → 编译验证通过 ✅ → 模拟器截图验证

┌─ 📌 软著复用 ────────────────────────────────┐
│ 1. 全部 .swift 源码 = 软著源代码文档原材料    │
│    SOURCE_DIR = ${PROJECT_DIR}/${PROJECT_NAME}/│
│ 2. 项目模块划分 → 软著设计说明书 "模块划分"   │
│ 3. 开发完成后 git tag v1.0.0 → 版本锚点      │
└─────────────────────────────────────────────┘
```

---

## P1-Phase 6: Day 1 收尾 & 视觉验证 (Day 1, 7:30-8:00)

> **[SHELL]** + **[DIALOG]** 模拟器截图 + 用户确认

```
产出物:
□ screenshot_visual_check.png (模拟器截图)
□ DAY1_REPORT.md

┌─ 📌 软著复用 ──────────────────────────────────┐
│ screenshot_visual_check.png                   │
│ → 可作为软著用户手册的 "首页" 截图              │
│   (模拟器截图分辨率高、无水印, 非常适合)         │
└───────────────────────────────────────────────┘
```

---

## P1-Phase 7-8: Day 2 集成测试 & 无障碍 (Day 2, 0:00-3:30)

> **[DEBUG]** + **[REVIEW]** + **[VALIDATE]** 质量验证

```
产出物:
□ 集成测试报告
□ HIG 合规验证 (180 处 HIG 引用)
□ PrivacyInfo.xcprivacy

┌─ 📌 软著复用 ──────────────────────────────────┐
│ 无障碍审计结果可写入软著用户手册的 "特殊说明"    │
│ (展示软件对不同用户群体的适配性)                 │
└───────────────────────────────────────────────┘
```

---

## P1-Phase 9: App Store 素材准备 (Day 2, 3:30-4:30)

> **[SHELL]** + **[WRITE]** + **[DIALOG]** 图标 + 截图 + 元数据

```
产出物:
□ App 图标 (1024x1024 + 各尺寸)
□ App Store 截图 (6.7" + 6.5")
□ fastlane/metadata/ (描述/关键词/宣传文本)

┌─ 📌 软著复用 ★★★ 关键交叉点 ─────────────────┐
│                                               │
│ 1. App Store 截图 → 软著用户手册截图            │
│    (截图已符合分辨率要求, 直接插入手册)          │
│                                               │
│ 2. App Store 描述 → 软著用户手册 "软件概述"     │
│    (功能列表、适用场景可直接复用)               │
│                                               │
│ 3. App Store 关键词 → 软著 "面向领域"          │
│                                               │
│ 4. 技术支持 URL → 用户手册 "联系方式"           │
│                                               │
│ 5. 首次上架日期 → 软著 "首次发表日期"           │
│    (App Store Connect 显示的上架日期即官方证明)  │
│                                               │
│ ⚡ 省时: 截图+描述复用节省软著准备 60% 时间      │
└───────────────────────────────────────────────┘
```

---

## P1-Phase 10-13: Archive → 提交审核 → 归档 (Day 2, 4:30-8:00)

> **[SHELL]** + **[DIALOG]** + **[GIT]** 上架最后冲刺

```
产出物:
□ TestFlight Build
□ App Store Connect 提交
□ Git tag v1.0.0

┌─ 📌 软著复用 ──────────────────────────────────┐
│ 1. App Store 上架成功 = 软著 "已发表证明"       │
│    (App Store Connect 截图 = 官方发表证据)      │
│                                               │
│ 2. Git tag v1.0.0 = 版本号锚点                 │
│    (软著源代码提取时 checkout 此 tag)            │
│                                               │
│ 3. 上架日期 = 软著 "首次发表日期"              │
│    (精确到日, App Store Connect 可查)           │
└───────────────────────────────────────────────┘
```

---

## 🔀 交叉点总结: PART 1 → PART 2 材料映射

| PART 1 产出 | 路径/命令 | PART 2 用途 | 处理方式 |
|------------|----------|------------|---------|
| 项目名称 | `${PROJECT_NAME}` | 软件全称 | ⚠️ 需加 "软件/系统" 后缀和 "V版本号" |
| 版本号 | `Info.plist CFBundleShortVersionString` | 软著版本号 | 直接填入 (确保 V 大写) |
| 功能列表 | `SPECS.md` 核心功能 | 用户手册 + 申请表 开发目的 | 改写为技术文档语气 |
| 技术栈 | `SPECS.md` 技术选型 | 申请表 编程语言/开发环境 | 直接复制 |
| 运行环境 | `Info.plist MinimumOSVersion` | 申请表 软件环境 | 直接填入 |
| 项目目录结构 | `ls ${PROJECT_NAME}/` | 设计说明书 模块划分 | 直接引用 |
| 全部源代码 | `${PROJECT_NAME}/*.swift` | 源代码文档 (PDF) | git checkout tag → find → 排版 |
| 模拟器截图 | `xcrun simctl io screenshot` | 用户手册 功能截图 | 每功能截1张 (非 App Store 营销图) |
| 设计线框图 | `DESIGN_SPECS.md` ASCII 图 | 用户手册 界面说明 | 改写为操作步骤 |
| 上架日期 | ASC "Ready for Sale" 日期 | 首次发表日期 | 精确填入 yyyy-mm-dd |
| ASC 上架确认 | ASC App 状态截图 | 已发表证明 | 截图作为证明材料 |
| Git tag | `git tag v1.0.0` | 版本锚点 | `git checkout -b copyright-v1.0.0 v1.0.0` |
| Bundle ID | `${BUNDLE_ID}` | 软著不直接需要 | 作为软件标识归档 |

---

# PART 2: 软件著作权申请

> **详细步骤**: 参见 `../copyright/../copyright/Software_Copyright_Application_SOP.md` (1269 行)
> **关键原则**: 最大限度复用 PART 1 的产出，避免重复劳动

---

## P2-Phase 1: 信息收集 (软著 D-Day, 0:00-0:30)

> **[DIALOG]** + **[WRITE]** 复用 PART 1 信息，仅补充软著特有字段

```
不需要重复收集的信息 (从 PART 1 直接复制):
□ 软件全称 ← PART 1 SPECS.md 或 ${DISPLAY_NAME} + 功能描述
□ 版本号   ← PART 1 项目版本号 (如 "V1.0.0")
□ 编程语言 ← PART 1 SPECS.md 技术栈
□ 硬件环境 ← PART 1 Info.plist / DESIGN_SPECS.md
□ 软件环境 ← PART 1 Info.plist / DESIGN_SPECS.md
□ 开发目的 ← PART 1 SPECS.md 产品描述
□ 面向领域 ← PART 1 SPECS.md 目标用户

仅需新收集的字段 (5 分钟):
□ 申请人信息 (姓名/证件号/地址)
□ 软件分类号 (移动应用 → 30213-0000)
□ 开发完成日期 (代码最后提交日期)
□ 首次发表日期 (App Store 上架日期)

⚡ 省时: 信息收集从 30 分钟缩短到 10 分钟
```

---

## P2-Phase 2: 源代码文档 (软著 D-Day, 0:30-1:00)

> **[SHELL]** + **[WRITE]** 从上架版本 Git tag 提取源代码 → 文件标识 → Python/手动 PDF

```
┌─ 执行步骤 ─────────────────────────────────────┐
│                                                │
│ 1. 加载 PART 1 项目变量:                        │
│    source /tmp/sop_project.env                 │
│    cd ${PROJECT_DIR}                           │
│                                                │
│ 2. 切换到上架版本 (用临时分支, 不破坏工作区):     │
│    git checkout -b copyright-v1.0.0 v1.0.0     │
│    (⚠️ 不要直接 checkout tag → 会 detached HEAD) │
│                                                │
│ 3. 提取源代码:                                  │
│    SOURCE_DIR="${PROJECT_DIR}/${PROJECT_NAME}"  │
│    bash copyright_materials/extract_source.sh  │
│    (运行 ../copyright/Software_Copyright_Application_SOP.md  │
│     中 Phase 2.2 的完整脚本)                     │
│                                                │
│ 4. 生成 PDF:                                   │
│    python3 generate_source_pdf.py              │
│    (或手动 Word/Pages 排版)                     │
│                                                │
│ 5. 回到主分支:                                  │
│    git checkout main                           │
│    git branch -d copyright-v1.0.0  # 清理临时分支│
│                                                │
│ 6. 确认 PDF 包含:                               │
│    - 前30页: @main App 入口 (第一个文件第一行)    │
│    - 业务逻辑: ViewModel CRUD 操作              │
│    - 后30页: 最后几个文件 + 程序结尾              │
│    - 文件分隔: "// ===== 文件: xxx.swift ====="  │
│    - 第三方库已排除 (Pods/.build/node_modules)    │
│                                                │
│ ⚡ 省时: 源码路径/版本号/排除目录已知              │
└────────────────────────────────────────────────┘
```

---

## P2-Phase 3: 用户手册 (软著 D-Day, 1:00-1:30)

> **[WRITE]** 复用 PART 1 素材。关键: 软著用户手册 ≠ App Store 宣传截图。

```
┌─ 截图选择策略 (重要!) ──────────────────────────┐
│                                                │
│ ⚠️ App Store 截图 = 营销素材 (有设备框/文案)     │
│    → 不适合直接用于软著用户手册!                 │
│                                                │
│ 软著用户手册需要的是:                            │
│ ✅ 纯应用界面截图 (无设备框/无营销文字)           │
│ ✅ 显示实际功能操作的截图                         │
│                                                │
│ 截图来源优先级:                                  │
│ 1. 模拟器截图 (PART 1 Phase 5.6)                │
│    xcrun simctl io booted screenshot            │
│    → 最纯净、无水印、可精确截取每个功能界面       │
│                                                │
│ 2. Xcode Canvas 原型截图 (PART 1 Phase 1.5)     │
│    → 适合未上架或需要特定状态的界面               │
│                                                │
│ 3. App Store 截图 (去框版本)                     │
│    → 仅当模拟器不可用时使用, 需裁掉设备框         │
│                                                │
│ 快速截图命令:                                    │
│ xcrun simctl io booted screenshot \             │
│   copyright_materials/screenshots/home.png      │
└────────────────────────────────────────────────┘

┌─ 快速填充策略 ──────────────────────────────────┐
│                                                │
│ 1. "软件概述"                                   │
│    ← 改写 PART 1 SPECS.md 产品描述              │
│    ← 不是直接复制 App Store 营销描述!            │
│    ← 采用技术文档语气: "本软件是一款...用于..."   │
│                                                │
│ 2. "功能列表"                                   │
│    ← 从 DESIGN_SPECS.md "屏幕清单" 提取         │
│    ← 每功能一句话: "首页: 展示任务列表,支持筛选"  │
│                                                │
│ 3. "运行环境"                                   │
│    ← 从 Info.plist 提取:                        │
│    MinimumOSVersion → "iOS 17.0 及以上"         │
│    UIRequiredDeviceCapabilities → 硬件要求       │
│                                                │
│ 4. "功能操作说明" (核心章节)                     │
│    ← 每张模拟器截图配 3-5 步操作步骤              │
│    ← 步骤格式: "1. 点击[xx] → 2. 输入[xx] → ..." │
│    ← 描述的不是"好看在哪"而是"怎么用"             │
│                                                │
│ 5. "联系方式"                                   │
│    ← PART 1 Phase 9.4 的 support_url 和邮箱     │
│                                                │
│ ⚡ 省时: 截图从模拟器直接截取, 无需重新设计       │
└────────────────────────────────────────────────┘
```

---

## P2-Phase 4: 权利归属 & 申请表 (软著 D-Day, 1:30-2:00)

> **[WRITE]** + **[DIALOG]** 生成权利文件 + 预填申请表

```
申请表预填 (80% 字段已从 PART 1 获知):
□ 软件全称/版本号/分类号   ← PART 1
□ 编程语言/开发环境        ← PART 1
□ 硬件/软件环境            ← PART 1
□ 代码行数                 ← P2-Phase 2 统计
□ 开发目的/面向领域        ← PART 1
□ 首次发表日期             ← App Store 上架日

仅需用户填写的 20%:
□ 申请人信息 (姓名/证件)
□ 开发完成日期
□ 权利归属 (独立/合作/职务)
```

---

## P2-Phase 5-8: 核验 → 提交 → 领证

> **[VALIDATE]** + **[DIALOG]** 按 ../copyright/Software_Copyright_Application_SOP.md 执行

```
Phase 5: 五维度核验 (15 分钟)
Phase 6: 在线提交 (30 分钟)
Phase 7: 审查追踪 (每周检查)
Phase 8: 证书归档 (领证后)

┌─ 📌 证书归档到 PART 1 项目 ───────────────────┐
│ git add docs/copyright/                       │
│ git commit -m "Add software copyright cert"    │
│ git tag v1.0.0-copyright                      │
│ (软著证书与源码在同一仓库, 永久关联)             │
└───────────────────────────────────────────────┘
```

---

## 📊 全流程效率对比

| 环节 | 独立申请软著 | 使用本流水线 | 节省 | 原因 |
|------|-----------|-----------|------|------|
| 信息收集 | 30 分钟 | 10 分钟 | 67% | SPECS.md + Info.plist 已有 80% 字段 |
| 源代码路径确认 | 15 分钟 | 1 分钟 | 93% | `$PROJECT_DIR` 已持久化在 /tmp/sop_project.env |
| 源代码提取 & PDF | 45 分钟 | 20 分钟 | 56% | 排除目录已知, 脚本直接运行 |
| 用户手册截图 | 30 分钟 | 5 分钟 | 83% | 模拟器 `xcrun simctl io screenshot` 批量截取 |
| 用户手册编写 | 60 分钟 | 25 分钟 | 58% | 功能列表/运行环境已有, 仅需补操作步骤 |
| 申请表填写 | 30 分钟 | 8 分钟 | 73% | APPLICATION_FORM.md 预填 80% |
| 已发表证明 | 15 分钟 | 1 分钟 | 93% | ASC 上架截图直接使用 |
| **总计** | **~3.75 小时** | **~1.2 小时** | **~68%** | |

> 注: 上述时间为纯操作时间, 不含审查等待 (30 工作日, 两种方式相同)

---

## 🗺️ 完整流程图

```
                    PART 1: 2 天开发 + 上架
                    ═══════════════════
                              │
    ┌─────────────────────────┼─────────────────────────┐
    │                         │                         │
    ▼                         ▼                         ▼
 SPECS.md             全部源代码                  App Store 截图
 (需求文档)          (.swift 文件)               (6.7" 截图)
    │                    │                            │
    │                    │                            │
    └────────────────────┼────────────────────────────┘
                         │
                    ╔═════╧═════╗
                    ║  上架成功  ║
                    ║ (Day 2 末)║
                    ╚═════╤═════╝
                          │
            ┌─────────────┼─────────────┐
            │             │             │
            ▼             ▼             ▼
       git tag      上架日期       上架截图
       v1.0.0      (精确到日)     (已发表证明)
            │             │             │
            └─────────────┼─────────────┘
                          │
                    PART 2: 软著申请
                    ═══════════════
                          │
            ┌─────────────┼─────────────┐
            │             │             │
            ▼             ▼             ▼
      源代码提取      用户手册       权利证明
      (复用PART1)   (复用截图+描述)  (新生成)
            │             │             │
            └─────────────┼─────────────┘
                          │
                    在线提交 → 审查 → 领证
                          │
                          ▼
                   📜 软著登记证书
                    归档到 Git 仓库
```

---

## ⚡ Claude Code 执行策略

### 执行阶段

```
═══════════════════════════════════════════════════════════
阶段 A: PART 1 开发 (Day 1-2)
═══════════════════════════════════════════════════════════
1. 执行 ../ios/iOS_App_2Day_Development_SOP.md Phase 0 → Phase 13
2. 产出: App Archive 已上传到 App Store Connect
3. 产出: git tag v1.0.0 已创建 (Phase 13 Step 13.1)
4. 产出: 所有项目变量持久化在 /tmp/sop_project.env
5. 状态: App 在 ASC 中等待审核

═══════════════════════════════════════════════════════════
阶段 B: App Store 审核等待 (1-3 天) ↔ 软著材料可并行准备
═══════════════════════════════════════════════════════════
审核结果处理:
├─ 审核通过 → App 上架 (Ready for Sale)
│   → 记录上架日期: [yyyy-mm-dd]
│   → 进入阶段 C
│
└─ 审核被拒 → 查看拒绝原因
    → 修复问题 (通常 1-2 小时)
    → 在 ASC 中回复/重新提交
    → 等待再次审核 (1-2 天)
    → ⚡ 软著材料可以在此期间准备! 不需要等 App 上架
    → 唯一需要等上架的是 "首次发表日期"

═══════════════════════════════════════════════════════════
阶段 C: PART 2 软著申请 (上架后 ~1 小时 + 审查等待)
═══════════════════════════════════════════════════════════
1. 加载 PART 1 环境变量
2. 执行 ../copyright/Software_Copyright_Application_SOP.md Phase 1 → Phase 5
   (材料准备, 大量复用 PART 1)
3. Phase 6 在线提交 (需要用户手动操作 ASC/版权中心)
4. Phase 7 审查追踪 (每周检查, Claude Code 提醒)
5. Phase 8 证书归档

═══════════════════════════════════════════════════════════
⚠️ 并行策略: 软著材料准备可以与 App Store 审核并行!
   - 源代码文档: 不依赖 App 上架 (代码已存在)
   - 用户手册: 截图用模拟器, 不依赖 App Store 截图
   - 申请表: 除 "首次发表日期" 外都可预填
   - 权利证明: 不依赖 App 上架
   → 在等待审核的 1-3 天内完成软著材料准备
   → App 一上架, 填入上架日期即可提交
═══════════════════════════════════════════════════════════
```

### 关键命令速查

```bash
# ===== PART 1 → PART 2 环境变量传递 =====
source /tmp/sop_project.env         # PROJECT_NAME, PROJECT_DIR, BUNDLE_ID
source /tmp/sop_simulator.env       # SIMULATOR_ID, SIMULATOR_NAME, TEAM_ID
echo "PROJECT_DIR=${PROJECT_DIR}"
echo "BUNDLE_ID=${BUNDLE_ID}"

# ===== 提取上架版本源代码 (用临时分支避免 detached HEAD) =====
cd ${PROJECT_DIR}
git checkout -b copyright-v1.0.0 v1.0.0   # 基于 tag 创建临时分支
SOURCE_DIR="${PROJECT_DIR}/${PROJECT_NAME}"
echo "源码目录: ${SOURCE_DIR}"
ls ${SOURCE_DIR}/App/                       # 确认目录存在

# ===== 模拟器截图 (用于软著用户手册) =====
# 启动 App 后执行:
xcrun simctl io booted screenshot \
  copyright_materials/screenshots/home.png
xcrun simctl io booted screenshot \
  copyright_materials/screenshots/feature1.png
# ... 每个功能截一张

# ===== 复用项目文档 =====
mkdir -p copyright_materials
cp DESIGN_SPECS.md copyright_materials/
cp SPECS.md copyright_materials/
cp fastlane/metadata/zh-Hans/description.txt copyright_materials/appstore_desc.txt

# ===== 软著完成后恢复 =====
git checkout main                        # 回到主分支
git branch -d copyright-v1.0.0           # 清理临时分支 (保留 tag)
```

### 版本号一致性验证

```bash
# 软著申请的 "版本号" 必须与代码一致
# 验证命令:
echo "Git tag:    $(git tag -l 'v*' | tail -1)"
echo "App 版本:   $(/usr/libexec/PlistBuddy -c 'Print CFBundleShortVersionString' ${PROJECT_DIR}/${PROJECT_NAME}/Resources/Info.plist 2>/dev/null)"
echo "软著版本:    V1.0.0  ← 需与上述一致 (注意 V 大写)"

# 确保软著申请表中的版本号 = Info.plist CFBundleShortVersionString = git tag
```

---

## ⚠️ 软件名称注意事项

```
App Store 名称 ≠ 软著软件全称 (通常不同, 也可以相同)

示例:
  App Store 显示名: "闪记"              (2字品牌名)
  软著软件全称:     "闪记智能笔记管理软件V1.0"  (品牌+功能+软件/系统+V版本)

规则:
  - 软著全称必须包含: 品牌 + 功能描述 + "软件/系统/平台/应用" + V版本号
  - App Store 名称可以只是品牌名 (2-30字符)
  - 软著简称可以填 App Store 名称 (如 "闪记")
  - 两者不强制一致, 但建议有包含关系

❌ 常见错误:
  - 用 App Store 名称直接当软著全称 → 缺少 "软件" 后缀和版本号 → 驳回
  - 软著全称和简称完全相同 → 必须不同 → 驳回
```

## 📋 全流程检查清单

```
═══════════════════════════════════════════════════════
PART 1 完成条件
═══════════════════════════════════════════════════════
□ App 已在 App Store Connect 提交审核
□ git tag v1.0.0 已创建 (Phase 13)
□ /tmp/sop_project.env 持久化正常
□ SPECS.md, DESIGN_SPECS.md 在项目根目录
□ 模拟器截图已保存 (screenshot_visual_check.png)
□ fastlane/screenshots/ 有 App Store 截图

═══════════════════════════════════════════════════════
App Store 审核结果
═══════════════════════════════════════════════════════
□ 审核状态: [通过 / 被拒]
□ 如通过 → Ready for Sale 日期: yyyy-mm-dd
□ 如被拒 → 拒绝原因: _______________ → 修复后重新提交

═══════════════════════════════════════════════════════
PART 1 → PART 2 交接 (审核期间可并行准备)
═══════════════════════════════════════════════════════
□ 软著软件全称已确认: _______________ (含"软件"后缀+V版本)
□ 版本号: V____ (与 git tag 一致, V 大写)
□ 分类号: 30213-0000 (移动应用软件)
□ SOURCE_DIR 路径确认: _______________
□ git checkout -b copyright-v1.0.0 v1.0.0 可正常切换
□ Info.plist CFBundleShortVersionString 已确认

═══════════════════════════════════════════════════════
PART 2 完成条件
═══════════════════════════════════════════════════════
□ 源代码 PDF 已生成 (≥60页, 页眉/页码正确)
□ 用户手册 PDF 已生成 (≥15页, ≥5张截图)
□ 权利归属证明已签署/盖章
□ 申请表字段已预填完成
□ 五维度核验全部通过
□ 实名认证已完成
□ 在线提交成功 → 受理号: _______________
□ 审查状态: [已受理 / 审查中 / 补正 / 已公告 / 已发证]
□ 电子证书已下载: _______________.pdf
□ 证书归档: git tag v1.0.0-copyright
```

---

## 📋 鸿蒙 → 软著流水线 (HarmonyOS 版)

> **适配**: 鸿蒙 App 开发完成后，复用开发阶段产物快速完成软著申请。

### 鸿蒙版材料映射

| 鸿蒙开发产出 | 软著用途 | 获取方式 |
|------------|---------|---------|
| SPECS.md (功能/技术栈) | 申请表 开发目的/编程语言 | 直接复制 |
| DESIGN_SPECS.md (运行环境) | 申请表 硬件/软件环境 | 直接复制 |
| DevEco Previewer 截图 | 用户手册功能截图 | 每个页面截图 |
| 全部 .ets 源码 | 源代码文档 (PDF) | find *.ets → 排版 |
| 项目结构 (entry/src/main/ets/) | 设计说明书模块划分 | 直接引用 |
| AGC 上架日期 | 首次发表日期 | 精确填入 |
| hvigorw 构建产物路径 | 版本号锚点 | `app.json5 versionName` |

### 关键差异 (vs iOS → 软著)

```
鸿蒙 vs iOS 软著申请差异:
  □ 软件分类号: 选 "30213-0000 移动应用软件" (同 iOS)
  □ 编程语言: 填 "ArkTS" (非 Swift)
  □ 开发环境: 填 "DevEco Studio 5.0, HarmonyOS SDK API 12"
  □ 软件环境: 填 "HarmonyOS NEXT 5.0+"
  □ 硬件环境: 填 "华为手机/平板, 麒麟芯片"
  □ 源代码格式: .ets 文件 (TypeScript 超集)
  □ 用户手册截图: DevEco Previewer 截图 (不是 App Store 截图)
  □ 已发表证明: AGC 上架截图 (不是 App Store Connect)

执行步骤:
  1. 鸿蒙 SOP Phase 13 完成后 → git tag v1.0.0
  2. 进入 copyright/ 目录 → 执行 Software_Copyright_Application_SOP.md
  3. 源代码提取: SOURCE_DIR = 项目 entry/src/main/ets/
  4. 截图: 复用 DevEco Previewer 截图
  5. 其他流程相同
```

---

> **流水线版本**: 1.2.0
> **最后更新**: 2026-06-08
> **关联文档**: `../ios/iOS_App_2Day_Development_SOP.md` + `../copyright/Software_Copyright_Application_SOP.md` + `../harmonyos/HarmonyOS_App_2Day_Development_SOP.md`
> **核心理念**: 开发阶段的每一项产出都是软著申请的材料，不要做两次

---

## 🤖 一键材料提取脚本

> Claude Code 执行此脚本自动从开发项目中提取软著所需全部材料。

### iOS 版

```bash
#!/bin/bash
# [SHELL] 一键提取 iOS 项目软著材料
# 使用: bash extract_copyright_materials_ios.sh

source /tmp/sop_project.env 2>/dev/null
PROJECT_DIR="${PROJECT_DIR:-.}"
OUTPUT="./copyright_materials"
mkdir -p ${OUTPUT}/{source,screenshots,docs}

echo "═══════════════════════════════════════"
echo "  一键提取 iOS → 软著材料"
echo "═══════════════════════════════════════"

# 1. 源代码 (保留文件结构)
echo "📄 [1/5] 提取源代码..."
SOURCE_DIR="${PROJECT_DIR}/${PROJECT_NAME:-}"
find "${SOURCE_DIR}" -name "*.swift" -not -path "*/Pods/*" -not -path "*/.build/*" | sort > ${OUTPUT}/source_files.txt
FILE_COUNT=$(wc -l < ${OUTPUT}/source_files.txt)
echo "   源文件: ${FILE_COUNT} 个"

# 带文件头标识
> ${OUTPUT}/source/full_source.txt
while IFS= read -r f; do
  rel="${f#${SOURCE_DIR}/}"
  echo "// ===== 文件: ${rel} =====" >> ${OUTPUT}/source/full_source.txt
  grep -v '^\s*$' "$f" >> ${OUTPUT}/source/full_source.txt 2>/dev/null || true
  echo "" >> ${OUTPUT}/source/full_source.txt
done < ${OUTPUT}/source_files.txt
TOTAL_LINES=$(wc -l < ${OUTPUT}/source/full_source.txt)
echo "   总行数: ${TOTAL_LINES}"
echo "   ✅ 源代码提取完成 → ${OUTPUT}/source/full_source.txt"

# 2. 截图 (从 fastlane 或手动目录)
echo "📸 [2/5] 收集截图..."
if [ -d "fastlane/screenshots" ]; then
  cp fastlane/screenshots/*.png ${OUTPUT}/screenshots/ 2>/dev/null
  SCREENSHOT_COUNT=$(ls ${OUTPUT}/screenshots/*.png 2>/dev/null | wc -l)
  echo "   截图: ${SCREENSHOT_COUNT} 张 (来源: fastlane)"
else
  # 检查是否有模拟器截图
  if [ -f "screenshot_visual_check.png" ]; then
    cp screenshot_visual_check.png ${OUTPUT}/screenshots/
    echo "   截图: 1 张 (来源: 模拟器截图)"
  else
    echo "   ⚠️ 未找到截图, 请手动补充"
  fi
fi

# 3. 项目文档
echo "📋 [3/5] 收集项目文档..."
cp SPECS.md ${OUTPUT}/docs/ 2>/dev/null && echo "   ✅ SPECS.md"
cp DESIGN_SPECS.md ${OUTPUT}/docs/ 2>/dev/null && echo "   ✅ DESIGN_SPECS.md"
cp PRD.md ${OUTPUT}/docs/ 2>/dev/null && echo "   ✅ PRD.md"

# 4. App Store 信息
echo "📱 [4/5] 提取 App Store 元数据..."
if [ -f "fastlane/metadata/zh-Hans/description.txt" ]; then
  cp fastlane/metadata/zh-Hans/description.txt ${OUTPUT}/docs/appstore_description.txt
  echo "   ✅ App Store 描述"
fi
BUNDLE_ID=$(grep "BUNDLE_ID" /tmp/sop_project.env 2>/dev/null | cut -d'"' -f2)
echo "   Bundle ID: ${BUNDLE_ID:-未知}"
echo "   上架日期: (请手动填入 ASC Ready for Sale 日期)"

# 5. 生成软著信息摘要
echo "📝 [5/5] 生成软著信息摘要..."
cat > ${OUTPUT}/COPYRIGHT_INFO_AUTO.md << INFOEOF
# 软著申请信息 — 自动提取

## 软件信息 (从项目自动提取)
- 建议全称: \${PROJECT_NAME:-MyApp} [功能描述]软件V1.0.0
- 版本号: 1.0.0
- 编程语言: Swift $(swift --version 2>/dev/null | head -1 | grep -oE '[0-9]+\.[0-9]+')
- 硬件环境: iPhone 12及以上机型
- 软件环境: iOS $(/usr/libexec/PlistBuddy -c 'Print MinimumOSVersion' ${SOURCE_DIR}/Resources/Info.plist 2>/dev/null || echo "17.0")+
- 代码行数: ${TOTAL_LINES}
- 分类号: 30213-0000 (移动应用软件)

## 需用户手动补充
- 申请人姓名/证件号
- 软件详细描述
- 开发完成日期
- 首次发表日期 (App Store 上架日期)
- 隐私政策 URL
INFOEOF
echo "   ✅ 软著信息摘要 → ${OUTPUT}/COPYRIGHT_INFO_AUTO.md"

echo ""
echo "═══════════════════════════════════════"
echo "  ✅ 材料提取完成!"
echo "  📂 输出目录: ${OUTPUT}/"
echo "     ├── source/   (源代码)"
echo "     ├── screenshots/ (截图)"
echo "     └── docs/     (项目文档)"
echo ""
echo "  下一步: 执行 copyright/Software_Copyright_Application_SOP.md Phase 2-5"
echo "═══════════════════════════════════════"
```

### 鸿蒙版

```bash
#!/bin/bash
# [SHELL] 一键提取鸿蒙项目软著材料
source /tmp/sop_harmony.env 2>/dev/null
PROJECT_DIR="${PROJECT_DIR:-.}"
OUTPUT="./copyright_materials"
mkdir -p ${OUTPUT}/{source,screenshots,docs}

echo "═══════════════════════════════════════"
echo "  一键提取 鸿蒙 → 软著材料"
echo "═══════════════════════════════════════"

# 1. ArkTS 源代码
echo "📄 [1/4] 提取 ArkTS 源代码..."
SOURCE_DIR="${PROJECT_DIR}/entry/src/main/ets"
find "${SOURCE_DIR}" -name "*.ets" -not -path "*/oh_modules/*" | sort > ${OUTPUT}/source_files.txt
echo "   源文件: $(wc -l < ${OUTPUT}/source_files.txt) 个"

> ${OUTPUT}/source/full_source.txt
while IFS= read -r f; do
  rel="${f#${PROJECT_DIR}/}"
  echo "// ===== 文件: ${rel} =====" >> ${OUTPUT}/source/full_source.txt
  grep -v '^\s*$' "$f" >> ${OUTPUT}/source/full_source.txt 2>/dev/null || true
  echo "" >> ${OUTPUT}/source/full_source.txt
done < ${OUTPUT}/source_files.txt
echo "   总行数: $(wc -l < ${OUTPUT}/source/full_source.txt)"
echo "   ✅ 完成"

# 2. 截图 (Previewer 手动截取)
echo "📸 [2/4] 截图提示..."
echo "   ⚠️ 鸿蒙截图需在 DevEco Previewer 中手动截取各页面"
echo "   需要: 首页/详情页/设置页 至少各1张"

# 3. 项目文档
echo "📋 [3/4] 收集项目文档..."
cp SPECS.md ${OUTPUT}/docs/ 2>/dev/null
cp DESIGN_SPECS.md ${OUTPUT}/docs/ 2>/dev/null
cp PRD.md ${OUTPUT}/docs/ 2>/dev/null
echo "   ✅ 文档收集完成"

# 4. 软著信息
echo "📝 [4/4] 生成软著信息摘要..."
cat > ${OUTPUT}/COPYRIGHT_INFO_AUTO.md << INFOEOF
# 软著申请信息 — 自动提取 (鸿蒙)

- 编程语言: ArkTS
- 开发环境: DevEco Studio 5.0, HarmonyOS SDK API 12
- 软件环境: HarmonyOS NEXT 5.0+
- 硬件环境: 华为手机/平板, 麒麟芯片
- 分类号: 30213-0000 (移动应用软件)
INFOEOF
echo "   ✅ 完成"

echo "═══════════════════════════════════════"
echo "  ✅ 鸿蒙材料提取完成 → ${OUTPUT}/"
echo "═══════════════════════════════════════"
```
