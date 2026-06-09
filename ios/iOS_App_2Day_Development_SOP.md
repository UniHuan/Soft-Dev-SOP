# 📱 Claude Code + Xcode 两天全自动 iOS App 开发 SOP

> **适用对象**: Claude Code (AI Agent) 全自动执行
> **目标**: 从零到 App Store 上架，两天 (16 小时) 完成
> **技术栈**: SwiftUI + MVVM + Swift Concurrency + SwiftData
> **最低要求**: macOS 14+, Xcode 16+, Apple Developer Account ($99/年)

---

## 🧠 Claude Code 技能调用矩阵

> **每个步骤标注了 Claude Code 需要调用的核心技能。Claude Code 在读到该步骤时自动切换对应模式。**

| 技能标识 | 技能名称 | 说明 | Claude Code 工具 |
|---------|---------|------|-----------------|
| `[SHELL]` | Shell 执行 | 执行 bash 命令、xcodebuild、git | `Bash` |
| `[WRITE]` | 文件写入 | 创建/修改 .swift .md .plist .yml 等文件 | `Edit` / `Write` |
| `[READ]` | 文件读取 | 读取现有代码、日志、配置 | `Read` |
| `[DIALOG]` | 用户交互 | 向用户提问、请求确认、展示结果 | 对话输出 |
| `[GENERATE]` | 代码生成 | 生成业务逻辑、UI、测试代码 | `Edit` / `Write` |
| `[REVIEW]` | 代码审查 | 审查生成代码的质量、HIG 合规、安全性 | `Read` → 分析 |
| `[DEBUG]` | 调试分析 | 分析编译错误/测试失败日志并修复 | `Read` + `Bash` + `Edit` |
| `[RESEARCH]` | 知识检索 | 查阅 HIG、API 文档、最佳实践 | 内置知识 + `Read` HIG_KB |
| `[VALIDATE]` | 验证检查 | 对照清单逐项验证 | `Bash` + `Read` + 分析 |
| `[GIT]` | 版本控制 | git add/commit/tag/push | `Bash` |

### 技能调用原则

```
1. 每个 Phase 开始前 → [RESEARCH] 查阅本 SOP 的 HIG 约束和设计规格
2. 每次生成代码后 → [REVIEW] 对照前后步骤自检代码质量
3. 每次 xcodebuild 后 → [DEBUG] 分析输出、自动修复、不把错误抛给用户
4. 每个 Phase 结束前 → [VALIDATE] 对照该 Phase 的验收条件逐项确认
5. 每个 Phase 结束后 → [GIT] 提交代码
6. 涉及不可逆操作 → [DIALOG] 必须获得用户确认
```

---

## 📋 总览时间线

```
Day 1 (8h)                              Day 2 (8h)
├─ [0.0h] 环境检查 & 项目初始化          ├─ [0.0h] 集成测试 & Bug 修复
├─ [0.5h] 产品需求确认                   ├─ [0.5h] 性能优化 & 无障碍
├─ [1.0h] PRD 产品需求文档                ├─ [2.5h] App Store 素材准备
├─ [1.5h] 高保真原型设计 & 设计系统       ├─ [3.5h] 内购/订阅配置
├─ [2.5h] 架构搭建 & 目录结构            ├─ [4.5h] Archive & TestFlight
├─ [3.0h] 数据层 (Model + Service)       ├─ [5.5h] App Store Connect 提交
├─ [5.0h] 业务逻辑层 (ViewModel)         └─ [7.0h] 提交审核 & 文档归档
├─ [6.5h] UI 层 (View) ← 按原型实现
└─ [8.0h] 自测 & 代码 Review
```

---

## ⚠️ 执行前提 (Claude Code 自动检查)

在执行本 SOP 前，Claude Code 必须逐条确认以下条件：

```bash
# [SHELL] 1. 确认 macOS 版本 >= 14.0
sw_vers -productVersion

# [SHELL] 2. 确认 Xcode 已安装且版本 >= 16.0
xcodebuild -version

# [SHELL] 3. 确认 xcode-select 指向正确路径
xcode-select -p

# [SHELL] 4. 检测可用 iOS 模拟器 (后续所有 xcodebuild 使用此设备)
AVAILABLE_SIMULATOR=$(xcrun simctl list devices available iPhone | grep -E 'iPhone (16|17|18)' | head -1 | sed -E 's/.*\(([A-F0-9-]+)\).*/\1/')
if [ -z "$AVAILABLE_SIMULATOR" ]; then
    echo "❌ 未找到可用的 iPhone 模拟器，请安装 iOS 18+ Simulator Runtime"
    xcrun simctl list runtimes
    exit 1
fi
SIMULATOR_NAME=$(xcrun simctl list devices | grep "$AVAILABLE_SIMULATOR" | sed -E 's/^[[:space:]]+(.+) \(.*/\1/')
echo "📱 将使用模拟器: $SIMULATOR_NAME ($AVAILABLE_SIMULATOR)"
# 保存到临时文件供后续步骤使用
echo "SIMULATOR_NAME=\"$SIMULATOR_NAME\"" > /tmp/sop_simulator.env
echo "SIMULATOR_ID=\"$AVAILABLE_SIMULATOR\"" >> /tmp/sop_simulator.env

# [SHELL] 5. 确认 SPM 可用
swift package --version 2>/dev/null || echo "SPM available via Xcode"

# [SHELL] 6. 确认 Git 已配置
git --version
git config user.name
git config user.email

# [SHELL] 7. 确认 Apple Developer 登录状态 & 自动获取 Team ID
TEAM_ID=$(security find-identity -v -p codesigning 2>/dev/null | \
  grep "Apple Development" | \
  head -1 | \
  grep -oE '[A-Z0-9]{10}' | \
  head -1)
if [ -z "$TEAM_ID" ]; then
    echo "⚠️ 未检测到 Apple Developer 证书。"
    echo "   请在 Xcode → Settings → Accounts 登录 Apple ID"
    echo "   或访问 https://developer.apple.com/account → Membership 查看 Team ID"
    echo "   Team ID 示例: ABC123DEF4 (10位字母数字)"
else
    echo "✅ Team ID: $TEAM_ID"
    echo "TEAM_ID=\"$TEAM_ID\"" >> /tmp/sop_simulator.env
fi

# [SHELL] 8. 检查 Fastlane (可选)
which fastlane >/dev/null 2>&1 && echo "✅ Fastlane 已安装" || echo "⚠️ Fastlane 未安装 (Phase 11 可用 xcodebuild 替代)"
```

> **如果任一检查失败**: 立即停止，向用户报告缺失项和修复方法，不要继续执行。

---

# 🗓️ DAY 1 — 从零到可运行 MVP

---

## Phase 0: 环境初始化 (0:00 - 0:30)

> **[SHELL]** + **[DIALOG]** 环境检查和项目骨架创建

### Step 0.0 — 环境自检 & 工具安装

```bash
# [SHELL] 检查 macOS 版本
MACOS_VERSION=$(sw_vers -productVersion)
echo "macOS: $MACOS_VERSION"
if [[ "$MACOS_VERSION" < "14.0" ]]; then
    echo "❌ 需要 macOS 14.0+，当前: $MACOS_VERSION"
    exit 1
fi

# [SHELL] 检查 Xcode
XCODE_VERSION=$(xcodebuild -version | head -1 | awk '{print $2}')
echo "Xcode: $XCODE_VERSION"

# [SHELL] 检查并安装 XcodeGen (项目生成工具)
if ! which xcodegen >/dev/null 2>&1; then
    echo "正在安装 XcodeGen..."
    if which brew >/dev/null 2>&1; then
        brew install xcodegen
    else
        echo "⚠️ Homebrew 未安装，将使用手动方式创建项目"
    fi
fi

# [SHELL] 检查 Git
git --version || { echo "❌ Git 未安装"; exit 1; }

# [SHELL] 检查并安装 Fastlane (Phase 9+ 需要)
if ! which fastlane >/dev/null 2>&1; then
    echo "正在安装 Fastlane..."
    gem install fastlane -NV 2>/dev/null || echo "⚠️ Fastlane 安装失败 (Phase 11 可用 xcodebuild 替代)"
fi
```

### Step 0.1 — 创建项目目录 & 工程文件

> **[SHELL]** + **[WRITE]** 创建完整可编译的 Xcode 项目

```bash
# [SHELL] 定义项目变量 — 持久化到文件，全 SOP 所有 bash 块 source 此文件
PROJECT_NAME="MyAwesomeApp"
BUNDLE_ID="com.example.myawesomeapp"
DISPLAY_NAME="我的App"
DEPLOYMENT_TARGET="17.0"

# 持久化项目变量
cat > /tmp/sop_project.env << PROJEOF
PROJECT_NAME="$PROJECT_NAME"
BUNDLE_ID="$BUNDLE_ID"
DISPLAY_NAME="$DISPLAY_NAME"
DEPLOYMENT_TARGET="$DEPLOYMENT_TARGET"
PROJECT_DIR="/Users/xurui/Projects/SOP/$PROJECT_NAME"
PROJEOF
echo "✅ 项目变量已持久化到 /tmp/sop_project.env"

# [SHELL] 创建完整目录结构
cd /Users/xurui/Projects/SOP
mkdir -p ${PROJECT_NAME}
cd ${PROJECT_NAME}

# 创建源码目录
mkdir -p ${PROJECT_NAME}/App
mkdir -p ${PROJECT_NAME}/Models
mkdir -p ${PROJECT_NAME}/ViewModels
mkdir -p ${PROJECT_NAME}/Views/Components
mkdir -p ${PROJECT_NAME}/Services
mkdir -p ${PROJECT_NAME}/Utilities
mkdir -p ${PROJECT_NAME}/Resources
mkdir -p ${PROJECT_NAME}Tests
mkdir -p ${PROJECT_NAME}UITests
mkdir -p Prototype/Screens
mkdir -p Prototype/Components
mkdir -p fastlane/metadata/zh-Hans
```

### Step 0.2 — 生成 Xcode 项目配置 (XcodeGen)

> **[WRITE]** 创建 `project.yml` → **[SHELL]** 运行 `xcodegen generate`

**创建 `${PROJECT_NAME}/project.yml`** — Claude Code 在写入前先获取 TEAM_ID:

```bash
# [SHELL] 动态获取 TEAM_ID 用于 project.yml
TEAM_ID=$(grep TEAM_ID /tmp/sop_simulator.env 2>/dev/null | cut -d'"' -f2)
if [ -z "$TEAM_ID" ]; then
    TEAM_ID="__CHANGE_ME__"  # 用户需手动填入
fi
echo "TEAM_ID=$TEAM_ID"
```

```yaml
# [WRITE] 项目定义文件 (将被写入 ${PROJECT_NAME}/project.yml)
name: ${PROJECT_NAME}
options:
  bundleIdPrefix: com.example
  deploymentTarget:
    iOS: "${DEPLOYMENT_TARGET}"
  xcodeVersion: "16.0"
  generateEmptyDirectories: true
  developmentLanguage: zh-Hans

settings:
  base:
    SWIFT_VERSION: "5.10"
    TARGETED_DEVICE_FAMILY: "1"
    ENABLE_USER_SCRIPT_SANDBOXING: "NO"

targets:
  ${PROJECT_NAME}:
    type: application
    platform: iOS
    sources:
      - path: ${PROJECT_NAME}
        type: group
    settings:
      base:
        PRODUCT_BUNDLE_IDENTIFIER: ${BUNDLE_ID}
        INFOPLIST_FILE: ${PROJECT_NAME}/Resources/Info.plist
        ASSETCATALOG_COMPILER_APPICON_NAME: AppIcon
        ENABLE_PREVIEWS: YES
        DEVELOPMENT_TEAM: ${TEAM_ID}    # 自动检测或用户填入
        CODE_SIGN_STYLE: Automatic
        PRODUCT_NAME: ${DISPLAY_NAME}
    preBuildScripts:
      - name: "SwiftLint"
        script: |
          if which swiftlint >/dev/null; then
            swiftlint --quiet
          fi
        basedOnDependencyAnalysis: false

  ${PROJECT_NAME}Tests:
    type: bundle.unit-test
    platform: iOS
    sources:
      - path: ${PROJECT_NAME}Tests
    dependencies:
      - target: ${PROJECT_NAME}
    settings:
      base:
        PRODUCT_BUNDLE_IDENTIFIER: ${BUNDLE_ID}.tests
        GENERATE_INFOPLIST_FILE: YES

  ${PROJECT_NAME}UITests:
    type: bundle.ui-testing
    platform: iOS
    sources:
      - path: ${PROJECT_NAME}UITests
    dependencies:
      - target: ${PROJECT_NAME}
    settings:
      base:
        PRODUCT_BUNDLE_IDENTIFIER: ${BUNDLE_ID}.uitests
        GENERATE_INFOPLIST_FILE: YES
```

**执行项目生成**:

```bash
# [SHELL] 生成 .xcodeproj
cd /Users/xurui/Projects/SOP/${PROJECT_NAME}

if which xcodegen >/dev/null 2>&1; then
    xcodegen generate --spec project.yml
    echo "✅ Xcode 项目已生成"

    # [SHELL] 首次构建验证 — 确保项目可编译
    xcodebuild build \
      -project ${PROJECT_NAME}.xcodeproj \
      -scheme ${PROJECT_NAME} \
      -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
      2>&1 | tail -5
else
    echo "⚠️ XcodeGen 不可用。请手动创建 Xcode 项目:"
    echo "  1. 打开 Xcode → File → New → Project"
    echo "  2. 选择 iOS → App → SwiftUI"
    echo "  3. Product Name: ${PROJECT_NAME}"
    echo "  4. 保存到: /Users/xurui/Projects/SOP/${PROJECT_NAME}"
    echo "  5. 创建后回复 '项目已创建' 继续"
    # [DIALOG] 等待用户确认
fi
```

### Step 0.3 — 创建 Assets 目录

> **[WRITE]** 创建 Asset Catalog 和基础资源

```bash
# [SHELL] 如果 XcodeGen 未自动创建 Assets.xcassets，手动创建
ASSETS_DIR="${PROJECT_NAME}/Resources/Assets.xcassets"
if [ ! -d "$ASSETS_DIR" ]; then
    mkdir -p "$ASSETS_DIR"
fi

# [WRITE] 创建 AccentColor (HIG: 支持 Light/Dark/HighContrast)
mkdir -p "${ASSETS_DIR}/AccentColor.colorset"
cat > "${ASSETS_DIR}/AccentColor.colorset/Contents.json" << 'EOF'
{
  "colors" : [
    {
      "color" : {
        "color-space" : "srgb",
        "components" : {
          "alpha" : "1.000",
          "blue" : "1.000",
          "green" : "0.478",
          "red" : "0.000"
        }
      },
      "idiom" : "universal"
    },
    {
      "appearances" : [
        {
          "appearance" : "luminosity",
          "value" : "dark"
        }
      ],
      "color" : {
        "color-space" : "srgb",
        "components" : {
          "alpha" : "1.000",
          "blue" : "1.000",
          "green" : "0.518",
          "red" : "0.039"
        }
      },
      "idiom" : "universal"
    }
  ],
  "info" : {
    "author" : "xcode",
    "version" : 1
  }
}
EOF
echo "✅ AccentColor 已配置 (Light + Dark 变体)"
fi

# [WRITE] 创建 LaunchBackground 颜色 (HIG: 启动屏背景与首页一致)
mkdir -p "${ASSETS_DIR}/LaunchBackground.colorset"
cat > "${ASSETS_DIR}/LaunchBackground.colorset/Contents.json" << 'EOF'
{
  "colors" : [
    {
      "color" : { "color-space" : "srgb", "components" : { "alpha" : "1.000", "blue" : "0.969", "green" : "0.949", "red" : "0.949" } },
      "idiom" : "universal"
    },
    {
      "appearances" : [ { "appearance" : "luminosity", "value" : "dark" } ],
      "color" : { "color-space" : "srgb", "components" : { "alpha" : "1.000", "blue" : "0.118", "green" : "0.110", "red" : "0.110" } },
      "idiom" : "universal"
    }
  ],
  "info" : { "author" : "xcode", "version" : 1 }
}
EOF
echo "✅ LaunchBackground 已配置 (Light + Dark 变体, 与系统背景一致)"

# [SHELL] 也是创建 AppIcon 占位目录
mkdir -p "${ASSETS_DIR}/AppIcon.appiconset"

# [WRITE] 创建空的 Contents.json (图标在 Phase 9 填充)
cat > "${ASSETS_DIR}/AppIcon.appiconset/Contents.json" << 'EOF'
{
  "images" : [],
  "info" : { "author" : "xcode", "version" : 1 }
}
EOF
```

### Step 0.4 — 初始化 Git 仓库

```bash
# [SHELL] + [GIT]
cd /Users/xurui/Projects/SOP/${PROJECT_NAME}
git init
git checkout -b main

# [WRITE] 创建 .gitignore
cat > .gitignore << 'EOF'
*.xcuserdata
*.xcworkspace
xcuserdata/
DerivedData/
*.moved-aside
*.pbxuser
*.mode1v3
*.mode2v3
*.perspectivev3
*.hmap
*.ipa
*.dSYM.zip
*.dSYM
Pods/
.build/
.swiftpm/
fastlane/report.xml
fastlane/Preview.html
fastlane/screenshots/**/*.png
fastlane/test_output
.DS_Store
*.orig
EOF

# [SHELL] 首次编译验证 (确保骨架可编译)
xcodebuild build \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  -quiet 2>&1 || echo "⚠️ 首次编译可能有预期错误(缺少入口文件)，将在后续步骤解决"

# [GIT]
git add -A
git commit -m "Initial project structure with XcodeGen"
```

> **Phase 0 验收条件**:
> - [ ] `ls *.xcodeproj` 项目文件存在
> - [ ] `git log --oneline` 有初始提交
> - [ ] 目录结构完整 (App/Models/ViewModels/Views/Services/Utilities/Resources)
> - [ ] Assets.xcassets 包含 AccentColor
> - [ ] `${SIMULATOR_ID}` 可用模拟器已检测并启动

---

### 全局约定: 模拟器管理 & xcodebuild 目标规范

> **所有后续 `xcodebuild` 命令使用统一的模拟器目标。Claude Code 在每次执行前先 source 环境变量。**

```bash
# [SHELL] 加载全局变量 — 在 SOP 任何 xcodebuild 调用前执行
source /tmp/sop_simulator.env 2>/dev/null

# [SHELL] 确保模拟器已启动
xcrun simctl boot "$SIMULATOR_ID" 2>/dev/null || echo "模拟器已启动"

# xcodebuild 统一目标格式 (全 SOP 使用):
XCODE_DESTINATION="platform=iOS Simulator,id=$SIMULATOR_ID"
# 或: XCODE_DESTINATION="platform=iOS Simulator,name=$SIMULATOR_NAME"
```

---

## Phase 1: 产品需求澄清 (0:30 - 1:00)

> **[DIALOG]** + **[WRITE]** 需求访谈 → 输出 SPECS.md

### Step 1.1 — 需求确认对话

> **[DIALOG]** 向用户提出结构化问题，获得明确回答前不得继续。

```
我需要确认以下信息来构建你的 App：

1. App 核心功能 (一句话描述):
   [示例: "一个极简的每日记账 App"]
   [示例: "一个 AI 聊天客户端"]
   [示例: "一个番茄钟专注计时器"]

2. 目标用户群体:
   [ ] 普通消费者 (B2C)
   [ ] 企业用户 (B2B)
   [ ] 开发者工具

3. 盈利模式:
   [ ] 免费 + 广告
   [ ] 免费 + 内购 (IAP)
   [ ] 付费下载
   [ ] 订阅制 (Auto-Renewable Subscription)
   [ ] 免费 (不盈利/开源)

4. 是否需要后端服务:
   [ ] 不需要 (纯本地 App)
   [ ] 需要 (使用 CloudKit/Firebase/自建后端)

5. 是否需要用户系统:
   [ ] 不需要
   [ ] Sign in with Apple (推荐)
   [ ] 邮箱注册

6. 核心数据是否需要云端同步:
   [ ] 仅本地存储 (CoreData/SwiftData/UserDefaults)
   [ ] iCloud 同步
   [ ] 自有后端同步

7. UI 风格偏好:
   [ ] 系统原生风格 (Cupertino)
   [ ] 极简/现代
   [ ] Material Design

8. 是否支持深色模式:
   [ ] 是 (推荐)
   [ ] 否
```

### Step 1.2 — 输出产品规格书

> **[GENERATE]** + **[WRITE]** + **[DIALOG]** 整理用户回答，生成 `SPECS.md` 并请用户确认:

```markdown
# 产品规格书 — ${PROJECT_NAME}

## 核心功能
- [ ] 功能1: xxx
- [ ] 功能2: xxx
- [ ] 功能3: xxx

## 技术选型
- UI: SwiftUI
- 架构: MVVM
- 本地存储: ${STORAGE_SOLUTION}
- 后端: ${BACKEND_SOLUTION}
- 认证: ${AUTH_SOLUTION}

## 屏幕清单
1. 启动屏 (LaunchScreen)
2. 主页 (HomeView)
3. 详情页 (DetailView)
4. 设置页 (SettingsView)

## 数据模型
- Model1: xxx
- Model2: xxx

## App Store 信息
- 分类: ${CATEGORY}
- 年龄分级: 4+
- 价格: ${PRICE}
```

> ⚠️ **用户必须回复 "确认" 或修改意见，否则停止执行。**

---

## Phase 1.2: PRD 产品需求文档 (1:00 - 1:30)

> **[GENERATE]** + **[WRITE]** + **[DIALOG]** 基于 SPECS.md 生成完整 PRD，作为高保真原型的唯一输入

### PRD 输出物

Claude Code 生成 `PRD.md`，包含以下章节。**PRD 不确认，不进原型**。

````markdown
# 产品需求文档 (PRD) — ${PROJECT_NAME}

> 版本: 1.0 | 日期: yyyy-mm-dd | 基于: SPECS.md
> **本文档是 Phase 1.5 原型设计的唯一输入，原型不得偏离 PRD 定义。**

---

## 1. 产品概述

### 1.1 产品定位
- 一句话描述: [从 SPECS.md]
- 目标用户: [从 SPECS.md]
- 核心价值: [解决什么痛点]

### 1.2 用户画像 (2-3 个)
| 画像 | 年龄 | 场景 | 痛点 | 期望 |
|------|------|------|------|------|
| [画像1] | [25-35] | [通勤时快速记录] | [忘记重要事项] | [3秒内完成记录] |
| [画像2] | [...] | [...] | [...] | [...] |

---

## 2. 功能规格

### 2.1 功能清单 (MoSCoW 优先级)

| 优先级 | 功能 | 描述 | 输入 | 输出 |
|--------|------|------|------|------|
| **M** (Must) | [功能1] | [用户做什么] | [用户输入什么] | [系统返回什么] |
| **M** | [功能2] | [...] | [...] | [...] |
| **S** (Should) | [功能3] | [...] | [...] | [...] |
| **C** (Could) | [功能4] | [...] | [...] | [...] |

### 2.2 详细功能描述

#### 功能: [功能1名称]

```
触发条件: [用户如何触发]
前置条件: [需要什么状态]
基本流程:
  1. 用户 [操作]
  2. 系统 [响应]
  3. 用户 [下一步操作]
  4. 系统 [最终结果]
异常流程:
  - 网络不可用 → [离线处理/提示]
  - 输入为空 → [验证提示]
  - 数据冲突 → [冲突解决策略]
```

---

## 3. 用户流程 (User Flow)

### 3.1 核心流程

```
[启动App] → [首页] → [点击添加] → [填写表单] → [保存] → [返回首页(刷新)]
                            ↘ [取消] → [返回首页]
```

### 3.2 页面跳转图

```
Index(首页) ──→ DetailPage(详情/编辑)
   │                │
   ├──→ SettingsPage(设置)
   │
   └──→ [筛选/搜索] (内联)
```

---

## 4. 数据模型

### 4.1 实体定义

| 实体 | 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| TaskItem | id | String | ✅ | 唯一标识 |
| | title | String | ✅ | 标题 (1-100字符) |
| | priority | Enum | ✅ | LOW/MEDIUM/HIGH/URGENT |
| | isCompleted | Bool | ✅ | 默认 false |
| | dueDate | Date | ❌ | 截止日期 |
| | createdAt | Date | ✅ | 自动生成 |

### 4.2 数据流

```
[用户输入] → ViewModel.validate() → Model → DatabaseService → RelationalStore
                                                              ↓
[UI 刷新] ← @State/refreshFlag ← ViewModel.loadTasks() ← query
```

---

## 5. 屏幕规格 (Screen Specs)

> **此章节是 Phase 1.5 原型设计的直接输入。每个屏幕必须逐项实现。**

### 5.1 首页 (Index)

| 区域 | 组件 | 数据源 | 交互 |
|------|------|--------|------|
| 顶部标题 | Text("我的任务") | 静态 | 无 |
| 搜索栏 | Search() | → viewModel.searchText | 输入即搜索 |
| 筛选标签 | 水平Scroll+FilterPill×4 | → viewModel.selectedFilter | 点击切换 |
| 任务列表 | List+LazyForEach | ← viewModel.filteredTasks | 点击→详情, 左滑→删除 |
| 空状态 | EmptyStateView | 列表为空时显示 | 引导用户添加 |
| 添加按钮 | Button("添加任务") | — | → DetailPage(新建) |

**状态覆盖**:
- [ ] 加载中: LoadingProgress
- [ ] 空列表: 空状态引导
- [ ] 有数据: 任务列表
- [ ] 搜索无结果: "未找到 'xxx'"
- [ ] 网络错误: 错误提示+重试

### 5.2 详情页 (DetailPage)

| 区域 | 组件 | 数据源 | 交互 |
|------|------|--------|------|
| 导航栏 | ←取消 / 标题 / 保存→ | — | 取消=返回, 保存=提交 |
| 标题输入 | TextInput | ↔ task.title | 必填, 1-100字符 |
| 描述输入 | TextArea | ↔ task.description | 可选 |
| 优先级选择 | 4个Button(低中高紧急) | ↔ task.priority | 单选 |
| 截止日期 | TextInput/DatePicker | ↔ task.dueDate | 可选 |

### 5.3 设置页 (SettingsPage)

| 区域 | 组件 | 数据源 | 交互 |
|------|------|--------|------|
| 触觉反馈 | Toggle | ↔ AppStorage | 开关切换 |
| 关于 | Text+版本号 | 静态 | 无 |

---

## 6. 非功能性需求

| 类别 | 要求 |
|------|------|
| 性能 | 冷启动 < 2s, 列表滚动 60fps |
| 兼容性 | [iOS 17.0+ / HarmonyOS API 12+] |
| 无障碍 | 所有交互元素有 accessibilityLabel, 触摸目标 ≥ 44pt(iOS)/48vp(鸿蒙) |
| 离线 | 全部数据本地存储, 无网络不影响使用 |
| 隐私 | 不收集个人数据, 不需要网络权限 |

---

## 7. 验收标准 (Acceptance Criteria)

```
□ AC-1: 用户可在 3 步内完成添加任务 (首页→添加→输入→保存)
□ AC-2: 筛选切换后列表立即更新 (< 0.3s)
□ AC-3: 搜索支持模糊匹配, 结果 < 0.5s
□ AC-4: 已完成任务有删除线+绿色图标
□ AC-5: 无网络时所有功能正常 (纯本地)
□ AC-6: 空列表时显示引导文案
□ AC-7: 删除有确认弹窗
□ AC-8: 支持 Dynamic Type / 系统字体缩放
````

> ⚠️ **用户必须回复 "PRD 确认" 或修改意见，否则停止执行。PRD 确认后方可进入 Phase 1.5 原型设计。**

---

## Phase 1.5: 高保真原型设计 & 设计系统 (1:30 - 2:30)

> **输入**: PRD.md (Phase 1.2) → **输出**: DESIGN_SPECS.md + Prototype/ 文件
> **原型必须 1:1 覆盖 PRD 第 5 章定义的每个屏幕和每个状态。**

> **[GENERATE]** + **[WRITE]** + **[DIALOG]** SwiftUI Previews 作为高保真原型工具。原型代码 = 未来 UI 代码的基础，零浪费。
> **产出物**: `DESIGN_SPECS.md` + `Prototype/` 目录下的完整屏幕原型 SwiftUI 文件

### Step 1.5.1 — 设计系统定义 (Design System)

> **[RESEARCH]** + **[GENERATE]** + **[WRITE]** 根据 SPECS.md 定义完整设计系统，作为后续所有 UI 开发的唯一真相源。
> Claude Code 需查阅 HIG_KNOWLEDGE_BASE.md 中的色彩/字体/间距规范确保合规。

**创建 `DESIGN_SPECS.md`**:

````markdown
# 设计规格书 — ${PROJECT_NAME}

> 本文档由 Claude Code 根据产品需求自动生成，是后续 UI 开发的唯一设计参考。
> 所有间距、颜色、字号、组件规格均在此定义，不得偏离。

---

## 1. 品牌标识

| 属性 | 值 |
|------|-----|
| App 名称 | ${DISPLAY_NAME} |
| 品牌基调 | [极简专业 / 活泼温暖 / 科技感 / ...] |
| 品牌关键词 | [简洁, 高效, 可信赖, ...] |
| 强调色 | #007AFF (iOS Blue) |
| 图标风格 | SF Symbols / 自定义 |

## 2. 色彩系统

### 主色调
```
Primary:      #007AFF (Light) / #0A84FF (Dark)
Secondary:    #5856D6 (Light) / #5E5CE6 (Dark)
```

### 语义色
```
Success:      #34C759    (系统绿)
Warning:      #FF9500    (系统橙)
Error:        #FF3B30    (系统红)
Info:         #007AFF    (系统蓝)
```

### 中性色
```
Background Primary:    #FFFFFF (Light) / #000000 (Dark)
Background Secondary:  #F2F2F7 (Light) / #1C1C1E (Dark)
Background Tertiary:   #FFFFFF (Light) / #2C2C2E (Dark)
Text Primary:          #000000 (Light) / #FFFFFF (Dark)
Text Secondary:        #3C3C43 60% (Light) / #EBEBF5 60% (Dark)
Text Tertiary:         #3C3C43 30% (Light) / #EBEBF5 30% (Dark)
Separator:             #3C3C43 20% (Light) / #545458 60% (Dark)
```

### WCAG 对比度验证
| 组合 | Light 模式 | Dark 模式 | 标准 |
|------|-----------|-----------|------|
| Text Primary on Bg Primary | 21.0:1 ✅ | 21.0:1 ✅ | ≥4.5:1 |
| Text Secondary on Bg Primary | 7.5:1 ✅ | 7.5:1 ✅ | ≥4.5:1 |
| Accent on Bg Primary | 4.0:1 ⚠️ | 4.0:1 ⚠️ | ≥3:1 (≥18pt) |

## 3. 字体系统 (Typography Scale)

| 样式 | 字号 | 字重 | 行高 | Dynamic Type | 用途 |
|------|------|------|------|-------------|------|
| largeTitle | 34pt | Bold | 41pt | ✅ | 主标题 |
| title1 | 28pt | Regular | 34pt | ✅ | 页面标题 |
| title2 | 22pt | Regular | 28pt | ✅ | 二级标题 |
| title3 | 20pt | Regular | 25pt | ✅ | 三级标题 |
| headline | 17pt | Semibold | 22pt | ✅ | 段落标题 |
| body | 17pt | Regular | 22pt | ✅ | 正文 |
| callout | 16pt | Regular | 21pt | ✅ | 标注 |
| subhead | 15pt | Regular | 20pt | ✅ | 副标题 |
| footnote | 13pt | Regular | 18pt | ✅ | 脚注 |
| caption1 | 12pt | Regular | 16pt | ✅ | 说明文字 |
| caption2 | 11pt | Regular | 13pt | ✅ | 最小文字 |

> **HIG 规则**: 最小字号 11pt；避免 Light/Thin/Ultralight；正文用 Regular/Medium/Semibold/Bold

## 4. 间距系统 (8pt Grid)

```
├─ xs: 4pt   (半格 — 紧凑关联元素)
├─ sm: 8pt   (1格 — 紧密关联元素)
├─ md: 16pt  (2格 — 标准间距)
├─ lg: 24pt  (3格 — 段落间距)
├─ xl: 32pt  (4格 — 区块间距)
└─ 2xl: 48pt (6格 — 页面级间距)
```

## 5. 圆角 & 阴影

| 元素 | 圆角 | 说明 |
|------|------|------|
| 按钮 (小) | 8pt | capsule 或 rounded |
| 按钮 (大) | 12pt | 主操作按钮 |
| 卡片 | 12pt | 内容卡片 |
| Sheet/Modal | 16pt (顶部) | 模态视图 |
| 输入框 | 8pt | 文本输入 |
| 图标容器 | 10pt | App 图标圆角 |

| 阴影层级 | Y偏移 | 模糊 | 透明度 | 用途 |
|---------|-------|------|--------|------|
| Level 0 | 0 | 0 | 0 | 平坦 (无阴影) |
| Level 1 | 1pt | 4pt | 8% | 卡片轻微浮起 |
| Level 2 | 2pt | 8pt | 12% | 导航栏/底部栏 |
| Level 3 | 4pt | 16pt | 16% | 模态/弹出层 |

## 6. 组件规范

### 按钮
```
Primary Button:
  Background: Accent Color
  Text: White, Semibold, 17pt
  Height: 50pt, Corner: 12pt
  Padding H: 24pt, Min Width: fill

Secondary Button:
  Background: Accent 15%
  Text: Accent, Semibold, 17pt
  Height: 50pt, Corner: 12pt

Tertiary Button:
  Background: transparent
  Text: Accent, Regular, 17pt
  Height: 44pt
```

### 列表行
```
List Row (Standard):
  Height: 44pt (minimum)
  Leading: icon/image 40×40pt + 12pt gap
  Trailing: chevron.right / toggle / badge
  Separator: 1pt, system separator color
  Padding H: 16pt (content), 20pt (separator)
```

### 导航栏
```
Navigation Bar:
  Height: 44pt (standard) / 96pt (large title)
  Background: Material.regular (Liquid Glass)
  Title: largeTitle / title1
  Back button: 系统自动 (chevron + 上级标题)
```

### 标签栏
```
Tab Bar:
  Height: 49pt (+ safeArea)
  Background: Material.regular (Liquid Glass)
  Icons: SF Symbols, 25pt
  Labels: caption2, 10pt
  Active: filled icon + Accent Color
  Inactive: outline icon + secondary
```

## 7. 屏幕清单 & 线框图描述

### 7.1 启动屏 (Launch Screen)
```
┌─────────────────────────────┐
│                             │
│       [背景色与首页一致]      │
│                             │
│   (无文字、无Logo、无动画)    │
│                             │
└─────────────────────────────┘
```
> HIG: 启动屏 = 首屏减去动态内容

### 7.2 首页 (Home — 主列表页)
```
┌─────────────────────────────┐
│ 导航栏: "首页" (大标题)  [+] │ ← Material.regular
│─────────────────────────────│
│ [全部] [进行中] [已完成]     │ ← Filter Pills, 水平滚动
│─────────────────────────────│
│  排序: [最新 ▼]     共 N 项  │
│─────────────────────────────│
│ ┌─────────────────────────┐ │
│ │ ○ 任务标题               │ │ ← Checkmark circle
│ │   📅 截止日期            │ │
│ │                 [高]    │ │ ← Priority badge
│ │─────────────────────────│ │
│ │ ● 已完成任务 (删除线)     │ │
│ │─────────────────────────│ │
│ │ ○ 另一个任务             │ │
│ └─────────────────────────┘ │
│─────────────────────────────│
│  [🏠首页]  [🔍发现]  [👤我的] │ ← Tab Bar, Material.regular
└─────────────────────────────┘
```

### 7.3 详情页 (Detail)
```
┌─────────────────────────────┐
│ ← 返回    详情    [编辑]     │ ← Navigation Bar
│─────────────────────────────│
│                             │
│  任务标题 (title1)           │
│  ─────────────────────────  │
│  创建时间: 2026-06-08       │
│  优先级:    [高] Badge      │
│  状态:      已完成 ✓        │
│  截止日期:  2026-06-15      │
│  ─────────────────────────  │
│  详细描述文字...             │
│                             │
│  [标记完成]  [编辑]  [删除]   │ ← 操作按钮组
│                             │
└─────────────────────────────┘
```

[... 为 SPECS.md 中的每个屏幕生成线框图]
````

### Step 1.5.2 — 生成高保真 SwiftUI 原型

> **Claude Code 创建 `Prototype/` 目录，为每个屏幕生成独立的 SwiftUI Preview 文件。这些文件仅包含视觉层，不含业务逻辑，作为设计参考。**

**目录结构**:
```bash
mkdir -p ${PROJECT_NAME}/Prototype/Screens
mkdir -p ${PROJECT_NAME}/Prototype/Components
```

**创建 `Prototype/DesignSystem.swift`** — 原型专用设计系统 (所有屏幕引用此文件):

```swift
// ${PROJECT_NAME}/Prototype/DesignSystem.swift
// 原型设计系统 — 所有原型屏幕的颜色/字体/间距唯一来源
// 后续 Phase 5 UI 开发直接引用或转换此文件

import SwiftUI

// MARK: - Prototype Colors
enum ProtoColor {
    static let primary = Color(hex: "007AFF")
    static let primaryDark = Color(hex: "0A84FF")
    static let secondary = Color(hex: "5856D6")

    static let bgPrimary = Color(.systemBackground)
    static let bgSecondary = Color(.systemGroupedBackground)

    static let textPrimary = Color(.label)
    static let textSecondary = Color(.secondaryLabel)
    static let textTertiary = Color(.tertiaryLabel)

    static let success = Color.green
    static let warning = Color.orange
    static let error = Color.red

    static let separator = Color(.separator)
}

extension Color {
    init(hex: String) {
        let hex = hex.trimmingCharacters(in: CharacterSet.alphanumerics.inverted)
        var int: UInt64 = 0
        Scanner(string: hex).scanHexInt64(&int)
        let a, r, g, b: UInt64
        switch hex.count {
        case 3: (a, r, g, b) = (255, (int >> 8) * 17, (int >> 4 & 0xF) * 17, (int & 0xF) * 17)
        case 6: (a, r, g, b) = (255, int >> 16, int >> 8 & 0xFF, int & 0xFF)
        case 8: (a, r, g, b) = (int >> 24, int >> 16 & 0xFF, int >> 8 & 0xFF, int & 0xFF)
        default: (a, r, g, b) = (255, 0, 0, 0)
        }
        self.init(
            .sRGB,
            red: Double(r) / 255,
            green: Double(g) / 255,
            blue: Double(b) / 255,
            opacity: Double(a) / 255
        )
    }
}

// MARK: - Prototype Typography
enum ProtoFont {
    static let largeTitle = Font.system(.largeTitle, design: .default).weight(.bold)
    static let title1 = Font.system(.title, design: .default)
    static let title2 = Font.system(.title2, design: .default)
    static let title3 = Font.system(.title3, design: .default)
    static let headline = Font.system(.headline, design: .default)
    static let body = Font.system(.body, design: .default)
    static let callout = Font.system(.callout, design: .default)
    static let subhead = Font.system(.subheadline, design: .default)
    static let footnote = Font.system(.footnote, design: .default)
    static let caption1 = Font.system(.caption, design: .default)
    static let caption2 = Font.system(.caption2, design: .default)
}

// MARK: - Prototype Spacing (8pt grid)
enum ProtoSpacing {
    static let xs: CGFloat = 4
    static let sm: CGFloat = 8
    static let md: CGFloat = 16
    static let lg: CGFloat = 24
    static let xl: CGFloat = 32
    static let xxl: CGFloat = 48
}

// MARK: - Prototype Corner Radius
enum ProtoCorner {
    static let sm: CGFloat = 8
    static let md: CGFloat = 12
    static let lg: CGFloat = 16
    static let full: CGFloat = 9999  // capsule
}

// MARK: - Reusable Prototype Components

/// 原型按钮 — 展示按钮的三种样式
enum ProtoButtonStyle {
    case primary, secondary, tertiary
}

struct ProtoButton: View {
    let title: String
    let style: ProtoButtonStyle
    let action: () -> Void

    var body: some View {
        Button(action: action) {
            Text(title)
                .font(ProtoFont.headline)
                .frame(maxWidth: .infinity)
                .frame(height: 50)
        }
        .buttonStyle(ProtoButtonStyleModifier(style: style))
    }
}

struct ProtoButtonStyleModifier: ButtonStyle {
    let style: ProtoButtonStyle

    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .background(background)
            .foregroundColor(foregroundColor)
            .cornerRadius(ProtoCorner.md)
            .opacity(configuration.isPressed ? 0.7 : 1.0)
    }

    private var background: Color {
        switch style {
        case .primary: return ProtoColor.primary
        case .secondary: return ProtoColor.primary.opacity(0.12)
        case .tertiary: return .clear
        }
    }

    private var foregroundColor: Color {
        switch style {
        case .primary: return .white
        case .secondary, .tertiary: return ProtoColor.primary
        }
    }
}

/// 原型列表行
struct ProtoListRow: View {
    let icon: String
    let title: String
    let subtitle: String?
    let trailingText: String?

    init(icon: String, title: String, subtitle: String? = nil, trailingText: String? = nil) {
        self.icon = icon
        self.title = title
        self.subtitle = subtitle
        self.trailingText = trailingText
    }

    var body: some View {
        HStack(spacing: ProtoSpacing.md) {
            Image(systemName: icon)
                .font(.title3)
                .foregroundColor(ProtoColor.primary)
                .frame(width: 28)

            VStack(alignment: .leading, spacing: ProtoSpacing.xs) {
                Text(title)
                    .font(ProtoFont.body)
                    .foregroundColor(ProtoColor.textPrimary)
                if let subtitle = subtitle {
                    Text(subtitle)
                        .font(ProtoFont.caption1)
                        .foregroundColor(ProtoColor.textSecondary)
                }
            }

            Spacer()

            if let trailing = trailingText {
                Text(trailing)
                    .font(ProtoFont.callout)
                    .foregroundColor(ProtoColor.textSecondary)
            }

            Image(systemName: "chevron.right")
                .font(.caption)
                .foregroundColor(ProtoColor.textTertiary)
        }
        .padding(.vertical, ProtoSpacing.sm)
    }
}

/// 原型标签/徽章
struct ProtoBadge: View {
    let text: String
    let color: Color

    var body: some View {
        Text(text)
            .font(ProtoFont.caption2)
            .fontWeight(.medium)
            .padding(.horizontal, ProtoSpacing.sm)
            .padding(.vertical, ProtoSpacing.xs)
            .background(color.opacity(0.15))
            .foregroundColor(color)
            .cornerRadius(ProtoCorner.sm)
    }
}

/// 原型空状态
struct ProtoEmptyState: View {
    let icon: String
    let title: String
    let message: String

    var body: some View {
        VStack(spacing: ProtoSpacing.lg) {
            Spacer()
            Image(systemName: icon)
                .font(.system(size: 56))
                .foregroundColor(ProtoColor.textTertiary)
            Text(title)
                .font(ProtoFont.headline)
            Text(message)
                .font(ProtoFont.subhead)
                .foregroundColor(ProtoColor.textSecondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal, ProtoSpacing.xxl)
            Spacer()
        }
    }
}

// MARK: - Preview Helper

/// Dark Mode 双预览包装器
struct ProtoPreview<Content: View>: View {
    let title: String
    let content: Content
    @State private var colorScheme: ColorScheme = .light

    init(_ title: String, @ViewBuilder content: () -> Content) {
        self.title = title
        self.content = content()
    }

    var body: some View {
        VStack(spacing: 0) {
            // 模式切换栏
            HStack {
                Text("📱 \(title)")
                    .font(ProtoFont.headline)
                Spacer()
                Picker("Mode", selection: $colorScheme) {
                    Text("☀️ 亮色").tag(ColorScheme.light)
                    Text("🌙 暗色").tag(ColorScheme.dark)
                }
                .pickerStyle(.segmented)
                .frame(width: 160)
            }
            .padding()
            .background(ProtoColor.bgSecondary)

            // 原型内容
            content
                .preferredColorScheme(colorScheme)
        }
        .background(ProtoColor.bgPrimary)
    }
}
```

### Step 1.5.3 — 逐屏幕生成高保真原型

> **Claude Code 为 SPECS.md 中列出的每个屏幕创建独立原型文件。每个文件包含完整布局、假数据、亮/暗模式切换。**

**创建 `Prototype/Screens/HomeScreen.swift`** (示例 — 为每个屏幕生成类似文件):

```swift
// ${PROJECT_NAME}/Prototype/Screens/HomeScreen.swift
// 首页原型 — 高保真视觉稿
// 后续 Phase 5 的 HomeView.swift 按此原型实现

import SwiftUI

struct HomeScreenPrototype: View {
    @State private var selectedFilter = 0
    @State private var sortOption = 0

    let filters = ["全部", "进行中", "已完成", "高优先级"]
    let sortOptions = ["最新", "最早", "优先级"]

    // 假数据
    struct MockTask: Identifiable {
        let id = UUID()
        let title: String
        let priority: String
        let priorityColor: Color
        let dueDate: String?
        let isCompleted: Bool
    }

    let tasks: [MockTask] = [
        .init(title: "完成 Q2 产品需求文档", priority: "高", priorityColor: .orange, dueDate: "6月12日", isCompleted: false),
        .init(title: "设计新版本首页布局", priority: "中", priorityColor: .blue, dueDate: "6月15日", isCompleted: false),
        .init(title: "代码审查 PR #234", priority: "紧急", priorityColor: .red, dueDate: "6月9日", isCompleted: false),
        .init(title: "更新 App Store 截图", priority: "低", priorityColor: .secondary, dueDate: nil, isCompleted: true),
        .init(title: "修复首页加载崩溃 Bug", priority: "紧急", priorityColor: .red, dueDate: "6月10日", isCompleted: false),
    ]

    var body: some View {
        ProtoPreview("首页 - 任务列表") {
            VStack(spacing: 0) {
                // 筛选标签
                ScrollView(.horizontal, showsIndicators: false) {
                    HStack(spacing: ProtoSpacing.sm) {
                        ForEach(Array(filters.enumerated()), id: \.offset) { index, filter in
                            Text(filter)
                                .font(ProtoFont.subhead)
                                .fontWeight(selectedFilter == index ? .semibold : .regular)
                                .padding(.horizontal, ProtoSpacing.md)
                                .padding(.vertical, ProtoSpacing.sm)
                                .background(
                                    Capsule()
                                        .fill(selectedFilter == index ? ProtoColor.primary : Color(.systemGray6))
                                )
                                .foregroundColor(selectedFilter == index ? .white : ProtoColor.textPrimary)
                                .onTapGesture { selectedFilter = index }
                        }
                    }
                    .padding(.horizontal, ProtoSpacing.md)
                    .padding(.vertical, ProtoSpacing.sm)
                }

                Divider()

                // 排序栏
                HStack {
                    Text("排序方式")
                        .font(ProtoFont.caption1)
                        .foregroundColor(ProtoColor.textSecondary)
                    Picker("", selection: $sortOption) {
                        ForEach(Array(sortOptions.enumerated()), id: \.offset) { i, opt in
                            Text(opt).tag(i)
                        }
                    }
                    .pickerStyle(.menu)
                    Spacer()
                    Text("\(tasks.count) 项")
                        .font(ProtoFont.caption1)
                        .foregroundColor(ProtoColor.textSecondary)
                }
                .padding(.horizontal, ProtoSpacing.md)
                .padding(.vertical, ProtoSpacing.sm)

                // 任务列表
                List {
                    ForEach(tasks) { task in
                        HStack(spacing: ProtoSpacing.md) {
                            // 完成状态图标
                            Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
                                .font(.title2)
                                .foregroundColor(task.isCompleted ? ProtoColor.success : ProtoColor.textTertiary)

                            // 内容
                            VStack(alignment: .leading, spacing: ProtoSpacing.xs) {
                                Text(task.title)
                                    .font(ProtoFont.body)
                                    .strikethrough(task.isCompleted)
                                    .foregroundColor(task.isCompleted ? ProtoColor.textTertiary : ProtoColor.textPrimary)

                                if let dueDate = task.dueDate {
                                    HStack(spacing: ProtoSpacing.xs) {
                                        Image(systemName: "calendar")
                                            .font(.caption2)
                                        Text("截止 \(dueDate)")
                                            .font(ProtoFont.caption1)
                                    }
                                    .foregroundColor(ProtoColor.warning)
                                }
                            }

                            Spacer()

                            // 优先级标签
                            ProtoBadge(text: task.priority, color: task.priorityColor)
                        }
                        .padding(.vertical, ProtoSpacing.xs)
                    }
                }
                .listStyle(.plain)
            }
        }
    }
}

#Preview("首页原型") {
    HomeScreenPrototype()
}
```

**创建 `Prototype/Screens/DetailScreen.swift`**:
```swift
// ${PROJECT_NAME}/Prototype/Screens/DetailScreen.swift
// 详情页原型 — 高保真视觉稿

import SwiftUI

struct DetailScreenPrototype: View {
    var body: some View {
        ProtoPreview("详情页") {
            ScrollView {
                VStack(alignment: .leading, spacing: ProtoSpacing.lg) {
                    // 标题区
                    VStack(alignment: .leading, spacing: ProtoSpacing.sm) {
                        Text("完成 Q2 产品需求文档")
                            .font(ProtoFont.title1)
                            .foregroundColor(ProtoColor.textPrimary)

                        HStack(spacing: ProtoSpacing.md) {
                            ProtoBadge(text: "高", color: .orange)
                            ProtoBadge(text: "进行中", color: ProtoColor.primary)
                        }
                    }
                    .padding(.horizontal, ProtoSpacing.md)

                    Divider()
                        .padding(.horizontal, ProtoSpacing.md)

                    // 信息区
                    VStack(spacing: ProtoSpacing.md) {
                        ProtoDetailRow(icon: "calendar", label: "截止日期", value: "2026年6月12日")
                        ProtoDetailRow(icon: "clock", label: "创建时间", value: "2026年6月8日 14:30")
                        ProtoDetailRow(icon: "flag", label: "优先级", value: "高")
                    }
                    .padding(.horizontal, ProtoSpacing.md)

                    Divider()
                        .padding(.horizontal, ProtoSpacing.md)

                    // 描述区
                    VStack(alignment: .leading, spacing: ProtoSpacing.sm) {
                        Text("描述")
                            .font(ProtoFont.headline)
                            .foregroundColor(ProtoColor.textPrimary)
                        Text("需要包含产品背景、目标用户、核心功能流程、非功能性需求等内容。文档完成后需要在团队内部进行评审。")
                            .font(ProtoFont.body)
                            .foregroundColor(ProtoColor.textSecondary)
                            .lineSpacing(4)
                    }
                    .padding(.horizontal, ProtoSpacing.md)

                    Spacer(minLength: ProtoSpacing.xl)

                    // 操作按钮
                    VStack(spacing: ProtoSpacing.md) {
                        ProtoButton(title: "标记为已完成", style: .primary) {}
                        ProtoButton(title: "编辑任务", style: .secondary) {}
                        ProtoButton(title: "删除任务", style: .tertiary) {}
                            .foregroundColor(ProtoColor.error)
                    }
                    .padding(.horizontal, ProtoSpacing.md)
                }
                .padding(.vertical, ProtoSpacing.lg)
            }
        }
    }
}

struct ProtoDetailRow: View {
    let icon: String
    let label: String
    let value: String

    var body: some View {
        HStack(spacing: ProtoSpacing.md) {
            Image(systemName: icon)
                .font(.body)
                .foregroundColor(ProtoColor.textSecondary)
                .frame(width: 24)

            Text(label)
                .font(ProtoFont.body)
                .foregroundColor(ProtoColor.textSecondary)

            Spacer()

            Text(value)
                .font(ProtoFont.body)
                .foregroundColor(ProtoColor.textPrimary)
        }
    }
}

#Preview("详情页原型") {
    DetailScreenPrototype()
}
```

**创建 `Prototype/Screens/SettingsScreen.swift`**:
```swift
// ${PROJECT_NAME}/Prototype/Screens/SettingsScreen.swift
import SwiftUI

struct SettingsScreenPrototype: View {
    @State private var hapticEnabled = true
    @State private var defaultPriority = 1

    var body: some View {
        ProtoPreview("设置") {
            Form {
                Section {
                    ProtoListRow(icon: "bell.fill", title: "通知设置", trailingText: "开启")
                    ProtoListRow(icon: "paintbrush.fill", title: "外观", trailingText: "跟随系统")
                    ProtoListRow(icon: "hand.rays.fill", title: "触觉反馈", trailingText: hapticEnabled ? "开启" : "关闭")
                } header: {
                    Text("偏好设置")
                }

                Section {
                    ProtoListRow(icon: "info.circle", title: "关于", trailingText: "v1.0.0")
                    ProtoListRow(icon: "hand.raised.fill", title: "隐私政策")
                    ProtoListRow(icon: "doc.text.fill", title: "服务条款")
                }
            }
            .scrollContentBackground(.hidden)
        }
    }
}

#Preview("设置页原型") {
    SettingsScreenPrototype()
}
```

> **Claude Code 执行**: 为 SPECS.md 中**每一个**屏幕创建对应的原型文件。每个文件必须包含:
> 1. `ProtoPreview` 包装 (支持亮/暗模式实时切换)
> 2. 完整布局 (所有 UI 元素、间距、颜色)
> 3. 逼真的假数据 (不少于 3-5 条，覆盖各种状态)
> 4. 所有交互状态 (空态、正常态、完成态、错误态)
> 5. `#Preview` 宏 (可在 Xcode Canvas 中直接预览)

### Step 1.5.4 — 原型确认 & 设计交接

```bash
# Claude Code 列出所有原型文件
echo "=== 已生成的高保真原型 ==="
find ${PROJECT_NAME}/Prototype -name "*.swift" | sort | while read f; do
    echo "📱 $(basename $f .swift)"
done

# 输出原型预览说明
echo "
请在 Xcode 中打开以下文件查看高保真原型:
$(find ${PROJECT_NAME}/Prototype -name '*.swift' | sed 's/^/  - /')

操作方式:
1. 打开任一原型文件
2. Xcode → Editor → Canvas (⌥⌘↵)
3. 使用预览顶部的亮色/暗色切换按钮查看两种模式
4. 确认设计后回复 '原型确认' 继续开发
"
```

> ⚠️ **用户必须回复 "原型确认" 或修改意见，否则停止执行。**
> 原型确认后，`Prototype/` 目录作为 Phase 5 UI 开发的**唯一设计参考**，所有 View 按原型 1:1 像素级还原。

### Step 1.5.5 — 原型到代码的映射约定

> **Claude Code 建立从原型到实际代码的映射规则，确保 Phase 5 忠实还原设计:**

```
映射约定:
  ProtoColor.primary        → Color.appPrimary (DesignTokens.swift)
  ProtoColor.bgPrimary      → Color.appCardBackground
  ProtoFont.largeTitle      → Font.appLargeTitle
  ProtoFont.body            → Font.appBody
  ProtoSpacing.md           → .spacingMD
  ProtoCorner.md            → .cornerMD
  ProtoButton               → 实际 Button with .buttonStyle()
  ProtoListRow              → 实际 List row + TaskRowView
  ProtoBadge                → 实际 priority badge component
  ProtoEmptyState           → EmptyStateView
  ProtoPreview              → (仅原型阶段使用，不进入生产代码)
```

> **Claude Code 执行**: `git add -A && git commit -m "Add high-fidelity prototypes & design system"`

---

## Phase 2: 架构搭建 (2:00 - 2:30)

> **[WRITE]** + **[GENERATE]** + **[SHELL]** 生成所有基础架构文件，确保可编译

### Step 2.0 — iOS 平台设计原则 & HIG 指导 (必读)

> **[RESEARCH]** Claude Code 在搭建架构前查阅 HIG_KNOWLEDGE_BASE.md，将 iOS 平台特性贯彻到所有代码。

#### iOS 平台特征 (HIG)
| 维度 | 特征 |
|------|------|
| **屏幕** | 中等尺寸、高分辨率 |
| **人体工学** | 单手或双手握持，视距 1-2 英尺，可横竖屏 |
| **输入** | 多点触控手势、虚拟键盘、语音；陀螺仪和加速度计 |
| **交互** | 数秒快速查看到一小时沉浸使用；频繁多应用切换 |
| **系统特性** | Widgets, Spotlight, Shortcuts, Live Activities |

#### 三大核心设计原则
1. **层级 (Hierarchy)** — 用 Liquid Glass 材质区分控件层和内容层，建立清晰的视觉层级
2. **和谐 (Harmony)** — 与系统界面保持一致，让 App 感觉像 iOS 原生体验
3. **一致性 (Consistency)** — 采用平台惯例，适应横竖屏、暗黑模式、动态字体等系统变化

#### 架构层面的 5 条 HIG 铁律
```
1. 内容延伸到屏幕边缘 → 使用 .ignoresSafeArea() 处理背景
2. 控件放在屏幕中下部 → Tab bars、toolbars、底部按钮组
3. 精简控件数量 → 次要操作放在 .contextMenu 或上滑菜单
4. 最少 44pt 触摸目标 → 所有可交互元素 ≥ 44×44pt
5. 支持横竖屏和分屏 → 使用 GeometryReader、ViewThatFits、自适应布局
```

### Step 2.1 — 验证 Xcode 项目

> **[SHELL]** 确认项目文件正常，首次编译验证

```bash
cd /Users/xurui/Projects/SOP/${PROJECT_NAME}

# [SHELL] 确认 .xcodeproj 存在
ls -la *.xcodeproj && echo "✅ 项目文件存在" || echo "❌ 项目文件不存在"

# [SHELL] 如果 XcodeGen 未生成，手动打开 Xcode 创建
if [ ! -d "${PROJECT_NAME}.xcodeproj" ]; then
    echo "📱 请手动创建 Xcode 项目后继续..."
    # [DIALOG] 用户确认
fi
```

### Step 2.2 — 基础应用入口

> **[GENERATE]** + **[WRITE]** 创建 App 入口和 ContentView

**创建 `${PROJECT_NAME}App.swift`**:

```swift
// ${PROJECT_NAME}/App/${PROJECT_NAME}App.swift
import SwiftUI

@main
struct ${PROJECT_NAME}App: App {
    @StateObject private var appState = AppState()
    @Environment(\.scenePhase) var scenePhase
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
                .onChange(of: scenePhase) { newPhase in
                    switch newPhase {
                    case .active:
                        appState.handleAppBecameActive()
                    case .inactive:
                        appState.handleAppBecameInactive()
                    case .background:
                        appState.handleAppWentToBackground()
                    @unknown default:
                        break
                    }
                }
        }
    }
}

final class AppState: ObservableObject {
    @Published var isFirstLaunch: Bool
    @Published var isAuthenticated: Bool = false
    
    init() {
        isFirstLaunch = !UserDefaults.standard.bool(forKey: "hasLaunched")
        if isFirstLaunch {
            UserDefaults.standard.set(true, forKey: "hasLaunched")
        }
    }
    
    func handleAppBecameActive() {}
    func handleAppBecameInactive() {}
    func handleAppWentToBackground() {}
}
```

**创建 Stub 视图 (占位，Phase 5 替换为真实实现)**:

```swift
// ${PROJECT_NAME}/Views/ContentView.swift
import SwiftUI

struct ContentView: View {
    @EnvironmentObject var appState: AppState

    var body: some View {
        TabView {
            HomeView()
                .tabItem {
                    Label("首页", systemImage: "house.fill")
                }

            DiscoverView()
                .tabItem {
                    Label("发现", systemImage: "magnifyingglass")
                }

            ProfileView()
                .tabItem {
                    Label("我的", systemImage: "person.fill")
                }
        }
    }
}

// MARK: - Stub Views (Phase 5 替换)

struct HomeView: View {
    var body: some View {
        NavigationStack {
            ContentUnavailableView("首页", systemImage: "house.fill",
                description: Text("Phase 5 将实现完整 UI"))
        }
    }
}

struct DiscoverView: View {
    var body: some View {
        NavigationStack {
            ContentUnavailableView("发现", systemImage: "magnifyingglass",
                description: Text("Phase 5 将实现完整 UI"))
        }
    }
}

struct ProfileView: View {
    var body: some View {
        NavigationStack {
            ContentUnavailableView("我的", systemImage: "person.fill",
                description: Text("Phase 5 将实现完整 UI"))
        }
    }
}

#Preview {
    ContentView()
        .environmentObject(AppState())
}
```

> **Note**: Phase 5 的 `HomeView.swift` 将替换此处 Stub 定义的 `HomeView`。Claude Code 在 Phase 5 删除 Stub 并在独立文件中创建完整实现。

### Step 2.3 — 创建基础 Protocol 和 Utilities

**创建 `Services/ServiceProtocol.swift`**:

```swift
// ${PROJECT_NAME}/Services/ServiceProtocol.swift
import Foundation

protocol DataServiceProtocol: AnyObject, Sendable {
    associatedtype Item: Identifiable & Codable
    func fetchAll() async throws -> [Item]
    func fetch(by id: Item.ID) async throws -> Item?
    func create(_ item: Item) async throws -> Item
    func update(_ item: Item) async throws -> Item
    func delete(_ item: Item) async throws
}

protocol StorageServiceProtocol: Sendable {
    func save<T: Codable>(_ value: T, forKey key: String) async throws
    func load<T: Codable>(_ type: T.Type, forKey key: String) async throws -> T?
    func remove(forKey key: String) async
    func clear() async
}

protocol AnalyticsServiceProtocol: Sendable {
    func logEvent(_ name: String, parameters: [String: Any]?)
    func logScreen(_ screenName: String)
    func setUserProperty(_ value: String, forName name: String)
}
```

**创建 `Utilities/ErrorHandler.swift`**:

```swift
// ${PROJECT_NAME}/Utilities/ErrorHandler.swift
import Foundation
import SwiftUI

enum AppError: LocalizedError {
    case networkError(underlying: Error)
    case storageError(reason: String)
    case validationError(message: String)
    case authError(reason: String)
    case unknown
    
    var errorDescription: String? {
        switch self {
        case .networkError(let error):
            return "网络错误: \(error.localizedDescription)"
        case .storageError(let reason):
            return "存储错误: \(reason)"
        case .validationError(let message):
            return message
        case .authError(let reason):
            return "认证失败: \(reason)"
        case .unknown:
            return "发生未知错误"
        }
    }
    
    var recoverySuggestion: String? {
        switch self {
        case .networkError:
            return "请检查网络连接后重试"
        case .storageError:
            return "请重启应用后重试"
        case .validationError:
            return "请检查输入内容"
        case .authError:
            return "请重新登录"
        case .unknown:
            return "请稍后重试"
        }
    }
}

struct ErrorAlertModifier: ViewModifier {
    @Binding var error: AppError?
    
    func body(content: Content) -> some View {
        content
            .alert(
                "出错了",
                isPresented: .init(
                    get: { error != nil },
                    set: { if !$0 { error = nil } }
                ),
                presenting: error
            ) { _ in
                Button("确定") { error = nil }
            } message: { err in
                Text(err.recoverySuggestion ?? "")
            }
    }
}

extension View {
    func errorAlert(_ error: Binding<AppError?>) -> some View {
        modifier(ErrorAlertModifier(error: error))
    }
}
```

**创建 `Utilities/AppConstants.swift`**:

```swift
// ${PROJECT_NAME}/Utilities/AppConstants.swift
import Foundation

enum AppConstants {
    // Bundle
    static var appVersion: String {
        Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "1.0"
    }
    static var buildNumber: String {
        Bundle.main.infoDictionary?["CFBundleVersion"] as? String ?? "1"
    }
    
    // API (如果需要)
    static let apiBaseURL = "https://api.example.com/v1"
    
    // Feature Flags
    static let enableAnalytics = true
    static let enableCrashReporting = true
    
    // UI
    static let cornerRadius: CGFloat = 12
    static let defaultPadding: CGFloat = 16
    static let minimumTapArea: CGFloat = 44
    
    // Storage Keys
    enum StorageKey {
        static let userPreferences = "user_preferences"
        static let cachedData = "cached_data"
        static let lastSyncTimestamp = "last_sync_timestamp"
    }
}
```

### Step 2.4 — 创建 Info.plist

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleDevelopmentRegion</key>
    <string>$(DEVELOPMENT_LANGUAGE)</string>
    <key>CFBundleDisplayName</key>
    <string>${DISPLAY_NAME}</string>
    <key>CFBundleExecutable</key>
    <string>$(EXECUTABLE_NAME)</string>
    <key>CFBundleIdentifier</key>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)</string>
    <key>CFBundleInfoDictionaryVersion</key>
    <string>6.0</string>
    <key>CFBundleName</key>
    <string>$(PRODUCT_NAME)</string>
    <key>CFBundlePackageType</key>
    <string>$(PRODUCT_BUNDLE_PACKAGE_TYPE)</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0.0</string>
    <key>CFBundleVersion</key>
    <string>1</string>
    <key>LSRequiresIPhoneOS</key>
    <true/>
    <key>UIApplicationSceneManifest</key>
    <dict>
        <key>UIApplicationSupportsMultipleScenes</key>
        <false/>
    </dict>
    <key>UILaunchScreen</key>
    <dict>
        <key>UIColorName</key>
        <string>LaunchBackground</string>
        <key>UIImageName</key>
        <string></string>
    </dict>
    <key>UISupportedInterfaceOrientations</key>
    <array>
        <string>UIInterfaceOrientationPortrait</string>
    </array>
    <key>ITSAppUsesNonExemptEncryption</key>
    <false/>
</dict>
</plist>
```

> **Claude Code 执行**: 创建所有上述文件 →

```bash
# [SHELL] 增量编译验证 — Phase 2 结束前必须编译通过
xcodebuild build \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  2>&1 | tail -10

# [DEBUG] 如有编译错误，Claude Code 自动分析并修复
# [GIT] 编译通过后提交
git add -A && git commit -m "Add architecture foundation"
```

---

## Phase 3: 数据层开发 (2:30 - 4:30)

> **[GENERATE]** + **[WRITE]** + **[DEBUG]** 创建 Model + StorageService + NetworkService
> 每次生成后立即 `xcodebuild build` 验证编译

### Step 3.1 — 定义数据模型

> **[GENERATE]** + **[WRITE]** 根据 SPECS.md 创建 SwiftData Model

> Claude Code 根据 SPECS.md 中的数据模型部分创建 Model 文件

**示例模型 — `Models/TaskItem.swift`** (根据实际需求调整):

```swift
// ${PROJECT_NAME}/Models/TaskItem.swift
import Foundation
import SwiftData

@Model
final class TaskItem {
    @Attribute(.unique) var id: UUID
    var title: String
    var taskDescription: String?
    var priority: Priority
    var isCompleted: Bool
    var dueDate: Date?
    var createdAt: Date
    var completedAt: Date?
    var sortOrder: Int
    
    enum Priority: Int, Codable, CaseIterable {
        case low = 0
        case medium = 1
        case high = 2
        case urgent = 3
        
        var displayName: String {
            switch self {
            case .low: return "低"
            case .medium: return "中"
            case .high: return "高"
            case .urgent: return "紧急"
            }
        }
        
        var color: String {
            switch self {
            case .low: return "priorityLow"
            case .medium: return "priorityMedium"
            case .high: return "priorityHigh"
            case .urgent: return "priorityUrgent"
            }
        }
    }
    
    init(
        id: UUID = UUID(),
        title: String,
        description: String? = nil,
        priority: Priority = .medium,
        isCompleted: Bool = false,
        dueDate: Date? = nil,
        sortOrder: Int = 0
    ) {
        self.id = id
        self.title = title
        self.taskDescription = description
        self.priority = priority
        self.isCompleted = isCompleted
        self.dueDate = dueDate
        self.createdAt = Date()
        self.sortOrder = sortOrder
    }
}
```

### Step 3.2 — 创建存储服务

**`Services/StorageService.swift`** (基于 SwiftData):

```swift
// ${PROJECT_NAME}/Services/StorageService.swift
import Foundation
import SwiftData

@MainActor
final class StorageService: StorageServiceProtocol {
    static let shared = StorageService()
    
    private let container: ModelContainer
    private let context: ModelContext
    
    private init() {
        do {
            let schema = Schema([
                TaskItem.self,  // 根据实际 Model 添加
            ])
            let modelConfiguration = ModelConfiguration(
                schema: schema,
                isStoredInMemoryOnly: false,
                allowsSave: true
            )
            container = try ModelContainer(
                for: schema,
                configurations: [modelConfiguration]
            )
            context = container.mainContext
        } catch {
            fatalError("Failed to create ModelContainer: \(error)")
        }
    }
    
    // MARK: - Generic Operations
    
    func fetchAll<T: PersistentModel>() async throws -> [T] {
        let descriptor = FetchDescriptor<T>()
        return try context.fetch(descriptor)
    }
    
    func fetch<T: PersistentModel>(
        predicate: Predicate<T>?
    ) async throws -> [T] {
        var descriptor = FetchDescriptor<T>()
        if let predicate = predicate {
            descriptor.predicate = predicate
        }
        return try context.fetch(descriptor)
    }
    
    func save() async throws {
        try context.save()
    }
    
    func insert<T: PersistentModel>(_ model: T) {
        context.insert(model)
    }
    
    func delete<T: PersistentModel>(_ model: T) {
        context.delete(model)
    }
    
    // MARK: - Key-Value Storage (for preferences)
    
    func save<T: Codable>(_ value: T, forKey key: String) async throws {
        let data = try JSONEncoder().encode(value)
        UserDefaults.standard.set(data, forKey: key)
    }
    
    func load<T: Codable>(_ type: T.Type, forKey key: String) async throws -> T? {
        guard let data = UserDefaults.standard.data(forKey: key) else {
            return nil
        }
        return try JSONDecoder().decode(type, from: data)
    }
    
    func remove(forKey key: String) async {
        UserDefaults.standard.removeObject(forKey: key)
    }
    
    func clear() async {
        if let bundleID = Bundle.main.bundleIdentifier {
            UserDefaults.standard.removePersistentDomain(forName: bundleID)
        }
    }
}
```

### Step 3.3 — 创建网络服务 (如需要)

**`Services/NetworkService.swift`**:

```swift
// ${PROJECT_NAME}/Services/NetworkService.swift
import Foundation

actor NetworkService {
    static let shared = NetworkService()
    
    private let session: URLSession
    private let decoder: JSONDecoder
    private let encoder: JSONEncoder
    
    private init() {
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 30
        config.timeoutIntervalForResource = 60
        config.waitsForConnectivity = true
        
        session = URLSession(configuration: config)
        decoder = JSONDecoder()
        encoder = JSONEncoder()
    }
    
    func request<T: Decodable>(
        _ endpoint: String,
        method: HTTPMethod = .get,
        body: (some Encodable)? = nil,
        headers: [String: String]? = nil
    ) async throws -> T {
        guard let url = URL(string: "\(AppConstants.apiBaseURL)/\(endpoint)") else {
            throw AppError.validationError(message: "Invalid URL")
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = method.rawValue
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        if let body = body {
            request.httpBody = try encoder.encode(body)
        }
        
        headers?.forEach { request.setValue($1, forHTTPHeaderField: $0) }
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw AppError.networkError(underlying: NSError(domain: "", code: -1))
        }
        
        switch httpResponse.statusCode {
        case 200...299:
            return try decoder.decode(T.self, from: data)
        case 401:
            throw AppError.authError(reason: "Unauthorized")
        default:
            throw AppError.networkError(
                underlying: NSError(
                    domain: "",
                    code: httpResponse.statusCode,
                    userInfo: [NSLocalizedDescriptionKey: "HTTP \(httpResponse.statusCode)"]
                )
            )
        }
    }
}

enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case put = "PUT"
    case delete = "DELETE"
    case patch = "PATCH"
}
```

> **Claude Code 执行**: 创建 Model + Service 文件 →

**Step 3.4 — 更新 App 入口注入 SwiftData**

```swift
// 修改 ${PROJECT_NAME}App.swift，在 WindowGroup 上添加 modelContainer
@main
struct ${PROJECT_NAME}App: App {
    @StateObject private var appState = AppState()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
                .onChange(of: scenePhase) { ... }
        }
        .modelContainer(for: [TaskItem.self])  // ← 添加此行，注入 SwiftData 容器
    }
}
```

```bash
# [SHELL] 增量编译验证 — 确保数据层可编译
xcodebuild build \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  2>&1 | tail -5
# [DEBUG] 如有错误，自动分析并修复
```
```bash
# [GIT] 编译通过后提交
git add -A && git commit -m "Add data layer"
```

---

## Phase 4: 业务逻辑层 — ViewModel (4:30 - 6:00)

> **[GENERATE]** + **[WRITE]** + **[REVIEW]** 基于 Model 创建所有 ViewModel

### Step 4.1 — 基础 ViewModel 协议

**`ViewModels/BaseViewModel.swift`**:

```swift
// ${PROJECT_NAME}/ViewModels/BaseViewModel.swift
import SwiftUI

@MainActor
protocol ViewModelProtocol: ObservableObject {
    associatedtype State: Equatable
    associatedtype Action
    
    var state: State { get }
    func send(_ action: Action) async
}

@MainActor
class BaseViewModel: ObservableObject {
    @Published var isLoading: Bool = false
    @Published var error: AppError?
    @Published var toastMessage: String?
    
    func performLoading<T>(_ operation: () async throws -> T) async -> T? {
        isLoading = true
        error = nil
        defer { isLoading = false }
        
        do {
            return try await operation()
        } catch let error as AppError {
            self.error = error
            return nil
        } catch {
            self.error = .unknown
            return nil
        }
    }
    
    func showToast(_ message: String) {
        withAnimation {
            toastMessage = message
        }
        Task {
            try? await Task.sleep(nanoseconds: 3_000_000_000)
            await MainActor.run {
                withAnimation {
                    if toastMessage == message {
                        toastMessage = nil
                    }
                }
            }
        }
    }
}
```

### Step 4.2 — 业务 ViewModel (示例)

**`ViewModels/TaskListViewModel.swift`** (根据 SPECS.md 创建对应 ViewModel):

```swift
// ${PROJECT_NAME}/ViewModels/TaskListViewModel.swift
import SwiftUI
import SwiftData

@MainActor
final class TaskListViewModel: BaseViewModel {
    @Published var tasks: [TaskItem] = []
    @Published var searchText: String = ""
    @Published var selectedFilter: FilterOption = .all
    @Published var sortOrder: SortOption = .dateDesc
    @Published var isAddingTask: Bool = false
    
    private let storageService = StorageService.shared
    
    enum FilterOption: CaseIterable {
        case all, active, completed, highPriority
        
        var displayName: String {
            switch self {
            case .all: return "全部"
            case .active: return "进行中"
            case .completed: return "已完成"
            case .highPriority: return "高优先级"
            }
        }
    }
    
    enum SortOption: CaseIterable {
        case dateDesc, dateAsc, priorityDesc, titleAsc
        
        var displayName: String {
            switch self {
            case .dateDesc: return "最新"
            case .dateAsc: return "最早"
            case .priorityDesc: return "优先级"
            case .titleAsc: return "标题"
            }
        }
    }
    
    var filteredTasks: [TaskItem] {
        var result = tasks
        
        // Filter
        switch selectedFilter {
        case .all: break
        case .active:
            result = result.filter { !$0.isCompleted }
        case .completed:
            result = result.filter { $0.isCompleted }
        case .highPriority:
            result = result.filter { $0.priority == .urgent || $0.priority == .high }
        }
        
        // Search
        if !searchText.isEmpty {
            result = result.filter { $0.title.localizedCaseInsensitiveContains(searchText) }
        }
        
        // Sort
        switch sortOrder {
        case .dateDesc:
            result.sort { $0.createdAt > $1.createdAt }
        case .dateAsc:
            result.sort { $0.createdAt < $1.createdAt }
        case .priorityDesc:
            result.sort { $0.priority.rawValue > $1.priority.rawValue }
        case .titleAsc:
            result.sort { $0.title.localizedCompare($1.title) == .orderedAscending }
        }
        
        return result
    }
    
    func loadTasks() async {
        do {
            tasks = try await storageService.fetchAll()
        } catch {
            self.error = .storageError(reason: error.localizedDescription)
        }
    }
    
    func addTask(title: String, priority: TaskItem.Priority = .medium) async {
        let task = TaskItem(title: title, priority: priority, sortOrder: tasks.count)
        storageService.insert(task)
        await saveAndReload()
        showToast("任务已添加")
    }
    
    func toggleComplete(_ task: TaskItem) async {
        task.isCompleted.toggle()
        if task.isCompleted {
            task.completedAt = Date()
        } else {
            task.completedAt = nil
        }
        await saveAndReload()
    }
    
    func deleteTask(_ task: TaskItem) async {
        storageService.delete(task)
        await saveAndReload()
        showToast("任务已删除")
    }
    
    func updateTask(_ task: TaskItem, title: String, priority: TaskItem.Priority) async {
        task.title = title
        task.priority = priority
        await saveAndReload()
        showToast("任务已更新")
    }
    
    private func saveAndReload() async {
        do {
            try await storageService.save()
            await loadTasks()
        } catch {
            self.error = .storageError(reason: error.localizedDescription)
        }
    }
}
```

> **Claude Code 执行**: 为 SPECS.md 中的每个功能模块创建 ViewModel →

```bash
# [SHELL] 增量编译验证
xcodebuild build \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  -quiet 2>&1 | tail -5
# [DEBUG] 修复编译错误
```
```bash
# [GIT]
git add -A && git commit -m "Add ViewModels"
```

---

## Phase 5: UI 层开发 ← 按高保真原型实现 (6:00 - 7:30)

> **[GENERATE]** + **[WRITE]** + **[REVIEW]** + **[DEBUG]** 1:1 像素级还原 Prototype/ 设计稿
> **⚠️ 铁律: 所有 View 代码严格遵循 DESIGN_SPECS.md 和 Prototype 中的颜色/字号/间距/圆角/阴影定义。**
> **每当不确定 UI 细节时，查阅原型文件而非自行发挥。**

### Step 5.0 — 清理 Stub + 原型映射 & 设计 Token 同步

> **[READ]** + **[EDIT]** Claude Code 先清理 Phase 2 创建的 Stub 视图，再确认原型映射。

```bash
# [SHELL] 从 ContentView.swift 中移除 Stub 定义 (HomeView, DiscoverView, ProfileView)
source /tmp/sop_project.env 2>/dev/null
cd ${PROJECT_DIR}
```

**步骤**:
1. **[READ]** 打开 `${PROJECT_NAME}/Views/ContentView.swift`
2. **[EDIT]** 删除 `// MARK: - Stub Views` 及以下的所有 Stub 代码 (保留 ContentView 和 #Preview)
3. **[EDIT]** 确保文件末尾只有 `ContentView` 和 `#Preview`

> **Claude Code 在开始写 UI 代码前，先确认 Prototype 中的设计值与 DesignTokens.swift 完全同步。**

```bash
# Claude Code 自检: 遍历 Prototype/ 中的所有设计值，确保 DesignTokens 有对应常量
echo "检查设计 Token 同步..."
grep -n "ProtoColor\." ${PROJECT_NAME}/Prototype/DesignSystem.swift | sort -u
grep -n "ProtoFont\."  ${PROJECT_NAME}/Prototype/DesignSystem.swift | sort -u
grep -n "ProtoSpacing\." ${PROJECT_NAME}/Prototype/DesignSystem.swift | sort -u
echo "以上所有设计值应已在 Views/Components/DesignTokens.swift 中有对应声明"
```

**`Prototype/` → 生产代码映射文件** (Claude Code 在实现每个 View 时查阅):

```
// 映射速查表 (Claude Code 编码参考)

颜色映射:
  ProtoColor.primary       → Color.appPrimary       (DesignTokens.swift)
  ProtoColor.bgPrimary     → Color.appCardBackground
  ProtoColor.bgSecondary   → Color.appBackground
  ProtoColor.textPrimary   → Color(.label) / Color.primary
  ProtoColor.textSecondary → Color(.secondaryLabel) / Color.secondary
  ProtoColor.textTertiary  → Color(.tertiaryLabel)
  ProtoColor.separator     → Color(.separator)
  ProtoColor.success       → Color.appSuccess
  ProtoColor.warning       → Color.appWarning
  ProtoColor.error         → Color.appError

字体映射:
  ProtoFont.largeTitle → Font.appLargeTitle
  ProtoFont.title1     → Font.appTitle
  ProtoFont.title2     → Font.appTitle2
  ProtoFont.title3     → Font.appTitle3
  ProtoFont.headline   → Font.appHeadline
  ProtoFont.body       → Font.appBody
  ProtoFont.subhead    → Font.appSubheadline
  ProtoFont.footnote   → Font.appFootnote
  ProtoFont.caption1   → Font.appCaption
  ProtoFont.caption2   → Font.appCaption2

间距映射:
  ProtoSpacing.xs  → .spacingXS (4pt)
  ProtoSpacing.sm  → .spacingSM (8pt)
  ProtoSpacing.md  → .spacingMD (16pt)
  ProtoSpacing.lg  → .spacingLG (24pt)
  ProtoSpacing.xl  → .spacingXL (32pt)
  ProtoSpacing.xxl → .spacingXL * 1.5

圆角映射:
  ProtoCorner.sm  → .cornerSM (8pt)
  ProtoCorner.md  → .cornerMD (12pt)
  ProtoCorner.lg  → .cornerLG (16pt)

组件映射:
  ProtoButton(.primary)   → Button + .buttonStyle(.borderedProminent)
  ProtoButton(.secondary) → Button + .buttonStyle(.bordered)
  ProtoButton(.tertiary)  → Button + .buttonStyle(.plain)
  ProtoListRow            → 对应屏幕的 Row View
  ProtoBadge              → 对应屏幕的 badge component
  ProtoEmptyState         → EmptyStateView (DesignTokens.swift)
```

### Step 5.1 — 设计系统 & 公共组件

> 按 `Prototype/DesignSystem.swift` 定义的生产代码设计系统。以下代码直接映射原型中的所有设计 Token。

**`Views/Components/DesignTokens.swift`**:

```swift
// ${PROJECT_NAME}/Views/Components/DesignTokens.swift
import SwiftUI

// MARK: - Color Palette (HIG 合规)
// 原则: 语义颜色自动适配亮/暗模式；确保 WCAG AA 对比度 (文本 4.5:1, 大文本 3:1)
extension Color {
    // 主色 — 使用 Asset Catalog Color Set 以支持亮/暗/高对比度变体
    static let appPrimary   = Color("AccentColor")           // Asset 中定义 Light/Dark/HighContrast
    static let appSecondary = Color.indigo

    // 语义背景色 — 使用系统颜色自动适配
    static let appBackground     = Color(.systemGroupedBackground)
    static let appCardBackground = Color(.systemBackground)

    // 语义功能色 (HIG: 同一颜色不表达不同含义)
    static let appSuccess = Color.green
    static let appWarning = Color.orange
    static let appError   = Color.red
    static let appInfo    = Color.blue
}

// 在 Assets.xcassets 中创建 AccentColor Color Set:
// - Any Appearance: #007AFF (系统蓝)
// - Dark Appearance: #0A84FF (暗黑模式蓝)
// - 高对比度: 相应变体

// MARK: - Typography (HIG 合规)
// SF Pro 为 iOS 系统字体；优先 Regular/Medium/Semibold/Bold 字重
// 避免 Thin/Light/Ultralight 字重（HIG 建议）
// 最小字号 11pt (iOS)；正文默认 17pt
extension Font {
    // 基于系统 Dynamic Type 样式 (自动响应用户字体大小设置)
    static let appLargeTitle  = Font.system(.largeTitle,  design: .default).weight(.bold)
    static let appTitle       = Font.system(.title,       design: .default).weight(.semibold)
    static let appTitle2      = Font.system(.title2,      design: .default).weight(.semibold)
    static let appTitle3      = Font.system(.title3,      design: .default).weight(.regular)
    static let appHeadline    = Font.system(.headline,    design: .default).weight(.semibold)
    static let appBody        = Font.system(.body,        design: .default)  // 17pt
    static let appCallout     = Font.system(.callout,     design: .default)  // 16pt
    static let appSubheadline = Font.system(.subheadline, design: .default)  // 15pt
    static let appFootnote    = Font.system(.footnote,    design: .default)  // 13pt
    static let appCaption     = Font.system(.caption,     design: .default)  // 12pt
    static let appCaption2    = Font.system(.caption2,    design: .default)  // 11pt — 最小字号
}

// MARK: - Dynamic Type 辅助 (HIG: 支持 ≥200% 放大)
struct ScaledFont: ViewModifier {
    @ScaledMetric var size: CGFloat
    let weight: Font.Weight
    let design: Font.Design

    init(
        size: CGFloat,
        weight: Font.Weight = .regular,
        design: Font.Design = .default
    ) {
        _size = ScaledMetric(wrappedValue: size)
        self.weight = weight
        self.design = design
    }

    func body(content: Content) -> some View {
        content
            .font(.system(size: size, weight: weight, design: design))
    }
}

extension View {
    func scaledFont(
        size: CGFloat,
        weight: Font.Weight = .regular,
        design: Font.Design = .default
    ) -> some View {
        modifier(ScaledFont(size: size, weight: weight, design: design))
    }
}

// MARK: - Materials (Liquid Glass — HIG 新设计语言)
// 原则: Liquid Glass 仅用于控件层/导航层, 不在内容层使用; 节制使用
enum AppMaterial {
    /// 常规 Liquid Glass — 模糊并调整背景亮度 (标签栏、侧边栏、导航栏)
    static let navigationBar   = Material.regularMaterial
    /// 透明 Liquid Glass — 高度半透明 (媒体背景上的控件)
    static let overlayOnMedia  = Material.thinMaterial
    /// 内容层内视觉区分 — 标准材质
    static let contentDivider  = Material.ultraThinMaterial
}

// MARK: - Layout Constants (HIG 合规)
extension CGFloat {
    // HIG 最低触摸目标: 44pt
    static let minimumTapTarget: CGFloat = 44

    // 间距 (基于 8pt 网格系统)
    static let spacingXS: CGFloat = 4
    static let spacingSM: CGFloat = 8
    static let spacingMD: CGFloat = 16
    static let spacingLG: CGFloat = 24
    static let spacingXL: CGFloat = 32

    // 圆角
    static let cornerSM: CGFloat = 8
    static let cornerMD: CGFloat = 12
    static let cornerLG: CGFloat = 16
}

// MARK: - SF Symbols 使用规范 (HIG)
// 不能用于 app 图标、logo 或商标；自动适配 Dark Mode
// 渲染模式: .monochrome(默认) / .hierarchical / .palette / .multicolor
extension Image {
    // 系统图标 — 统一用法
    static func appIcon(_ name: String) -> Image {
        Image(systemName: name)
    }
}

// MARK: - Common Modifiers (HIG 合规)
struct CardModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding(.spacingMD)
            .background(
                RoundedRectangle(cornerRadius: .cornerMD)
                    .fill(Color.appCardBackground)
            )
            .shadow(color: .black.opacity(0.06), radius: 8, x: 0, y: 1)
    }
}

struct PressableModifier: ViewModifier {
    @State private var isPressed = false

    func body(content: Content) -> some View {
        content
            .scaleEffect(isPressed ? 0.97 : 1.0)
            .animation(.interactiveSpring(response: 0.3, dampingFraction: 0.7), value: isPressed)
            .onLongPressGesture(
                minimumDuration: .infinity,
                pressing: { pressing in
                    isPressed = pressing
                },
                perform: {}
            )
    }
}

/// 确保最小触摸区域 ≥ 44pt (HIG 要求)
struct MinimumTapAreaModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .frame(minWidth: .minimumTapTarget, minHeight: .minimumTapTarget)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardModifier())
    }

    func pressable() -> some View {
        modifier(PressableModifier())
    }

    func minimumTapArea() -> some View {
        modifier(MinimumTapAreaModifier())
    }
}

// MARK: - Reusable Components

/// 空状态视图 (HIG: 给重要信息足够空间，不拥挤)
struct EmptyStateView: View {
    let icon: String
    let title: String
    let message: String
    var action: (() -> Void)? = nil
    var actionTitle: String? = nil

    var body: some View {
        VStack(spacing: .spacingLG) {
            Spacer()
            Image(systemName: icon)
                .font(.system(size: 56))
                .foregroundColor(.secondary.opacity(0.6))
                .accessibilityHidden(true)  // 装饰性图标
            Text(title)
                .font(.appHeadline)
                .foregroundColor(.primary)
            Text(message)
                .font(.appSubheadline)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
                .padding(.horizontal, .spacingXL * 2)
            if let action = action, let actionTitle = actionTitle {
                Button(action: action) {
                    Text(actionTitle)
                        .fontWeight(.semibold)
                }
                .buttonStyle(.borderedProminent)
                .padding(.top, .spacingSM)
                .minimumTapArea()
            }
            Spacer()
        }
        .accessibilityElement(children: .combine)
    }
}

/// 加载状态 (HIG: 尽快展示内容，用占位符而非空白；区分确定/不确定进度)
enum LoadingState {
    case idle
    case loading(progress: Double?)  // nil = 不确定进度
    case loaded
    case error(message: String)
}

struct LoadingModifier: ViewModifier {
    let state: LoadingState

    func body(content: Content) -> some View {
        switch state {
        case .idle, .loaded:
            content
        case .loading(let progress):
            ZStack(alignment: .center) {
                content
                    .redacted(reason: .placeholder)  // HIG: 骨架屏优于空白
                    .disabled(true)

                if let progress = progress {
                    // 确定进度 — 用于已知时长的操作
                    VStack(spacing: .spacingMD) {
                        ProgressView(value: progress)
                            .progressViewStyle(.linear)
                            .frame(width: 200)
                        Text("\(Int(progress * 100))%")
                            .font(.appCaption)
                            .foregroundColor(.secondary)
                    }
                } else {
                    // 不确定进度 — 用于未知时长的操作
                    ProgressView()
                        .scaleEffect(1.2)
                }
            }
        case .error(let message):
            VStack(spacing: .spacingMD) {
                Image(systemName: "exclamationmark.triangle.fill")
                    .font(.largeTitle)
                    .foregroundColor(.appError)
                    .accessibilityHidden(true)
                Text(message)
                    .font(.appBody)
                    .foregroundColor(.secondary)
                Button("重试") {
                    // action via callback
                }
                .buttonStyle(.bordered)
                .minimumTapArea()
            }
            .frame(maxWidth: .infinity, maxHeight: .infinity)
            .background(Color.appBackground)
        }
    }
}

extension View {
    func loadingState(_ state: LoadingState) -> some View {
        modifier(LoadingModifier(state: state))
    }
}

// MARK: - HIG 反馈模式 (视觉 + 声音 + 触觉)

/// 触觉反馈工具
enum HapticFeedback {
    static func light() {
        UIImpactFeedbackGenerator(style: .light).impactOccurred()
    }
    static func medium() {
        UIImpactFeedbackGenerator(style: .medium).impactOccurred()
    }
    static func heavy() {
        UIImpactFeedbackGenerator(style: .heavy).impactOccurred()
    }
    static func success() {
        UINotificationFeedbackGenerator().notificationOccurred(.success)
    }
    static func warning() {
        UINotificationFeedbackGenerator().notificationOccurred(.warning)
    }
    static func error() {
        UINotificationFeedbackGenerator().notificationOccurred(.error)
    }
    static func selection() {
        UISelectionFeedbackGenerator().selectionChanged()
    }
}

// MARK: - Toast Modifier (轻量反馈 — 状态信息)
struct ToastModifier: ViewModifier {
    @Binding var message: String?

    func body(content: Content) -> some View {
        ZStack(alignment: .top) {
            content
            if let message = message {
                Text(message)
                    .font(.subheadline)
                    .fontWeight(.medium)
                    .foregroundColor(.white)
                    .padding(.horizontal, .spacingLG)
                    .padding(.vertical, .spacingSM + 4)
                    .background(Material.regularMaterial)  // HIG: Liquid Glass 控件
                    .cornerRadius(.cornerLG)
                    .padding(.top, 100)
                    .transition(.move(edge: .top).combined(with: .opacity))
                    .zIndex(999)
            }
        }
        .animation(.spring(response: 0.4), value: message)
    }
}

extension View {
    func toast(message: Binding<String?>) -> some View {
        modifier(ToastModifier(message: message))
    }
}

// MARK: - HIG 包容性图标选择指南
/// 使用 SF Symbols 时的包容性原则:
/// - figure.stand / figure.walk → 性别中立
/// - person.fill → 通用人物
/// - 避免使用 figure.stand.dress 等有性别暗示的图标(除非 UI 场景明确需要)

// MARK: - Dark Mode 测试辅助
/// 在 Preview 中同时预览亮/暗模式 (HIG: 必须测试两种外观)
struct DarkModePreview<Content: View>: View {
    let content: Content
    var body: some View {
        ForEach(ColorScheme.allCases, id: \.self) { scheme in
            content
                .preferredColorScheme(scheme)
                .previewDisplayName(scheme == .dark ? "Dark" : "Light")
        }
    }
}
```

### Step 5.2 — 首页视图 (示例)

**`Views/HomeView.swift`**:

```swift
// ${PROJECT_NAME}/Views/HomeView.swift
import SwiftUI

struct HomeView: View {
    @StateObject private var viewModel = TaskListViewModel()
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                // Filter Pills
                filterSection
                
                // Sort Picker
                sortSection
                
                // Task List
                taskListSection
            }
            .background(Color.appBackground)
            .navigationTitle("任务")
            .searchable(
                text: $viewModel.searchText,
                placement: .navigationBarDrawer(displayMode: .always),
                prompt: "搜索任务"
            )
            .toolbar {
                ToolbarItem(placement: .primaryAction) {
                    Button {
                        viewModel.isAddingTask = true
                    } label: {
                        Image(systemName: "plus.circle.fill")
                            .font(.title3)
                    }
                }
            }
            .sheet(isPresented: $viewModel.isAddingTask) {
                AddTaskView(viewModel: viewModel)
            }
            .loadingOverlay(viewModel.isLoading)
            .toast(message: $viewModel.toastMessage)
            .errorAlert($viewModel.error)
            .task {
                await viewModel.loadTasks()
            }
            .refreshable {
                await viewModel.loadTasks()
            }
        }
    }
    
    // MARK: - Subviews
    
    private var filterSection: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            HStack(spacing: 8) {
                ForEach(TaskListViewModel.FilterOption.allCases, id: \.self) { option in
                    FilterPill(
                        title: option.displayName,
                        isSelected: viewModel.selectedFilter == option
                    ) {
                        withAnimation(.easeInOut(duration: 0.2)) {
                            viewModel.selectedFilter = option
                        }
                    }
                }
            }
            .padding(.horizontal)
            .padding(.vertical, 10)
        }
        .background(Color.appCardBackground)
    }
    
    private var sortSection: some View {
        HStack {
            Text("排序方式")
                .font(.appCaption)
                .foregroundColor(.secondary)
            Picker("排序", selection: $viewModel.sortOrder) {
                ForEach(TaskListViewModel.SortOption.allCases, id: \.self) { option in
                    Text(option.displayName).tag(option)
                }
            }
            .pickerStyle(.menu)
            Spacer()
            Text("\(viewModel.filteredTasks.count) 项")
                .font(.appCaption)
                .foregroundColor(.secondary)
        }
        .padding(.horizontal)
        .padding(.vertical, 8)
    }
    
    private var taskListSection: some View {
        Group {
            if viewModel.filteredTasks.isEmpty {
                EmptyStateView(
                    icon: "checklist",
                    title: "暂无任务",
                    message: "点击右上角 + 开始添加你的第一个任务",
                    action: { viewModel.isAddingTask = true },
                    actionTitle: "添加任务"
                )
            } else {
                List {
                    ForEach(viewModel.filteredTasks) { task in
                        TaskRowView(task: task, viewModel: viewModel)
                            .pressable()
                    }
                    .onDelete { indexSet in
                        for index in indexSet {
                            let task = viewModel.filteredTasks[index]
                            Task { await viewModel.deleteTask(task) }
                        }
                    }
                }
                .listStyle(.plain)
            }
        }
    }
}
```

**`Views/Components/FilterPill.swift`**:

```swift
// ${PROJECT_NAME}/Views/Components/FilterPill.swift
import SwiftUI

struct FilterPill: View {
    let title: String
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Text(title)
                .font(.subheadline)
                .fontWeight(isSelected ? .semibold : .regular)
                .padding(.horizontal, 16)
                .padding(.vertical, 8)
                .background(
                    Capsule()
                        .fill(isSelected ? Color.appPrimary : Color(.systemGray6))
                )
                .foregroundColor(isSelected ? .white : .primary)
        }
        .buttonStyle(.plain)
    }
}

#Preview {
    HStack {
        FilterPill(title: "全部", isSelected: true, action: {})
        FilterPill(title: "进行中", isSelected: false, action: {})
    }
    .padding()
}
```

**`Views/Components/TaskRowView.swift`**:

```swift
// ${PROJECT_NAME}/Views/Components/TaskRowView.swift
import SwiftUI

struct TaskRowView: View {
    let task: TaskItem
    @ObservedObject var viewModel: TaskListViewModel
    
    var body: some View {
        HStack(spacing: 12) {
            // Completion Toggle
            Button {
                Task { await viewModel.toggleComplete(task) }
            } label: {
                Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
                    .font(.title2)
                    .foregroundColor(task.isCompleted ? .appSuccess : .secondary)
            }
            .buttonStyle(.plain)
            
            // Content
            VStack(alignment: .leading, spacing: 4) {
                Text(task.title)
                    .font(.appBody)
                    .strikethrough(task.isCompleted)
                    .foregroundColor(task.isCompleted ? .secondary : .primary)
                
                if let dueDate = task.dueDate {
                    HStack(spacing: 4) {
                        Image(systemName: "calendar")
                            .font(.caption2)
                        Text(dueDate, style: .date)
                            .font(.caption2)
                    }
                    .foregroundColor(dueDate < Date() && !task.isCompleted ? .appError : .secondary)
                }
            }
            
            Spacer()
            
            // Priority Badge
            priorityBadge
        }
        .padding(.vertical, 4)
    }
    
    private var priorityBadge: some View {
        Text(task.priority.displayName)
            .font(.caption2)
            .fontWeight(.medium)
            .padding(.horizontal, 8)
            .padding(.vertical, 3)
            .background(priorityColor.opacity(0.2))
            .foregroundColor(priorityColor)
            .cornerRadius(6)
    }
    
    private var priorityColor: Color {
        switch task.priority {
        case .low: return .secondary
        case .medium: return .blue
        case .high: return .orange
        case .urgent: return .red
        }
    }
}
```

> **Claude Code 执行**: 为 SPECS.md 中的每个屏幕创建 View，严格按 Prototype/ 原型 1:1 还原 →

```bash
# [SHELL] Phase 5 编译验证 — 确保所有 UI 代码可编译
xcodebuild build \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  -quiet 2>&1 | tail -5
# [DEBUG] 修复编译错误
```
```bash
# [GIT]
git add -A && git commit -m "Add UI layer (prototype-driven)"
```

### Step 5.6 — 模拟器可视化验证 (Day 1 最终确认)

> **[SHELL]** + **[DIALOG]** 在模拟器中启动 App，用户确认视觉效果与原型一致

```bash
source /tmp/sop_project.env 2>/dev/null
source /tmp/sop_simulator.env 2>/dev/null

# [SHELL] 构建并安装到模拟器
xcodebuild build \
  -project ${PROJECT_DIR}/${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination "platform=iOS Simulator,id=$SIMULATOR_ID" \
  -quiet 2>&1 | tail -5

# [SHELL] 启动模拟器中的 App
xcrun simctl boot "$SIMULATOR_ID" 2>/dev/null
xcrun simctl launch "$SIMULATOR_ID" "$BUNDLE_ID"

# [SHELL] 截取模拟器截图供视觉对比
xcrun simctl io "$SIMULATOR_ID" screenshot \
  "${PROJECT_DIR}/screenshot_visual_check.png"
echo "📸 截图已保存: screenshot_visual_check.png"
```

> **[DIALOG]** 用户确认: "请在模拟器中确认 UI 视觉效果与 Prototype/ 原型一致。确认后回复 '视觉确认通过' 继续 Day 2，或提出修改意见。"

### Step 5.3 — HIG 导航模式适配

> **Claude Code 必须确保 App 使用符合 HIG 的导航模式。**

#### 5.3.1 标签栏 (Tab Bar) — iOS 主要导航模式

```swift
// 原则(HIG): 标签栏在屏幕底部；3-5 个标签；每个标签关联独立功能区域
// 使用 SF Symbols 填充样式表示选中
TabView(selection: $selectedTab) {
    HomeView()
        .tabItem {
            Label("首页", systemImage: selectedTab == .home ? "house.fill" : "house")
        }
        .tag(Tab.home)

    DiscoverView()
        .tabItem {
            Label("发现", systemImage: "magnifyingglass")
        }
        .tag(Tab.discover)

    ProfileView()
        .tabItem {
            Label("我的", systemImage: selectedTab == .profile ? "person.fill" : "person")
        }
        .tag(Tab.profile)
}
// HIG: 不要超过 5 个标签；第 5 个用 "更多" (...)
// HIG: 标签栏始终可见，切换时不重置页面状态
```

#### 5.3.2 导航栈 (NavigationStack) — 层级导航

```swift
// 原则(HIG): 清晰的面包屑路径；标题显示当前位置
// 使用 NavigationLink 进行层级深入；返回按钮自动生成
NavigationStack(path: $path) {
    List(items) { item in
        NavigationLink(value: item) {
            ItemRow(item: item)  // HIG: accessory 样式暗示可导航
        }
    }
    .navigationTitle("列表")              // HIG: 标题说明当前层级
    .navigationDestination(for: Item.self) { item in
        DetailView(item: item)
            .navigationTitle(item.title)   // 详情页标题 = 内容标题
    }
}
// HIG: 不要嵌套超过 3-4 层导航
// HIG: 在需要多选/批量操作的场景，使用 EditButton 进入编辑模式
```

#### 5.3.3 搜索 (Search) — HIG 集成搜索

```swift
// 原则(HIG): 搜索栏放在导航栏下方；提供 scope 选项；显示搜索建议
.searchable(
    text: $searchText,
    placement: .navigationBarDrawer(displayMode: .always),  // 始终显示
    prompt: "搜索任务"
) {
    // 搜索建议 (HIG: 帮助用户快速找到内容)
    ForEach(viewModel.recentSearches, id: \.self) { suggestion in
        Text(suggestion)
            .searchCompletion(suggestion)
    }
}
// HIG: 搜索时显示取消按钮；搜索无结果时显示空状态说明
// HIG: 考虑使用 .searchScopes() 提供过滤范围
```

#### 5.3.4 侧边栏 (Sidebar) — iPad/macOS 适配 (可选)

```swift
// 原则(HIG): iPad 横屏使用侧边栏；三栏布局 (Sidebar + List + Detail)
NavigationSplitView {
    // Sidebar — 顶层分类
    List(selection: $selectedCategory) {
        ForEach(categories) { category in
            Label(category.name, systemImage: category.icon)
                .tag(category)
        }
    }
    .navigationTitle("分类")
} content: {
    // Content List — 第二层
    List(selection: $selectedItem) {
        ForEach(items) { item in
            Text(item.title).tag(item)
        }
    }
} detail: {
    // Detail — 第三层
    if let item = selectedItem {
        DetailView(item: item)
    } else {
        ContentUnavailableView("选择一项", systemImage: "sidebar.left")
    }
}
// HIG: iPhone 上 NavigationSplitView 自动退化为 NavigationStack
```

---

### Step 5.4 — HIG 模态 & 反馈模式

> **Claude Code 在实现弹窗、确认框、操作表时必须遵循 HIG 模态规则。**

#### 5.4.1 模态使用原则

```
HIG 模态规则 (Claude Code 必须在所有代码中遵循):
✅ 仅在明确有益时使用模态 (获取关键信息、完成特定任务、沉浸体验)
✅ 模态任务保持简单、简短、精简
✅ 始终提供明显的关闭方式 (关闭按钮 或 下滑手势)
✅ 关闭前如可能丢失数据需确认 (.interactiveDismissDisabled() 配合确认弹窗)
❌ 避免"应用中的应用" (模态视图内不再嵌套复杂视图层级)
❌ 不要在模态中再开一个模态 (除非是系统 ActionSheet/Alert)
```

#### 5.4.2 Sheet 模态 — 标准任务型模态

```swift
// 原则(HIG): 使用 .sheet 处理独立任务；提供标题说明任务目的
.sheet(isPresented: $showSheet) {
    NavigationStack {    // HIG: Sheet 内嵌 NavigationStack 以显示标题和关闭按钮
        Form {
            // 表单内容 (创建/编辑)
        }
        .navigationTitle("新建任务")      // HIG: 必加标题
        .navigationBarTitleDisplayMode(.inline)
        .toolbar {
            ToolbarItem(placement: .cancellationAction) {
                Button("取消") { showSheet = false }
            }
            ToolbarItem(placement: .confirmationAction) {
                Button("保存") { save() }
                    .fontWeight(.semibold)
            }
        }
    }
    .interactiveDismissDisabled(hasUnsavedChanges)  // HIG: 有未保存数据时禁止下滑关闭
}
```

#### 5.4.3 Alert — 关键信息确认

```swift
// 原则(HIG): Alert 仅用于关键且可操作的信息；避免过度使用
.alert("删除此任务?", isPresented: $showDeleteAlert) {
    Button("取消", role: .cancel) { }
    Button("删除", role: .destructive) {
        Task { await viewModel.deleteTask(task) }
    }
} message: {
    Text("此操作不可撤销。")  // HIG: 说明后果
}
// HIG: 删除永远需要二次确认
// HIG: 避免 "确定/取消" 二选一的无意义弹窗 → 改为 Toast
```

#### 5.4.4 ConfirmationDialog — 多选项操作

```swift
// 原则(HIG): 用于从多个选项中选一个；在 iPhone 上显示为底部操作表
.confirmationDialog("选择操作", isPresented: $showActionSheet) {
    Button("编辑", action: edit)
    Button("分享", action: share)
    Button("删除", role: .destructive, action: delete)
    Button("取消", role: .cancel) { }
}
// HIG: iPad 上自动转为 Popover (箭头指向触发按钮)
```

#### 5.4.5 ContextMenu — 次要操作 (HIG 推荐)

```swift
// 原则(HIG): 界面精简控件，次要操作通过最小交互被发现
// 使用 .contextMenu 藏起次要操作，长按或右键触发
.contextMenu {
    Button {
        // 编辑
    } label: {
        Label("编辑", systemImage: "pencil")
    }
    Button(role: .destructive) {
        // 删除
    } label: {
        Label("删除", systemImage: "trash")
    }
}
// HIG: 优先使用 ContextMenu 而非满屏按钮
```

#### 5.4.6 反馈类型映射 (HIG)

```swift
// HIG 反馈分级 — Claude Code 按场景选择合适的反馈类型:
//
// 场景                            → 反馈类型
// ─────────────────────────────────────────────────
// 操作成功 (可忽略)               → Toast (自动消失, 3秒)
// 操作成功 (值得注意)             → Toast + Haptic(.success)
// 信息提示                       → 内嵌状态文本 (.foregroundColor(.secondary))
// 用户错误 (输入不合规)           → 内嵌错误文本 (红色, 紧邻输入框)
// 关键确认 (删除/不可逆)          → Alert (destructive)
// 明确操作 (多选项)               → ConfirmationDialog
// 关键错误 (网络/崩溃)            → Alert + 重试按钮
// 后台操作                       → ProgressView (内嵌或覆盖)
// 通用状态                      → ContentUnavailableView (iOS 17+)
```

---

### Step 5.5 — HIG 引导、启动 & 设置

#### 5.5.1 引导页 (Onboarding) — HIG 原则

```swift
// 原则(HIG):
// - 理想情况: 用户通过体验本身就能理解 App，无需引导
// - 如需引导: 快速、有趣、可选的流程
// - 通过互动教学，不是被动阅读
// - 推迟非必要权限请求 → 在用户即将使用功能时才请求

struct OnboardingView: View {
    @AppStorage("hasCompletedOnboarding") private var hasCompletedOnboarding = false
    @State private var currentPage = 0

    let pages = [
        OnboardingPage(
            icon: "hand.tap.fill",
            title: "快速上手",
            description: "通过简单手势完成任务管理"
        ),
        OnboardingPage(
            icon: "bell.badge.fill",
            title: "及时提醒",
            description: "再也不会忘记重要事项"
        ),
    ]

    var body: some View {
        TabView(selection: $currentPage) {
            ForEach(Array(pages.enumerated()), id: \.offset) { index, page in
                VStack(spacing: 24) {
                    Image(systemName: page.icon)
                        .font(.system(size: 80))
                        .foregroundColor(.appPrimary)
                        .accessibilityHidden(true)
                    Text(page.title)
                        .font(.appTitle)
                    Text(page.description)
                        .font(.appBody)
                        .foregroundColor(.secondary)
                        .multilineTextAlignment(.center)
                        .padding(.horizontal, 32)
                }
                .tag(index)
            }
        }
        .tabViewStyle(.page(indexDisplayMode: .always))
        .overlay(alignment: .bottom) {
            Button(currentPage < pages.count - 1 ? "继续" : "开始使用") {
                if currentPage < pages.count - 1 {
                    currentPage += 1
                } else {
                    hasCompletedOnboarding = true
                }
            }
            .buttonStyle(.borderedProminent)
            .padding(.bottom, 80)
            .minimumTapArea()
        }
    }
}

// HIG: 引导页永远可选 → 提供"跳过"按钮
// HIG: 不要在引导页教用户如何使用系统或设备
// HIG: 权限请求融入引导流程，说明为什么需要和好处
```

#### 5.5.2 启动画面 (Launch Screen) — HIG 关键规则

```
HIG 启动画面铁律 (Claude Code 必须遵循):
❌ 不要在启动画面上放文字 (不会随语言本地化)
❌ 不要做广告或品牌过度展示
❌ 不要延长启动时间
✅ 启动画面应与第一屏几乎相同 (仅不含动态内容)
✅ 使用 Info.plist 中的 UILaunchScreen 配置背景色
✅ 启动后尽快恢复之前状态 (滚动位置、窗口状态)
```

```xml
<!-- Info.plist 中的启动画面配置 (HIG 推荐方式) -->
<key>UILaunchScreen</key>
<dict>
    <key>UIColorName</key>
    <string>LaunchBackground</string>   <!-- 与首页背景色一致 -->
    <key>UIImageName</key>
    <string></string>                    <!-- 留空 = 纯色背景 -->
</dict>
```

#### 5.5.3 应用设置 — HIG 最小化原则

```swift
// 原则(HIG):
// - 提供对最多人数最优体验的默认设置 → 最小化设置数量
// - 将设置放在用户期望的位置 (标签栏或侧边栏的"设置"标签)
// - 尊重系统级设置，不重复系统全局选项
// - 不要用设置请求可通过其他方式获取的信息

struct SettingsView: View {
    @AppStorage("useHapticFeedback") private var useHaptic = true
    @AppStorage("defaultPriority") private var defaultPriority = 1

    var body: some View {
        NavigationStack {
            Form {
                // HIG: 仅放置 App 特有设置；不要重复系统设置 (如暗黑模式)
                Section {
                    Toggle("触觉反馈", systemImage: "hand.rays.fill", isOn: $useHaptic)
                    Picker("默认优先级", systemImage: "flag.fill", selection: $defaultPriority) {
                        Text("低").tag(0)
                        Text("中").tag(1)
                        Text("高").tag(2)
                    }
                } header: {
                    Text("偏好设置")
                }

                Section {
                    NavigationLink {
                        AboutView()
                    } label: {
                        Label("关于", systemImage: "info.circle")
                    }
                    Link(destination: URL(string: AppConstants.privacyURL)!) {
                        Label("隐私政策", systemImage: "hand.raised.fill")
                    }
                }
            }
            .navigationTitle("设置")
        }
    }
}
// HIG: 设置项 < 10 → Form 分组；设置项 > 10 → 考虑用 List + Section
// HIG: 设置值变化立即生效，不需要"保存"按钮
```

#### 5.5.4 内容不可用状态 (iOS 17+)

```swift
// HIG: 使用系统 ContentUnavailableView 统一空状态体验
ContentUnavailableView {
    Label("暂无数据", systemImage: "tray.fill")
} description: {
    Text("添加你的第一个项目开始使用")
} actions: {
    Button("添加项目") { showAddSheet = true }
        .buttonStyle(.borderedProminent)
}

// 搜索结果为空
ContentUnavailableView.search(text: searchText)

// 网络不可用
ContentUnavailableView(
    "无网络连接",
    systemImage: "wifi.slash",
    description: Text("请检查网络设置后重试")
)
```

> **Claude Code 执行**: `git add -A && git commit -m "Add HIG navigation, modality, onboarding & settings patterns"`

---

## Phase 6: 自测 & Code Review (7:30 - 8:00)

> **[SHELL]** + **[DEBUG]** + **[REVIEW]** + **[GIT]** Day 1 收尾验证

### Step 6.1 — 单元测试

> **[GENERATE]** + **[WRITE]** 创建 ViewModel 单元测试

**`${PROJECT_NAME}Tests/TaskListViewModelTests.swift`** (兼容 DI):

```swift
import XCTest
@testable import ${PROJECT_NAME}

@MainActor
final class TaskListViewModelTests: XCTestCase {
    var sut: TaskListViewModel!
    var mockStorage: MockStorageService!

    override func setUp() async throws {
        try await super.setUp()
        // 使用 Mock 注入，不依赖真实 SwiftData
        mockStorage = MockStorageService()
        sut = TaskListViewModel(storageService: mockStorage)
    }

    override func tearDown() async throws {
        sut = nil
        mockStorage = nil
        try await super.tearDown()
    }

    func testInitialState_empty() {
        XCTAssertTrue(sut.tasks.isEmpty)
        XCTAssertEqual(sut.selectedFilter, .all)
        XCTAssertEqual(sut.searchText, "")
    }

    func testFilterOption_allCasesExist() {
        XCTAssertEqual(TaskListViewModel.FilterOption.allCases.count, 4)
    }

    func testSortOption_allCasesExist() {
        XCTAssertEqual(TaskListViewModel.SortOption.allCases.count, 4)
    }
}

// MARK: - Mock Storage (支持单元测试)
final class MockStorageService: StorageServiceProtocol {
    var mockData: [TaskItem] = []
    var shouldThrow = false

    func fetchAll<T: PersistentModel>() async throws -> [T] {
        if shouldThrow { throw AppError.storageError(reason: "Mock error") }
        return mockData as! [T]
    }

    func save<T: Codable>(_ value: T, forKey key: String) async throws {}
    func load<T: Codable>(_ type: T.Type, forKey key: String) async throws -> T? { nil }
    func remove(forKey key: String) async {}
    func clear() async {}
}
```

### Step 6.2 — 执行测试 & 修复

```bash
# 运行单元测试
xcodebuild test \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  -only-testing:${PROJECT_NAME}Tests \
  2>&1 | tee test_results.log

# 如果有失败，Claude Code 自动分析日志并修复
```

### Step 6.3 — 运行 SwiftLint (可选)

```bash
# 安装 SwiftLint (如未安装)
which swiftlint || brew install swiftlint

# 运行检查
cd ${PROJECT_NAME}
swiftlint lint --reporter xcode
```

### Step 6.4 — Day 1 收尾

```bash
# 提交 Day 1 所有代码
git add -A
git commit -m "Day 1 complete: MVP with full MVVM architecture"

# 生成 Day 1 报告
echo "# Day 1 完成报告
- 项目架构: ✅ MVVM + SwiftData
- 数据模型: ✅ 已实现
- 业务逻辑: ✅ ViewModel 完成
- UI 界面: ✅ 所有屏幕完成
- 单元测试: ✅ 基础测试通过
- Git 提交: ✅ $(git rev-parse --short HEAD)
" > DAY1_REPORT.md
```

---

# 🗓️ DAY 2 — 从 MVP 到可发布

---

### Day 2 启动校验 (0:00 - 0:05)

> **[SHELL]** + **[DEBUG]** Claude Code 在 Day 2 开始前先验证 Day 1 产出完好

```bash
cd /Users/xurui/Projects/SOP/${PROJECT_NAME}

# [SHELL] 确认 Git 状态干净
git status --short || echo "⚠️ 有未提交的变更"

# [SHELL] 完整编译 Day 1 代码，确保一夜之间没有环境变化
echo "=== Day 2 启动编译验证 ==="
xcodebuild build \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  2>&1 | tail -10

# [SHELL] 运行单元测试 — 确保 Day 1 测试仍然通过
echo "=== Day 2 启动测试验证 ==="
xcodebuild test \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  -only-testing:${PROJECT_NAME}Tests \
  2>&1 | tail -15

# [DEBUG] 如有失败，先修复再继续 Day 2
echo "✅ Day 2 启动校验通过，开始 Day 2 开发"
```

---

## Phase 7: 集成测试 & Bug 修复 (0:05 - 2:00)

> **[SHELL]** + **[DEBUG]** + **[GENERATE]** Day 2 开始 — 质量验证和边缘情况修复

### Step 7.1 — UI 自动化测试

> **[GENERATE]** + **[WRITE]** 创建 UI 自动化测试

**`${PROJECT_NAME}UITests/AppUITests.swift`**:

```swift
import XCTest

final class AppUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments.append("--uitesting")
        app.launch()
    }
    
    func testMainTabBarExists() throws {
        let tabBar = app.tabBars.firstMatch
        XCTAssertTrue(tabBar.waitForExistence(timeout: 5))
    }
    
    func testAddTaskFlow() throws {
        // Navigate to home tab
        app.tabBars.buttons["首页"].tap()
        
        // Tap add button
        let addButton = app.navigationBars.buttons["plus.circle.fill"]
        if addButton.waitForExistence(timeout: 3) {
            addButton.tap()
            
            // Verify sheet appears
            let sheet = app.sheets.firstMatch
            XCTAssertTrue(sheet.waitForExistence(timeout: 3))
        }
    }
    
    func testSearchFunctionality() throws {
        app.tabBars.buttons["首页"].tap()
        
        let searchField = app.searchFields.firstMatch
        if searchField.waitForExistence(timeout: 3) {
            searchField.tap()
            searchField.typeText("测试搜索")
        }
    }
}
```

### Step 7.2 — 运行 UI 测试

```bash
# 运行 UI 测试
xcodebuild test \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  -only-testing:${PROJECT_NAME}UITests \
  2>&1 | tee ui_test_results.log

# Claude Code 分析失败测试并自动修复
```

### Step 7.3 — 边界情况处理 (HIG 增强)

Claude Code 检查并添加以下处理:

```swift
// 网络不可用时的离线模式
// 空状态处理 (列表为空、搜索结果为空)
// 输入验证 (空标题、超长文本)
// 错误状态 (网络错误、权限被拒)
// 加载状态 (骨架屏或 loading indicator)
```

#### HIG 交互边界情况

```
Claude Code 必须覆盖以下 HIG 交互场景:
□ 手势冲突: 自定义手势不与系统手势冲突 (如边缘滑动返回)
□ 手势替代: 每个自定义手势都有替代操作方式 (按钮/菜单)
□ 键盘适配: 输入框不被键盘遮挡 (iOS: keyboard avoidance)
□ 横竖屏: 横竖屏切换时 UI 不丢失状态、不挤压内容
□ 分屏 (iPad): 在 1/3、1/2、2/3、全屏宽度下 UI 都可用
□ 中断恢复: 来电/切换App后返回，状态不丢失
□ 通知处理: 点击通知进入对应页面 (Deep Link)
□ 内存警告: 收到 didReceiveMemoryWarning 时释放可重建的资源
□ VoiceOver 导航: 模态出现时焦点自动移到模态内
□ 动态字体极限: 辅助功能最大字号下 UI 不崩溃
```

#### HIG 权限请求时机

```swift
// ❌ 错误: 启动时一次性请求所有权限
// ✅ 正确: 在用户即将使用功能时才请求 (HIG)

// 反例 — 不要这样做:
func requestAllPermissionsOnLaunch() {
    requestCamera()      // 用户可能不需要相机
    requestLocation()    // 用户可能不需要定位
    requestNotification() // 应该在功能上下文中请求
}

// 正例 — HIG 推荐:
struct CameraFeatureView: View {
    @State private var showCamera = false
    
    var body: some View {
        Button("拍照") {
            // 用户点击才请求 — 有明确上下文
            requestCameraPermission { granted in
                if granted { showCamera = true }
            }
        }
    }
}

// HIG: 权限请求时说明为什么需要
// "允许访问相机以扫描文档" ✅ (有理由)
// "应用想要访问相机"     ❌ (无理由)
```

### Step 7.4 — Accessibility 基础扫描

> **[SHELL]** + **[REVIEW]** 自动化扫描 + 编译检查。完整审计在 Phase 8。

```bash
source /tmp/sop_project.env 2>/dev/null
source /tmp/sop_simulator.env 2>/dev/null

# [SHELL] 编译检查 (含长表达式警告)
xcodebuild build \
  -project ${PROJECT_DIR}/${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination "platform=iOS Simulator,id=$SIMULATOR_ID" \
  OTHER_SWIFT_FLAGS="-Xfrontend -warn-long-expression-type-checking=200"

# [SHELL] Claude Code 自动扫描代码中的无障碍问题:
echo "=== 扫描缺少 accessibilityLabel 的 Image ==="
grep -rn 'Image(systemName:' ${PROJECT_DIR}/${PROJECT_NAME}/Views/ --include="*.swift" | \
  grep -v 'accessibilityLabel\|accessibilityHidden\|Preview\|#Preview' | head -10

echo "=== 扫描触摸目标不足 44pt 的控件 ==="
grep -rn '\.frame\|\.padding' ${PROJECT_DIR}/${PROJECT_NAME}/Views/ --include="*.swift" | head -10

echo "=== 手动验证 ==="
echo "请在 Xcode 中: Xcode → Open Developer Tool → Accessibility Inspector → 选择模拟器 → Audit (⌘A)"
echo "✅ Phase 8 将进行完整 WCAG AA 合规审计"
```

---

## Phase 8: 性能优化 & 无障碍 (2:00 - 3:30)

> **[REVIEW]** + **[GENERATE]** + **[VALIDATE]** 逐项审计性能和无障碍合规

### Step 8.1 — 性能检查清单

> **[REVIEW]** 扫描所有 View，逐项检查性能问题

Claude Code 逐项检查并优化:

```swift
// ✅ 1. 图片缓存 (使用 NukeUI AsyncImage)
// ✅ 2. LazyVStack / LazyVGrid 替代 VStack 大列表
// ✅ 3. @StateObject vs @ObservedObject 正确使用
// ✅ 4. 避免在 body 中进行复杂计算
// ✅ 5. 适当使用 .task {} 而非 .onAppear
// ✅ 6. SwiftData 查询使用 BackgroundContext 写入
// ✅ 7. debounce 搜索输入
// ✅ 8. Image 使用 .resizingMode(.stretch) 减小内存
```

**搜索防抖实现**:

```swift
// ${PROJECT_NAME}/Utilities/Debounce.swift
import Foundation

final class Debouncer: @unchecked Sendable {
    private let delay: TimeInterval
    private var workItem: DispatchWorkItem?
    
    init(delay: TimeInterval = 0.3) {
        self.delay = delay
    }
    
    func debounce(action: @escaping @Sendable () async -> Void) {
        workItem?.cancel()
        let workItem = DispatchWorkItem {
            Task { await action() }
        }
        self.workItem = workItem
        DispatchQueue.main.asyncAfter(deadline: .now() + delay, execute: workItem)
    }
}
```

### Step 8.2 — 无障碍审计 (HIG 完整合规)

> **Claude Code 必须逐项检查以下 HIG 无障碍要求。这是 App Store 审核的硬性标准。**

#### 8.2.1 HIG 无障碍三大特质

```
✅ 直觉性 (Intuitive) — 使用熟悉且一致的交互使任务简单直接
✅ 可感知性 (Perceivable) — 不依赖单一方式传达信息 (视觉+听觉+语音+触觉)
✅ 适应性 (Adaptable) — 通过系统可及性功能或个性化设置适应不同使用方式
```

#### 8.2.2 Dynamic Type — 动态字体 (HIG 要求 ≥200% 放大)

```swift
// HIG 字体大小标准:
// iOS 默认正文字号: 17pt, 最小字号: 11pt
// 必须支持 Dynamic Type 至少 200% 放大

// ✅ 正确: 使用系统字体样式 (自动响应 Dynamic Type)
Text("标题")
    .font(.headline)  // 使用语义样式，自动适配大小

// ✅ 正确: 自定义字号使用 @ScaledMetric
@ScaledMetric var customSize: CGFloat = 16
Text("自定义")
    .font(.system(size: customSize))

// ❌ 错误: 硬编码字号
Text("错误")
    .font(.system(size: 16))  // 不会响应用户的字体设置

// ✅ 正确: layoutPriority 确保较大文本时有正确布局
VStack(alignment: .leading) {
    Text("标题").font(.headline)
    Text("副标题").font(.subheadline)
        .lineLimit(3)           // 允许文本换行而非截断
        .minimumScaleFactor(0.8) // 必要时缩小
}

// ✅ iOS 尺寸类别适配
@Environment(\.horizontalSizeClass) var horizontalSizeClass
@Environment(\.dynamicTypeSize) var dynamicTypeSize
```

#### 8.2.3 色彩对比度 — WCAG AA 合规 (HIG)

```swift
// HIG 最低对比度标准 (WCAG Level AA):
// ┌──────────────────────────┬─────────┐
// │ 文本 ≤ 17pt               │ 4.5 : 1 │
// │ 文本 ≥ 18pt               │ 3.0 : 1 │
// │ 粗体文本 (≥14pt/18pt)      │ 3.0 : 1 │
// │ 非文本 UI 组件 & 图形      │ 3.0 : 1 │
// └──────────────────────────┴─────────┘
// Dark Mode & Increase Contrast 下也需达标

// Claude Code 自查命令 (使用 Xcode Accessibility Inspector):
// Xcode → Open Developer Tool → Accessibility Inspector → Color Contrast Calculator

// ✅ 代码中确保对比度:
struct AccessibleLabel: View {
    var body: some View {
        Text("标签")
            .foregroundColor(.primary)       // 系统颜色自动适配
            .background(Color(.systemBackground))
        // 系统 primary 在亮色模式 ≈ #000000, 背景 #FFFFFF → 对比度 21:1 ✅
        // 系统 primary 在暗色模式 ≈ #FFFFFF, 背景 #000000 → 对比度 21:1 ✅
    }
}

// ❌ 避免的对比度陷阱:
// - 浅灰文字 on 浅灰背景 (如 .secondary on .systemGray5)
// - 蓝色文字 on 暗色背景 (暗黑模式下不够亮)
// - 彩色背景上的白色文字 (只在对比度足够时使用)
```

#### 8.2.4 VoiceOver 完整支持

```swift
// 所有交互元素必须有:
// 1. .accessibilityLabel() — 描述元素是什么
// 2. .accessibilityHint()  — 描述操作后发生什么
// 3. .accessibilityAddTraits() — 元素类型 (.isButton, .isHeader, .isSelected)
// 4. .accessibilityElement(children:) — 合并/分离复杂视图

// ✅ 正确的 VoiceOver 接入
Button {
    Task { await viewModel.toggleComplete(task) }
} label: {
    Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
        .font(.title2)
}
.accessibilityLabel(task.isCompleted ? "已完成: \(task.title)" : "未完成: \(task.title)")
.accessibilityHint("双击切换完成状态")
.accessibilityAddTraits(task.isCompleted ? [.isButton, .isSelected] : .isButton)
.minimumTapArea()  // HIG: 触摸目标≥44pt

// ✅ 装饰性元素隐藏
Image(systemName: "sparkles")
    .accessibilityHidden(true)  // 纯装饰性，不传达信息

// ✅ 复杂视图组合
VStack {
    Text(task.title)
    Text(task.dueDate, style: .date)
        .font(.caption)
        .foregroundColor(.secondary)
}
.accessibilityElement(children: .combine)  // 合并为单一可及性元素
.accessibilityLabel("\(task.title), 截止日期 \(task.dueDate.formatted())")

// ✅ 排序/过滤控件
HStack {
    ForEach(FilterOption.allCases, id: \.self) { option in
        Button { selectFilter(option) } label: {
            Text(option.displayName)
        }
        .accessibilityLabel(option.displayName)
        .accessibilityAddTraits(viewModel.selectedFilter == option ? .isSelected : [])
    }
}
.accessibilityLabel("筛选选项")  // 整体描述
```

#### 8.2.5 动效 & 动画 — Reduce Motion 支持

```swift
// HIG: 为用户提供关闭动效的选项
// 检测系统 Reduce Motion 设置
@Environment(\.accessibilityReduceMotion) var reduceMotion

struct AnimatedTransition: View {
    @Environment(\.accessibilityReduceMotion) var reduceMotion

    var body: some View {
        if reduceMotion {
            content  // 无动画版本
        } else {
            content
                .animation(.spring(), value: state)
        }
    }
}

// 或在 ViewModifier 中统一处理
extension View {
    func accessibleAnimation<T: Equatable>(_ animation: Animation?, value: T) -> some View {
        modifier(AccessibleAnimationModifier(animation: animation, value: value))
    }
}

struct AccessibleAnimationModifier<T: Equatable>: ViewModifier {
    let animation: Animation?
    let value: T
    @Environment(\.accessibilityReduceMotion) var reduceMotion

    func body(content: Content) -> some View {
        if reduceMotion {
            content
        } else {
            content.animation(animation, value: value)
        }
    }
}
```

#### 8.2.6 透明度 — Reduce Transparency 支持

```swift
// HIG: 减少透明度开启时，Liquid Glass 材质可能失效
// 确保在 reduceTransparency 下内容仍然可读

@Environment(\.accessibilityReduceTransparency) var reduceTransparency

.background(
    reduceTransparency
        ? Color.appBackground  // 不透明回退
        : Material.regularMaterial  // Liquid Glass
)
```

#### 8.2.7 多通道反馈 (HIG 要求)

```swift
// HIG: 不为单一感官通道设计反馈 — 同时提供视觉 + 声音 + 触觉
func confirmAction() {
    // 视觉反馈
    toastMessage = "操作成功"

    // 触觉反馈
    HapticFeedback.success()

    // 音频反馈 (通过 VoiceOver 或系统)
    UIAccessibility.post(notification: .announcement, argument: "操作成功")
}

// 错误反馈三通道
func handleError(_ error: Error) {
    // 1. 视觉: 红色提示 + 图标
    showErrorBanner(error.localizedDescription)

    // 2. 触觉: 错误震动
    HapticFeedback.error()

    // 3. 音频: VoiceOver 播报
    UIAccessibility.post(notification: .announcement, argument: "发生错误: \(error.localizedDescription)")
}
```

#### 8.2.8 包容性设计检查 (HIG)

```swift
// HIG 包容性考虑维度: 年龄、性别认同、种族、性取向、身体/认知属性、残障、语言文化、宗教等

// ✅ SF Symbols 性别中立
Image(systemName: "person.fill")          // 通用，不指定性别
Image(systemName: "figure.stand")         // 性别中立
// ❌ 避免
Image(systemName: "figure.stand.dress")   // 有性别暗示，除非UI场景需要

// ✅ Avatar 占位
Image(systemName: "person.crop.circle.fill")  // 通用头像占位

// ✅ 包容性语言
Text("你好，欢迎回来")          // 直接称呼用户，不假设性别
// ❌ 避免
Text("兄弟们，开始吧")           // 假设用户群体

// ✅ 肤色中性 Emoji
// 使用默认肤色 (黄色) 的 Emoji，除非用户主动选择肤色
Text("👍")  // 默认肤色
```

#### 8.2.9 Accessibility 自动化审计脚本

```bash
# Claude Code 使用 Accessibility Inspector 审计:
# 1. 在 Xcode 中打开 Accessibility Inspector
# 2. 选择模拟器或设备作为 Target
# 3. 运行 Audit (⌘ + A)

# 命令行检查 (使用 xcodebuild 编译标记)
xcodebuild build \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  OTHER_SWIFT_FLAGS="-Xfrontend -warn-long-expression-type-checking=200"

# Claude Code 手动检查清单 (逐页过):
# □ 每页的 VoiceOver 焦点顺序是否合理? (从上到下, 从左到右)
# □ 所有图片是否设置了 accessibilityLabel 或标记为 hidden?
# □ 所有按钮是否能在 Dynamic Type 最大时不被截断?
# □ 在 Dark Mode + Increase Contrast 下所有内容是否可读?
# □ Reduce Motion ON 时动画是否已移除?
# □ Reduce Transparency ON 时 UI 是否仍然可用?
# □ 执行具体操作后是否有触觉/音频反馈?
```

> **Claude Code 执行**: 逐项完成后 → `git commit -m "Add comprehensive HIG accessibility compliance"`

### Step 8.3 — Swift 代码质量 & 安全审计

> **[SHELL]** + **[VALIDATE]** Claude Code 运行自动化检查 + 手动审计清单

```bash
# [SHELL] 1. SwiftLint 静态分析
if which swiftlint >/dev/null; then
  swiftlint lint --reporter xcode 2>&1 | tail -20
else
  echo "⚠️ SwiftLint 未安装: brew install swiftlint"
fi

# [SHELL] 2. 编译警告检查
xcodebuild build \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  2>&1 | grep -i "warning" | head -10

# [SHELL] 3. 硬编码敏感信息扫描
grep -rn "apiKey\|api_key\|secret\|password\|token" ${PROJECT_NAME}/ \
  --include="*.swift" | grep -v "//\|example\|test\|Keychain" | head -10

# [SHELL] 4. 大文件检查 (>400行)
find ${PROJECT_NAME} -name "*.swift" -exec wc -l {} \; | \
  awk '$1 > 400 {print $1, $2}' | sort -rn | head -10

# [SHELL] 5. 强制解包检查 (可能导致崩溃)
grep -rn '!' ${PROJECT_NAME}/ --include="*.swift" | \
  grep -v "//\|#warning\|test\|Preview\|as!\|try!" | head -10
```

**Swift 代码规范检查清单**:

```
✅ 类型安全
  □ 无强制解包 (!) (使用 guard let / if let)
  □ 无隐式可选类型滥用
  □ 使用 enum 管理状态 (非魔法字符串)
  □ 协议声明使用 any Protocol (Swift 5.7+)

✅ SwiftUI 规范
  □ View body 简洁 (≤ 30 行)
  □ 复杂 View 拆分为子组件
  □ @StateObject 用于创建, @ObservedObject 用于接收
  □ 无过度使用 @EnvironmentObject (限制 2-3 个)

✅ 安全
  □ 敏感数据使用 Keychain (非 UserDefaults)
  □ API Key 不在代码中硬编码 (使用 xcconfig)
  □ 网络请求使用 HTTPS (App Transport Security)
  □ 输入验证完整 (防注入)

✅ 性能
  □ List/ScrollView 使用 LazyVStack/LazyVGrid
  □ 图片异步加载 (AsyncImage)
  □ 无主线程阻塞操作 (await async)
  □ 使用 debounce 限制搜索/输入频率

✅ 资源管理
  □ 字符串使用 NSLocalizedString (非硬编码)
  □ 颜色使用 Asset Catalog (支持深色模式)
  □ 图片使用 Asset Catalog (@2x/@3x)
```

**代码质量门禁**:
```
以下条件全部满足方可进入 Phase 9:
□ xcodebuild build 无 ERROR + 无 WARNING
□ SwiftLint 无 ERROR (WARNING < 10)
□ 无硬编码密钥/Token
□ 无 >400 行单个 .swift 文件
□ 无强制解包 (!) 滥用
```

### Step 8.4 — 隐私清单 (Privacy Manifest)

Apple 从 2024 年起要求 Privacy Manifest。Claude Code 创建:

**`${PROJECT_NAME}/Resources/PrivacyInfo.xcprivacy`**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSPrivacyTracking</key>
    <false/>
    <key>NSPrivacyTrackingDomains</key>
    <array/>
    <key>NSPrivacyCollectedDataTypes</key>
    <dict>
        <key>NSPrivacyCollectedDataTypeProductInteraction</key>
        <array>
            <dict>
                <key>NSPrivacyCollectedDataTypePurpose</key>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
                <key>NSPrivacyCollectedDataTypeLinkedToUser</key>
                <false/>
                <key>NSPrivacyCollectedDataTypeTracking</key>
                <false/>
            </dict>
        </array>
    </dict>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string>
            </array>
        </dict>
    </array>
</dict>
</plist>
```

> **Claude Code 执行**: `git commit -m "Add performance optimization, accessibility & privacy manifest"`

---

## Phase X: 依赖注入容器 & 架构微调 (可选, 嵌入各 Phase)

> **[WRITE]** + **[REVIEW]** 如果项目 Service ≥ 3 个或 ViewModel ≥ 5 个，引入轻量 DI 容器
> 不单独占用时间，在各 Phase 中渐进引入

### Step X.1 — 轻量 DI 容器 (无需 Swinject)

**创建 `${PROJECT_NAME}/Utilities/DIContainer.swift`**:

```swift
// ${PROJECT_NAME}/Utilities/DIContainer.swift
// 轻量 DI 容器 — 避免全局单例，便于测试时替换 Mock

@MainActor
final class DIContainer: ObservableObject {
    // Services
    let storageService: StorageService
    let networkService: NetworkService
    let iapService: IAPService

    // ViewModels (factory pattern — 每次创建新实例)
    func makeTaskListViewModel() -> TaskListViewModel {
        TaskListViewModel(storageService: storageService)
    }

    func makeSettingsViewModel() -> SettingsViewModel {
        SettingsViewModel(storageService: storageService)
    }

    init(
        storageService: StorageService = .shared,
        networkService: NetworkService = .shared,
        iapService: IAPService = .shared
    ) {
        self.storageService = storageService
        self.networkService = networkService
        self.iapService = iapService
    }
}

// MARK: - Preview Helper

extension DIContainer {
    /// 用于 SwiftUI Preview 的 Mock 容器
    static var preview: DIContainer {
        DIContainer(
            storageService: StorageService.mock(),
            networkService: NetworkService.mock(),
            iapService: IAPService.mock()
        )
    }
}
```

**更新 App 入口注入 DI**:

```swift
// ${PROJECT_NAME}App.swift 更新
@main
struct ${PROJECT_NAME}App: App {
    @StateObject private var container = DIContainer()
    @StateObject private var appState = AppState()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(container)
                .environmentObject(appState)
        }
    }
}
```

**更新 ViewModel 使用注入**:

```swift
// 改前: 直接使用单例
final class TaskListViewModel: BaseViewModel {
    private let storageService = StorageService.shared  // ❌ 硬依赖
}

// 改后: 依赖注入
final class TaskListViewModel: BaseViewModel {
    private let storageService: StorageService

    init(storageService: StorageService) {  // ✅ 可注入 Mock
        self.storageService = storageService
    }
}
```

> **Claude Code 执行**: `git commit -m "Add DI container for testability"`

---

## Phase 9: App Store 素材准备 (3:30 - 4:30)

> **[SHELL]** + **[WRITE]** + **[REVIEW]** + **[DIALOG]** 准备 App Store 所需的全部素材和元数据

### Step 9.0 — 在 App Store Connect 创建 App 记录

> **[DIALOG]** 必须先创建 App Record，Phase 11 才能上传构建版本。Claude Code 无法操作 ASC，输出操作指南让用户执行。

```
═══════════════════════════════════════════════
📱 请先在 App Store Connect 创建 App 记录
═══════════════════════════════════════════════

1. 打开 https://appstoreconnect.apple.com
2. 点击 "我的 App" → 左上角 ⊕ → "新建 App"
3. 填写以下信息:
   □ 平台: iOS
   □ 名称: ${DISPLAY_NAME}
   □ 主要语言: Simplified Chinese (zh-Hans)
   □ Bundle ID: ${BUNDLE_ID}
     (如果还没注册, 先去 "证书、标识符和配置文件" 注册)
   □ SKU: ${BUNDLE_ID} (唯一标识, 对外不可见)
   □ 用户访问权限: 完全访问
4. 点击 "创建"
5. 记录 Apple ID (数字 ID), 后续 Fastlane 需要

创建完成后回复 "App 记录已创建" 继续
═══════════════════════════════════════════════
```

> ⚠️ **Bundle ID 注册**: 如果 `${BUNDLE_ID}` 尚未在 Apple Developer Portal 注册，需要先去注册:
> 1. https://developer.apple.com → Certificates, Identifiers & Profiles
> 2. Identifiers → ⊕ → App IDs → 选择 "App" → 填写 Bundle ID
> 3. 确认 Capabilities (默认即可，如需要 Push Notifications 等在此添加)

### Step 9.1 — 应用图标 (HIG 分层设计)

> **HIG: App Icon 是品牌和用户体验的关键，出现在主屏幕、搜索结果、通知、设置、分享菜单等各处。**

#### HIG App Icon 设计要求

```
iOS App Icon 规范:
├─ 分层设计: 背景层 + 1~多个前景层
├─ Liquid Glass 属性: 高光、霜感、半透明
├─ 前景层边缘清晰 (避免柔化/羽化)
├─ 不同前景层用不同透明度增加深度感
├─ 背景使用从上到下、由浅到深的微妙渐变
├─ 优先矢量图形 (SVG/PDF)，避免光栅图
├─ 尺寸: 1024×1024 px (App Store Connect)
└─ 格式: PNG (无透明、无圆角 — 系统自动添加圆角)
```

Claude Code 指导:

```bash
# 1. 准备 1024x1024 PNG 图标文件 (无透明、无圆角)
# 2. 使用 Xcode 自带的 Icon Composer 导入和调整图标层
# 3. 或者使用 sips 生成多尺寸图标 (如果有源图)
```

**使用 sips 生成图标 (如果有源图)**:

```bash
SOURCE_ICON="${PROJECT_NAME}/Resources/AppIcon.png"
ASSETS_DIR="${PROJECT_NAME}/Resources/Assets.xcassets/AppIcon.appiconset"

mkdir -p ${ASSETS_DIR}

# 生成各尺寸 (iOS @2x 和 @3x)
for size in 20 29 40 60 76 83.5; do
    pixels2x=$(echo "$size * 2" | bc | cut -d. -f1)
    pixels3x=$(echo "$size * 3" | bc | cut -d. -f1)
    sips -z ${pixels2x} ${pixels2x} "$SOURCE_ICON" --out "${ASSETS_DIR}/icon_${size}@2x.png"
    sips -z ${pixels3x} ${pixels3x} "$SOURCE_ICON" --out "${ASSETS_DIR}/icon_${size}@3x.png"
done

# 1024x1024 for App Store (HIG: 必须提供)
cp "$SOURCE_ICON" "${ASSETS_DIR}/icon_1024.png"

# 创建 Contents.json (Xcode 会自动生成，但也可以手动)
cat > "${ASSETS_DIR}/Contents.json" << 'JSONEOF'
{
  "images" : [
    {"size" : "20x20", "idiom" : "iphone", "filename" : "icon_20@2x.png", "scale" : "2x"},
    {"size" : "20x20", "idiom" : "iphone", "filename" : "icon_20@3x.png", "scale" : "3x"},
    {"size" : "29x29", "idiom" : "iphone", "filename" : "icon_29@2x.png", "scale" : "2x"},
    {"size" : "29x29", "idiom" : "iphone", "filename" : "icon_29@3x.png", "scale" : "3x"},
    {"size" : "40x40", "idiom" : "iphone", "filename" : "icon_40@2x.png", "scale" : "2x"},
    {"size" : "40x40", "idiom" : "iphone", "filename" : "icon_40@3x.png", "scale" : "3x"},
    {"size" : "60x60", "idiom" : "iphone", "filename" : "icon_60@2x.png", "scale" : "2x"},
    {"size" : "60x60", "idiom" : "iphone", "filename" : "icon_60@3x.png", "scale" : "3x"},
    {"size" : "1024x1024", "idiom" : "ios-marketing", "filename" : "icon_1024.png", "scale" : "1x"}
  ],
  "info" : {"author" : "xcode", "version" : 1}
}
JSONEOF
```

> ⚠️ **用户必须提供原始图标文件**，否则 Claude Code 使用 SF Symbols 生成占位图标 (但 SF Symbols **不得**用作 App Icon — HIG 禁止)。

### Step 9.2 — 启动屏幕

SwiftUI 自动使用 LaunchScreen (在 Info.plist 中配置了 `UILaunchScreen`)。

如需自定义启动画面:

```swift
// 在 Info.plist 配置的 UILaunchScreen dictionary 中添加背景色等
```

### Step 9.3 — App Store 截图

```bash
mkdir -p fastlane/screenshots

# 使用 Fastlane Snapshot 自动截图 (推荐)
# 或手动截取以下尺寸:
# iPhone 6.7" Display: 1290 x 2796 px
# iPhone 6.5" Display: 1284 x 2778 px  (可选, 6.7" 会缩放)
# iPhone 5.5" Display: 1242 x 2208 px  (可选)
```

**创建 `fastlane/Snapfile`**:

```ruby
# fastlane/Snapfile
devices([
  "iPhone 16 Pro Max",   # 6.9"
  "iPhone 16 Pro",       # 6.3"
])

languages([
  "zh-Hans",  # 简体中文
  "en-US",    # 英文
])

scheme("${PROJECT_NAME}")

output_directory("./fastlane/screenshots")

clear_previous_screenshots(true)
```

### Step 9.4 — App Store 元数据 (HIG 写作指导)

#### HIG 写作核心原则

```
✅ 清晰: 使用易理解的词语，能少则少
✅ 面向所有人: 简单平实的语言，考虑可及性和本地化
✅ 以行动为导向: 按钮和链接使用动词
✅ 直接称呼用户: 用"你/你的"
✅ 保持语言模式一致性: 整句结构、标点风格统一
✅ 包容性语言: 避免无法被所有人理解或有冒犯性的图像和隐喻
```

**创建 `fastlane/metadata/zh-Hans/description.txt`**:

```text
[简洁有力的App描述，100-170字]

核心功能:
• 功能亮点1
• 功能亮点2
• 功能亮点3

适用场景:
• 场景1
• 场景2

[结尾号召性语句]
```

**创建 `fastlane/metadata/zh-Hans/keywords.txt`**:

```text
关键词1,关键词2,关键词3,关键词4,关键词5
```

**创建 `fastlane/metadata/zh-Hans/promotional_text.txt`**:

```text
[促销文本，最多170字，可随时更新无需审核]
```

**创建 `fastlane/metadata/zh-Hans/support_url.txt`**:
```text
https://example.com/support
```

**创建 `fastlane/metadata/zh-Hans/privacy_url.txt`**:
```text
https://example.com/privacy
```

**创建 `fastlane/metadata/copyright.txt`**:
```text
2026 Your Company Name
```

### Step 9.5 — HIG 品牌 & 应用内文案审查

> **Claude Code 必须审查 App 内所有文案是否符合 HIG 写作标准。**

#### 品牌原则 (HIG)
```
✅ 品牌始终让位于内容 — 用户来使用功能，不是看品牌展示
✅ 避免在整个应用中反复显示 Logo
✅ 选择强调色 (accent color) 建立品牌识别
✅ 自定义字体仅用在标题 (正文用系统字体)
✅ 启动画面不是品牌展示机会
```

#### 应用内文案检查清单

Claude Code 扫描所有 `.swift` 文件中的 `Text()` 和 `Label()` 文案:

```bash
# 提取所有硬编码文案
grep -rn 'Text("' ${PROJECT_NAME}/ --include="*.swift" | grep -v 'Preview' > app_strings_audit.txt

# Claude Code 逐条审查:
# □ 是否有生僻词或行业术语? → 替换为平实语言
# □ 按钮文案是否是动词? → "保存" "删除" "开始" (不是名词)
# □ 错误提示是否友好? → 不说 "Error 500"，说 "网络似乎有问题，请稍后重试"
# □ 是否直接称呼用户? → "你的任务" 而非 "用户的"
# □ 是否假设用户性别/年龄/身份? → 使用中性语言
# □ 感叹号是否过多? → HIG 建议少用
# □ 空状态文案是否有帮助? → 不只说"无数据"，而说"添加你的第一个项目"
```

#### 文案本地化准备 (HIG)

```swift
// ✅ 使用 String(localized:) 支持本地化
// ${PROJECT_NAME}/Resources/Localizable.xcstrings
Text("add_task_button", bundle: .main)  // 从 String Catalog 加载
    .font(.appBody)

// 创建本地化字符串目录 (Xcode 15+/iOS 17+)
// File → New → String Catalog → Localizable.xcstrings
// 支持: zh-Hans, en, zh-Hant, ja, ko 等

// ✅ 格式化数字/日期 (自动适配地区)
Text(itemCount, format: .number)          // 1,234 (en) / 1.234 (de)
Text(dueDate, format: .dateTime)          // 根据系统地区格式化

// ✅ 等宽数字 (表格对齐)
Text("\(price, format: .currency(code: "CNY"))")
    .monospacedDigit()
```

> **Claude Code 执行**: 完成素材准备 & 文案审查 → `git commit -m "Add App Store metadata, HIG branding & copy audit"`

---

## Phase 10: 内购 & 订阅配置 (4:30 - 5:30)

> **[GENERATE]** + **[WRITE]** + **[DIALOG]** StoreKit 2 集成；ASC 配置需用户手动操作
> ⚠️ 仅当 SPECS.md 中选择了盈利模式时才执行此阶段

### Step 10.1 — App Store Connect 配置 (Claude Code 指导用户手动操作)

> **[DIALOG]** Claude Code 无法直接操作 ASC，输出详细步骤让用户执行

> **Claude Code 无法直接操作 App Store Connect，需输出详细步骤让用户执行**:

```
请在 App Store Connect 执行以下操作:

1. 进入 https://appstoreconnect.apple.com
2. "App 内购买项目" → 创建
3. 选择类型:
   [ ] 消耗型 (Consumable) — 如金币
   [ ] 非消耗型 (Non-Consumable) — 如去广告
   [ ] 自动续期订阅 (Auto-Renewable Subscription)
   [ ] 非续期订阅 (Non-Renewing Subscription)
4. 填写参考名称、产品 ID (如 com.example.app.premium)
5. 设置价格、订阅时长等
6. 保存后记录 Product ID，后续代码中使用
```

### Step 10.2 — StoreKit 2 集成

**`Services/IAPService.swift`**:

```swift
// ${PROJECT_NAME}/Services/IAPService.swift
import StoreKit
import SwiftUI

@MainActor
final class IAPService: ObservableObject {
    static let shared = IAPService()
    
    @Published var products: [Product] = []
    @Published var purchasedProductIDs: Set<String> = []
    @Published var isLoadingProducts = false
    
    private let productIDs = [
        "com.example.app.premium_monthly",
        "com.example.app.premium_yearly",
        "com.example.app.lifetime",
        "com.example.app.remove_ads"
    ]
    
    private var updates: Task<Void, Never>?
    
    private init() {
        updates = listenForTransactions()
    }
    
    deinit {
        updates?.cancel()
    }
    
    // MARK: - Load Products
    
    func loadProducts() async {
        isLoadingProducts = true
        defer { isLoadingProducts = false }
        
        do {
            products = try await Product.products(for: productIDs)
                .sorted { $0.price < $1.price }
        } catch {
            print("Failed to load products: \(error)")
        }
    }
    
    // MARK: - Purchase
    
    func purchase(_ product: Product) async throws {
        let result = try await product.purchase()
        
        switch result {
        case .success(let verification):
            let transaction = try checkVerified(verification)
            await handlePurchase(transaction.productID)
            await transaction.finish()
            
        case .userCancelled:
            break
            
        case .pending:
            break
            
        @unknown default:
            break
        }
    }
    
    // MARK: - Restore
    
    func restorePurchases() async {
        for await result in Transaction.currentEntitlements {
            if case .verified(let transaction) = result {
                await handlePurchase(transaction.productID)
            }
        }
    }
    
    // MARK: - Check Status
    
    var isPremium: Bool {
        !purchasedProductIDs.isEmpty
    }
    
    func isPurchased(_ productID: String) -> Bool {
        purchasedProductIDs.contains(productID)
    }
    
    // MARK: - Private
    
    private func listenForTransactions() -> Task<Void, Never> {
        Task.detached {
            for await result in Transaction.updates {
                if case .verified(let transaction) = result {
                    await self.handlePurchase(transaction.productID)
                    await transaction.finish()
                }
            }
        }
    }
    
    private func checkVerified<T>(_ result: VerificationResult<T>) throws -> T {
        switch result {
        case .verified(let safe):
            return safe
        case .unverified:
            throw AppError.validationError(message: "Purchase verification failed")
        }
    }
    
    private func handlePurchase(_ productID: String) async {
        purchasedProductIDs.insert(productID)
        // 持久化购买状态
        try? await StorageService.shared.save(
            Array(purchasedProductIDs),
            forKey: "purchased_product_ids"
        )
    }
}
```

### Step 10.3 — 付费墙 UI

**`Views/PaywallView.swift`**:

```swift
// ${PROJECT_NAME}/Views/PaywallView.swift
import SwiftUI
import StoreKit

struct PaywallView: View {
    @Environment(\.dismiss) var dismiss
    @StateObject private var iapService = IAPService.shared
    @State private var selectedProduct: Product?
    @State private var isPurchasing = false
    @State private var purchaseError: String?
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 24) {
                    // Header
                    VStack(spacing: 8) {
                        Image(systemName: "crown.fill")
                            .font(.system(size: 60))
                            .foregroundColor(.appWarning)
                        Text("升级高级版")
                            .font(.appTitle)
                        Text("解锁所有功能，提升使用体验")
                            .font(.appBody)
                            .foregroundColor(.secondary)
                    }
                    .padding(.top, 20)
                    
                    // Features
                    VStack(spacing: 16) {
                        FeatureRow(icon: "infinity", title: "无限制使用")
                        FeatureRow(icon: "icloud.fill", title: "iCloud 同步")
                        FeatureRow(icon: "paintbrush.fill", title: "个性化主题")
                        FeatureRow(icon: "bell.badge.fill", title: "智能提醒")
                    }
                    .padding()
                    .cardStyle()
                    
                    // Products
                    if iapService.isLoadingProducts {
                        ProgressView()
                    } else {
                        ForEach(iapService.products) { product in
                            ProductRow(
                                product: product,
                                isSelected: selectedProduct?.id == product.id,
                                onSelect: { selectedProduct = product }
                            )
                        }
                    }
                    
                    // Purchase Button
                    Button {
                        purchase()
                    } label: {
                        HStack {
                            if isPurchasing {
                                ProgressView()
                                    .tint(.white)
                            }
                            Text("开始使用")
                                .fontWeight(.semibold)
                        }
                        .frame(maxWidth: .infinity)
                        .padding(.vertical, 16)
                        .background(Color.appPrimary)
                        .foregroundColor(.white)
                        .cornerRadius(12)
                    }
                    .disabled(selectedProduct == nil || isPurchasing)
                    .padding(.top)
                    
                    // Restore
                    Button("恢复购买") {
                        Task { await iapService.restorePurchases() }
                    }
                    .font(.caption)
                    .foregroundColor(.secondary)
                    
                    // Legal
                    HStack(spacing: 4) {
                        Link("隐私政策", destination: URL(string: AppConstants.privacyURL)!)
                        Text("·")
                        Link("服务条款", destination: URL(string: AppConstants.termsURL)!)
                    }
                    .font(.caption2)
                    .foregroundColor(.secondary)
                }
                .padding()
            }
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("关闭") { dismiss() }
                }
            }
        }
    }
    
    private func purchase() {
        guard let product = selectedProduct else { return }
        isPurchasing = true
        Task {
            do {
                try await iapService.purchase(product)
                dismiss()
            } catch {
                purchaseError = error.localizedDescription
            }
            isPurchasing = false
        }
    }
}

struct FeatureRow: View {
    let icon: String
    let title: String
    
    var body: some View {
        HStack(spacing: 12) {
            Image(systemName: icon)
                .font(.title3)
                .foregroundColor(.appPrimary)
                .frame(width: 30)
            Text(title)
                .font(.appBody)
            Spacer()
            Image(systemName: "checkmark")
                .foregroundColor(.appSuccess)
        }
    }
}

struct ProductRow: View {
    let product: Product
    let isSelected: Bool
    let onSelect: () -> Void
    
    var body: some View {
        Button(action: onSelect) {
            HStack {
                VStack(alignment: .leading, spacing: 4) {
                    Text(product.displayName)
                        .font(.headline)
                    Text(product.description)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                Spacer()
                Text(product.displayPrice)
                    .fontWeight(.semibold)
            }
            .padding()
            .background(
                RoundedRectangle(cornerRadius: 12)
                    .stroke(isSelected ? Color.appPrimary : Color(.systemGray5), lineWidth: 2)
            )
        }
        .buttonStyle(.plain)
    }
}
```

> **Claude Code 执行**: 完成 IAP 配置 → `git commit -m "Add IAP integration & paywall"`

---

## Phase 11: Archive & TestFlight (5:30 - 6:30)

> **[SHELL]** + **[DIALOG]** 签名验证 → Archive → 上传 ASC

### Step 11.1 — 代码签名配置

> **[SHELL]** 自动检测 Team ID 和证书

```bash
# 检测 Team ID
TEAM_ID=$(security find-identity -v -p codesigning 2>/dev/null | \
  grep "Apple Development" | \
  head -1 | \
  grep -oE '[A-Z0-9]{10}' | \
  head -1)

echo "Detected Team ID: ${TEAM_ID}"

# 如果为空，提示用户
if [ -z "$TEAM_ID" ]; then
  echo "❌ 未检测到有效的 Apple Developer 证书"
  echo "请在 Xcode → Settings → Accounts 中登录 Apple ID"
  echo "并确保已创建 Development Certificate"
  exit 1
fi
```

### Step 11.2 — 自动更新 Build Number

```bash
source /tmp/sop_project.env 2>/dev/null
cd ${PROJECT_DIR}

# 获取当前 build number
CURRENT_BUILD=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" \
  "${PROJECT_NAME}/Resources/Info.plist" 2>/dev/null || echo "1")

# 递增
NEW_BUILD=$((CURRENT_BUILD + 1))

# 更新 Info.plist
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion ${NEW_BUILD}" \
  "${PROJECT_NAME}/Resources/Info.plist"

echo "Build number updated: ${CURRENT_BUILD} → ${NEW_BUILD}"
```

### Step 11.3 — Archive

```bash
source /tmp/sop_project.env 2>/dev/null
cd ${PROJECT_DIR}

# 清理构建目录
xcodebuild clean \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -configuration Release

# 创建 Archive
ARCHIVE_PATH="${PROJECT_DIR}/build/${PROJECT_NAME}.xcarchive"

xcodebuild archive \
  -project ${PROJECT_NAME}.xcodeproj \
  -scheme ${PROJECT_NAME} \
  -configuration Release \
  -archivePath "${ARCHIVE_PATH}" \
  -destination 'generic/platform=iOS' \
  -allowProvisioningUpdates \
  -authenticationKeyIssuerID "${APPSTORE_ISSUER_ID}" \
  -authenticationKeyID "${APPSTORE_KEY_ID}" \
  -authenticationKeyPath "${APPSTORE_KEY_PATH}" \
  2>&1 | tee archive.log

# 检查结果
if [ $? -eq 0 ]; then
  echo "✅ Archive succeeded: ${ARCHIVE_PATH}"
else
  echo "❌ Archive failed. Check archive.log for details."
  # Claude Code 自动分析 archive.log 错误
fi
```

> ⚠️ **App Store Connect API Key**: 用户需要预先创建并配置以下环境变量:
> - `APPSTORE_ISSUER_ID`
> - `APPSTORE_KEY_ID`
> - `APPSTORE_KEY_PATH`

### Step 11.4 — 创建 ExportOptions.plist

> **[WRITE]** 创建导出配置，Claude Code 自动生成

```bash
# [SHELL] 加载项目变量
source /tmp/sop_project.env 2>/dev/null
cd ${PROJECT_DIR}
```

**创建 `${PROJECT_DIR}/ExportOptions.plist`**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>app-store</string>
    <key>teamID</key>
    <string>${TEAM_ID}</string>
    <key>uploadBitcode</key>
    <false/>
    <key>uploadSymbols</key>
    <true/>
    <key>compileBitcode</key>
    <false/>
    <key>destination</key>
    <string>upload</string>
    <key>signingStyle</key>
    <string>automatic</string>
    <key>stripSwiftSymbols</key>
    <true/>
    <key>manageAppVersionAndBuildNumber</key>
    <false/>
</dict>
</plist>
```

### Step 11.5 — 导出 IPA & 上传 ASC

> **[SHELL]** 导出 IPA 并上传。如果 ASC API Key 未配置，输出手动步骤。

```bash
# [SHELL] 导出 IPA
xcodebuild -exportArchive \
  -archivePath "${ARCHIVE_PATH}" \
  -exportPath "./build/${PROJECT_NAME}" \
  -exportOptionsPlist "./ExportOptions.plist" \
  2>&1 | tail -10

# [SHELL] 如果配置了 API Key，自动上传
if [ -n "${APPSTORE_KEY_ID}" ] && [ -n "${APPSTORE_ISSUER_ID}" ] && [ -f "${APPSTORE_KEY_PATH}" ]; then
    echo "✅ API Key 已配置，正在上传到 App Store Connect..."
    xcrun altool --upload-app \
      --type ios \
      --file "./build/${PROJECT_NAME}/${PROJECT_NAME}.ipa" \
      --apiKey "${APPSTORE_KEY_ID}" \
      --apiIssuer "${APPSTORE_ISSUER_ID}" \
      --verbose
    echo "✅ Upload completed. Visit https://appstoreconnect.apple.com to continue."
else
    echo "⚠️ ASC API Key 未配置。请按以下步骤创建:"
    echo ""
    echo "=== 创建 App Store Connect API Key ==="
    echo "1. 登录 https://appstoreconnect.apple.com"
    echo "2. 用户和访问 → 密钥 → App Store Connect API"
    echo "3. 点击 + 生成 API Key，输入名称 (如 'CI Upload')"
    echo "4. 选择访问权限: Developer"
    echo "5. 下载 .p8 私钥文件 (仅此一次!)"
    echo "6. 记录: Issuer ID, Key ID, .p8 文件路径"
    echo ""
    echo "然后设置环境变量并重新运行上传:"
    echo '  export APPSTORE_KEY_ID="ABC1234567"'
    echo '  export APPSTORE_ISSUER_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"'
    echo '  export APPSTORE_KEY_PATH="/path/to/AuthKey_ABC1234567.p8"'
    echo ""
    echo "或直接打开 Xcode → Window → Organizer → 选择 Archive → Distribute App"
fi
```

---

## Phase 12: App Store Connect 配置 & 提交 (6:30 - 7:30)

> **[DIALOG]** + **[VALIDATE]** Claude Code 准备所有素材和清单，用户执行 ASC 操作

### Step 12.1 — 配置 App Store Connect (Claude Code 指导用户)

> **[DIALOG]** Claude Code 无法直接操作 ASC，输出完整操作指南

> **Claude Code 无法直接操作 App Store Connect，以下为详细操作指南**:

```
# Claude Code 已为你准备好所有素材和元数据。
# 请按以下步骤在 App Store Connect 中完成配置：

## 1. App 信息页面
□ App 名称: ${DISPLAY_NAME}
□ 类别: 请选择 (参考 fastlane/metadata/ 目录中的配置)
□ 年龄分级: 请填写问卷

## 2. 价格与销售范围
□ 价格: 免费 / 付费 (参考 SPECS.md)
□ 销售范围: 全球 (或指定区域)

## 3. 隐私政策
□ 隐私政策 URL: (从 fastlane/metadata/ 中复制)

## 4. App 审核信息
□ 联系信息: 填写可用的邮箱和电话
□ 备注: (可选，向审核团队说明特殊功能)

## 5. 版本信息
□ 宣传文本: (从 fastlane/metadata/zh-Hans/promotional_text.txt 复制)
□ 描述: (从 fastlane/metadata/zh-Hans/description.txt 复制)
□ 关键词: (从 fastlane/metadata/zh-Hans/keywords.txt 复制)
□ 技术支持网址: (从 fastlane/metadata/zh-Hans/support_url.txt 复制)

## 6. App 截图和预览
□ 上传截图 (位置: fastlane/screenshots/)

## 7. 提交审核 ⬅️ 最后一步!
□ 确认所有信息填写完整
□ 确认构建版本已选择 (在 "构建版本" 区域点击 ⊕ 选择已上传的 build)
□ 点击右上角 "提交审核"
□ 回答 出口合规 / 广告标识符 相关问题
□ 确认提交!
```

> ⚠️ **构建版本选择**: 上传后需要等 15-30 分钟处理。如果 "构建版本" 区域没有可选项 → 等待或检查邮件 (可能被拒)。

### Step 12.2 — 使用 Fastlane 自动化提交 (高级)

**创建 `fastlane/Fastfile`**:

```ruby
# fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Push a new release build to the App Store"
  lane :release do
    # 增加 build number
    increment_build_number

    # 运行测试
    run_tests(scheme: "${PROJECT_NAME}")

    # 构建
    build_app(
      scheme: "${PROJECT_NAME}",
      export_method: "app-store"
    )

    # 上传到 App Store Connect
    upload_to_app_store(
      skip_metadata: false,
      skip_screenshots: false,
      automatic_release: false  # 手动发布
    )

    # 发送通知
    slack(
      message: "New build #{lane_context[SharedValues::BUILD_NUMBER]} uploaded to App Store Connect"
    )
  end

  desc "Upload to TestFlight"
  lane :beta do
    increment_build_number
    build_app(scheme: "${PROJECT_NAME}")
    upload_to_testflight
  end
end
```

### Step 12.3 — 最终提交检查清单 (HIG 完整合并)

```markdown
# Claude Code 最终检查清单 — ${PROJECT_NAME}

## 代码质量
- [ ] 无编译警告 (xcodebuild -quiet)
- [ ] 无 SwiftLint 错误
- [ ] 单元测试全部通过
- [ ] UI 测试全部通过

## 安全 & 隐私
- [ ] PrivacyInfo.xcprivacy 已配置
- [ ] ITSAppUsesNonExemptEncryption = NO (如无加密)
- [ ] NSAppTransportSecurity 已配置 (如使用 HTTP)
- [ ] 无硬编码 API Key
- [ ] 敏感数据使用 Keychain 存储
- [ ] 仅请求真正需要的数据访问权限 (HIG)
- [ ] 权限请求说明为什么需要和好处 (HIG)

## App Store 合规
- [ ] App 图标 (1024x1024) 已准备
- [ ] App 图标使用分层设计 (HIG: 背景 + 前景层，Liquid Glass 属性)
- [ ] 截图至少 1 套 (6.7" 必需)
- [ ] 隐私政策 URL 可访问
- [ ] 技术支持 URL 可访问
- [ ] 无私有 API 调用
- [ ] 无未声明的权限请求
- [ ] App 未使用 SF Symbols 作为图标/Logo (HIG: 禁止)

## HIG 可及性 (Accessibility)
- [ ] 支持 Dynamic Type (至少 200% 放大) (HIG)
- [ ] 色彩对比度达到 WCAG AA 标准 (文本 4.5:1, 非文本 3:1) (HIG)
- [ ] Dark Mode + Increase Contrast 下也满足对比度 (HIG)
- [ ] 所有交互元素 .accessibilityLabel() 已设置 (HIG)
- [ ] 所有装饰性图像 .accessibilityHidden(true) (HIG)
- [ ] VoiceOver 焦点导航顺序合理 (HIG)
- [ ] 使用 Accessibility Inspector 审计过 (HIG)
- [ ] 图标和图像包容性 (SF Symbols 性别中立) (HIG)
- [ ] Reduce Motion ON 时动画已移除 (HIG)
- [ ] Reduce Transparency ON 时 UI 仍然可用 (HIG)

## HIG 外观适配
- [ ] 亮色/暗色模式下都测试通过 (HIG)
- [ ] Increase Contrast 开启时正常 (HIG)
- [ ] Reduce Transparency 开启时正常 (HIG)
- [ ] 未提供应用内独立外观设置 (HIG: 尊重系统设置)
- [ ] 语义颜色使用系统 API (HIG: 自动适配外观)

## HIG 交互设计
- [ ] 模态使用必要且简单、有标题、有明确关闭方式 (HIG)
- [ ] 模态关闭前如有数据丢失风险已确认 (HIG)
- [ ] 控件在屏幕中下部便于手持操作 (HIG: iOS 人体工学)
- [ ] 所有触摸目标 ≥ 44×44pt (HIG)
- [ ] 加载尽快展示内容 — 先占位符再实际内容 (HIG)
- [ ] 进度指示: 已知时长用确定条 / 未知时长用不确定指示 (HIG)
- [ ] 支持多任务和状态恢复 (HIG)
- [ ] 手势有替代操作方式 (HIG)
- [ ] 反馈匹配信息重要性 (Alert/Toast/内嵌/触觉) (HIG)
- [ ] 后台操作使用 Background Tasks 框架 (HIG)

## HIG 导航体验
- [ ] 导航层级 ≤ 3-4 层深 (HIG)
- [ ] 标签栏 3-5 个标签，不超 5 个 (HIG)
- [ ] 标签栏切换不重置页面状态 (HIG)
- [ ] 搜索无结果时显示 ContentUnavailableView (HIG)
- [ ] 次要操作放入 .contextMenu 而非铺满按钮 (HIG)

## HIG 视觉设计
- [ ] Liquid Glass 仅用于控件层/导航层，不在内容层使用 (HIG)
- [ ] Liquid Glass 节制使用 (HIG)
- [ ] 媒体背景上的组件使用 clear 变体 (HIG)
- [ ] 字体: 优先 Regular/Medium/Semibold/Bold (HIG)
- [ ] 最小字号 ≥ 11pt (iOS) (HIG)
- [ ] 细体字有更大的推荐尺寸 (HIG)
- [ ] SF Symbols 渲染模式合理 (.palette / .hierarchical) (HIG)

## HIG 内容 & 品牌
- [ ] 品牌始终让位于内容 (HIG)
- [ ] 未在整个应用中反复显示 Logo (HIG)
- [ ] 启动画面与第一屏几乎相同，无文字/广告 (HIG)
- [ ] 文案清晰简洁，以行动为导向 (HIG)
- [ ] 直接称呼用户 "你/你的" (HIG)
- [ ] 包容性语言，无冒犯性表达 (HIG)
- [ ] 设置最小化，不重复系统全局选项 (HIG)

## HIG 引导 & 权限
- [ ] 引导页可选/可跳过 (HIG)
- [ ] 权限请求推迟到用户需要使用功能时 (HIG)
- [ ] 权限请求不在启动时轰炸用户 (HIG)

## 用户体验
- [ ] 启动屏幕显示正常 (HIG: 与第一屏几乎相同)
- [ ] 无崩溃 (在最新 OS 上验证)
- [ ] 离线状态处理完善
- [ ] 加载状态指示明确
- [ ] 错误提示用户友好
- [ ] 空状态有引导文案 (不是仅 "暂无数据")

## 性能
- [ ] 启动时间 < 2 秒
- [ ] 无主线程阻塞
- [ ] 内存使用正常
- [ ] 无循环引用导致的内存泄漏
\`\`\`

> **Claude Code 逐项确认后**: `git commit -m "Final HIG compliance checklist verified"`

---

## Phase 13: 提交审核 & 文档归档 (7:30 - 8:00)

> **[GIT]** + **[GENERATE]** + **[VALIDATE]** 最终提交、生成文档、输出报告

### Step 13.1 — 最终提交

> **[GIT]** 打 tag、push

```bash
# 最终代码提交
git add -A
git commit -m "Release v1.0.0 (Build ${NEW_BUILD}) - Ready for App Store submission"

# 创建 tag
git tag -a "v1.0.0" -m "Release v1.0.0 - Initial App Store submission"
```

### Step 13.2 — 生成项目文档

**创建 `README.md`**:

```markdown
# ${PROJECT_NAME}

## 项目概述
${APP_DESCRIPTION}

## 技术栈
- iOS ${DEPLOYMENT_TARGET}+
- SwiftUI
- MVVM 架构
- SwiftData 本地存储
- StoreKit 2 内购

## 项目结构
\`\`\`
${PROJECT_NAME}/
├── App/                    # 应用入口
├── Models/                 # 数据模型
├── ViewModels/             # 视图模型
├── Views/                  # 视图层
│   └── Components/         # 可复用组件
├── Services/               # 服务层
├── Utilities/              # 工具类
├── Resources/              # 资源文件
├── fastlane/               # CI/CD 配置
└── ${PROJECT_NAME}Tests/   # 测试
\`\`\`

## 快速开始
1. \`git clone [repo_url]\`
2. 打开 \`${PROJECT_NAME}.xcodeproj\`
3. 选择开发团队 (Signing & Capabilities)
4. \`Cmd + R\` 运行

## 发布流程
- 测试: \`fastlane beta\`
- 发布: \`fastlane release\`
\`\`\`
```

### Step 13.3 — 生成 SOP 执行报告

**创建 `SOP_EXECUTION_REPORT.md`**:

```markdown
# SOP 执行报告 — ${PROJECT_NAME}

## 基本信息
- 执行日期: $(date +%Y-%m-%d)
- 总耗时: [实际耗时]
- App 名称: ${PROJECT_NAME}
- Bundle ID: ${BUNDLE_ID}
- 版本: 1.0.0 (Build ${NEW_BUILD})

## 各阶段完成情况

| 阶段 | 状态 | 备注 |
|------|------|------|
| Phase 0: 环境检查 | ✅ | |
| Phase 1: 需求确认 | ✅ | |
| Phase 2: 架构搭建 | ✅ | |
| Phase 3: 数据层 | ✅ | |
| Phase 4: ViewModel | ✅ | |
| Phase 5: UI 层 | ✅ | |
| Phase 6: 自测 | ✅ | |
| Phase 7: 集成测试 | ✅ | |
| Phase 8: 性能优化 | ✅ | |
| Phase 9: 素材准备 | ✅ | |
| Phase 10: 内购配置 | [✅/N/A] | |
| Phase 11: Archive | ✅ | |
| Phase 12: ASC 配置 | ✅ | |
| Phase 13: 提交 | ✅ | |

## App Store 状态
- TestFlight: [已上传/待上传]
- 审核状态: [已提交/待提交]
- 预计上线: [日期]

## 技术债务 & 后续优化建议
1. [ ] 
2. [ ] 
3. [ ] 

## 签名
- 开发者: $(git config user.name)
- 邮箱: $(git config user.email)
\`\`\`
```

### Step 13.4 — App Store 审核防拒指南

> **[VALIDATE]** Claude Code 在提交前对照常见拒绝原因逐项检查。

```
App Store 审核 Top 10 拒绝原因 & 本 SOP 防范措施:

1. ❌ 崩溃/Bug → Phase 6/7 自动化测试 + 多模拟器验证
2. ❌ 链接损坏 → Phase 9.4 确认隐私政策/支持 URL 可访问
3. ❌ 占位符内容 → Phase 5 原型审查确保无 "Lorem Ipsum"
4. ❌ 权限请求信息不完整 → Phase 7.3 权限请求包含理由说明
5. ❌ 不完整的信息 → Phase 12.1 ASC 元数据完整填写
6. ❌ 误导用户 → Phase 9.5 文案审查确保描述诚实
7. ❌ 界面丑陋 → Phase 1.5 高保真原型 + HIG 合规
8. ❌ 重复 App/Spam → 确保 App 有独特功能价值
9. ❌ 未使用 Sign in with Apple → (如涉及第三方登录) Phase 3 集成
10. ❌ 缺少隐私清单 → Phase 8.3 PrivacyInfo.xcprivacy
```

### Step 13.5 — Sign in with Apple 集成 (如 SPECS 需要)

> **[GENERATE]** 仅当 Phase 1 用户选择了需要用户系统时执行

```swift
// ${PROJECT_NAME}/Services/AuthService.swift
import AuthenticationServices
import CryptoKit
import SwiftUI

@MainActor
final class AuthService: NSObject, ObservableObject {
    static let shared = AuthService()

    @Published var isAuthenticated = false
    @Published var userIdentifier: String?

    private var currentNonce: String?

    func handleSignInWithApple() {
        let nonce = randomNonceString()
        currentNonce = nonce

        let request = ASAuthorizationAppleIDProvider().createRequest()
        request.requestedScopes = [.fullName, .email]
        request.nonce = sha256(nonce)

        let controller = ASAuthorizationController(authorizationRequests: [request])
        controller.delegate = self
        controller.performRequests()
    }

    // Apple 登录按钮 (HIG 标准样式)
    func signInWithAppleButton() -> some View {
        SignInWithAppleButton(
            .signIn,
            onRequest: { request in
                let nonce = randomNonceString()
                self.currentNonce = nonce
                request.requestedScopes = [.fullName, .email]
                request.nonce = sha256(nonce)
            },
            onCompletion: { result in
                switch result {
                case .success(let auth):
                    if let credential = auth.credential as? ASAuthorizationAppleIDCredential {
                        self.userIdentifier = credential.user
                        self.isAuthenticated = true
                    }
                case .failure(let error):
                    print("Sign in with Apple failed: \(error)")
                }
            }
        )
        .signInWithAppleButtonStyle(.black)  // HIG: 深色/白色两种
        .frame(height: 50)
        .cornerRadius(ProtoCorner.md)
    }

    private func randomNonceString(length: Int = 32) -> String {
        precondition(length > 0)
        let charset: [Character] = Array("0123456789ABCDEFGHIJKLMNOPQRSTUVXYZabcdefghijklmnopqrstuvwxyz-._")
        var result = ""
        var remainingLength = length
        while remainingLength > 0 {
            var randoms = [UInt8](repeating: 0, count: 16)
            let errorCode = SecRandomCopyBytes(kSecRandomDefault, randoms.count, &randoms)
            if errorCode != errSecSuccess { break }
            for random in randoms {
                if remainingLength == 0 { break }
                if random < charset.count { result.append(charset[Int(random)]); remainingLength -= 1 }
            }
        }
        return result
    }

    private func sha256(_ input: String) -> String {
        let data = Data(input.utf8)
        let hash = SHA256.hash(data: data)
        return hash.compactMap { String(format: "%02x", $0) }.joined()
    }
}

extension AuthService: ASAuthorizationControllerDelegate {
    func authorizationController(controller: ASAuthorizationController,
                                  didCompleteWithAuthorization authorization: ASAuthorization) {
        if let credential = authorization.credential as? ASAuthorizationAppleIDCredential {
            userIdentifier = credential.user
            isAuthenticated = true
        }
    }

    func authorizationController(controller: ASAuthorizationController, didCompleteWithError error: Error) {
        print("Authorization failed: \(error.localizedDescription)")
    }
}
```

> **注意**: App Store 审核要求: 如果 App 包含任何第三方登录 (Google/Facebook/微信等)，**必须**同时提供 Sign in with Apple 选项。

### Step 13.6 — 完成收尾

> **[SHELL]** 输出最终汇总

```bash
source /tmp/sop_project.env 2>/dev/null
echo "
╔══════════════════════════════════════════╗
║                                          ║
║   🎉 iOS App 开发完成!                   ║
║                                          ║
║   App: ${PROJECT_NAME}                   ║
║   Version: 1.0.0 (Build ${NEW_BUILD})    ║
║   Status: 已上传至 App Store Connect     ║
║                                          ║
║   下一步:                                ║
║   1. 在 ASC 中完成元数据填写             ║
║   2. 提交审核                            ║
║   3. 等待审核结果 (通常 24-48 小时)      ║
║                                          ║
╚══════════════════════════════════════════╝
"
```

---

# 📚 附录

## A. 快速命令参考

```bash
# 构建 (Debug)
xcodebuild build -project ${PROJECT_NAME}.xcodeproj -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro'

# 运行测试
xcodebuild test -project ${PROJECT_NAME}.xcodeproj -scheme ${PROJECT_NAME} \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro'

# Archive (Release)
xcodebuild archive -project ${PROJECT_NAME}.xcodeproj -scheme ${PROJECT_NAME} \
  -configuration Release -archivePath ./build/App.xcarchive

# 导出 IPA
xcodebuild -exportArchive -archivePath ./build/App.xcarchive \
  -exportPath ./build/ -exportOptionsPlist ExportOptions.plist

# Git 操作
git add -A && git commit -m "message" && git push origin main
```

## B. 常见问题 & 解决方案

### Xcode 项目文件冲突
```bash
# 添加 .gitattributes
echo "*.pbxproj merge=union" >> .gitattributes
```

### 证书问题
```bash
# 查看可用证书
security find-identity -v -p codesigning

# 刷新 Provisioning Profiles
rm -rf ~/Library/MobileDevice/Provisioning\ Profiles/
```

### Swift Package 缓存问题
```bash
# 清理 SPM 缓存
rm -rf ~/Library/Developer/Xcode/DerivedData
rm -rf .build
xcodebuild -resolvePackageDependencies
```

## C. Claude Code 自动执行模式触发词

当用户说以下内容时，Claude Code 应自动开始执行 SOP:

- "按照 SOP 开发一个 iOS App"
- "两天开发一个 App"
- "执行 iOS 开发 SOP"
- "自动开发一个 [功能描述] App"

## D. 架构决策记录 (ADR)

| 决策 | 理由 |
|------|------|
| SwiftUI > UIKit | 开发速度更快，代码量减少 50%+ |
| MVVM > MVC | 更好的可测试性和代码组织 |
| SwiftData > Core Data | Apple 推荐的现代持久化方案，Swift 原生语法 |
| SPM > CocoaPods | Xcode 原生支持，无需额外工具 |
| StoreKit 2 > StoreKit 1 | 更简洁的 API，async/await 支持 |
| async/await > Combine | 代码更简洁，学习成本更低 |

## E. HIG 知识库 — SOP 集成映射

> 以下表格展示 HIG 知识库的每个知识领域在本 SOP 中的落地位置。

| HIG 知识领域 | SOP 集成位置 | 实现方 |
|-------------|-------------|--------|
| **三大设计原则** (层级/和谐/一致性) | Phase 2 Step 2.0 — iOS 平台设计原则 | 架构指导 |
| **iOS 平台特征** (屏幕/人体工学/输入) | Phase 2 Step 2.0 | 架构指导 |
| **可及性 — Dynamic Type** | Phase 8 Step 8.2.2 | @ScaledMetric, 系统字体样式 |
| **可及性 — WCAG 对比度** | Phase 8 Step 8.2.3 | 系统语义颜色, 4.5:1 标准 |
| **可及性 — VoiceOver** | Phase 8 Step 8.2.4 | .accessibilityLabel/Hint/Traits |
| **可及性 — Reduce Motion** | Phase 8 Step 8.2.5 | @Environment(\.accessibilityReduceMotion) |
| **可及性 — Reduce Transparency** | Phase 8 Step 8.2.6 | @Environment(\.accessibilityReduceTransparency) |
| **可及性 — 多通道反馈** | Phase 8 Step 8.2.7 + Phase 5 Step 5.1 (HapticFeedback) | 视觉+声音+触觉 |
| **可及性 — 包容性** | Phase 8 Step 8.2.8 | SF Symbols 中性, 文案审查 |
| **App 图标 — 分层设计** | Phase 9 Step 9.1 | Liquid Glass 属性, 矢量 |
| **颜色** | Phase 5 Step 5.1 (DesignTokens) | Asset Catalog Color Set, 亮/暗/高对比度变体 |
| **版式 — SF Pro** | Phase 5 Step 5.1 (Typography) | 系统字体样式, 最小 11pt |
| **版式 — Dynamic Type** | Phase 5 Step 5.1 (ScaledFont) + Phase 8 Step 8.2.2 | @ScaledMetric, ≥200% 放大 |
| **布局** | Phase 5 Step 5.1 (Layout Constants) | 44pt 触摸目标, 8pt 网格, 安全区 |
| **材质 — Liquid Glass** | Phase 5 Step 5.1 (AppMaterial) | regular/thin/ultraThin Material |
| **图标 — SF Symbols** | Phase 5 Step 5.1 (appIcon) | 四种渲染模式, 不能用于 Logo |
| **动效** | Phase 8 Step 8.2.5 | accessibleAnimation, Reduce Motion |
| **暗黑模式** | Phase 5 Step 5.1 (DarkModePreview) + Phase 8 Step 8.2.3 | ColorScheme 适配, 对比度验证 |
| **品牌** | Phase 9 Step 9.5 | 品牌让位于内容, Logo 使用规则 |
| **写作** | Phase 9 Step 9.4 + 9.5 | 清晰/包容/行动导向, 文案审查脚本 |
| **导航 — Tab Bar** | Phase 5 Step 5.3.1 | 3-5 标签, SF Symbols 填充 |
| **导航 — NavigationStack** | Phase 5 Step 5.3.2 | ≤ 3-4 层深, navigationTitle |
| **导航 — 搜索** | Phase 5 Step 5.3.3 | .searchable + .searchCompletion |
| **导航 — 侧边栏** | Phase 5 Step 5.3.4 | NavigationSplitView (iPad) |
| **模态** | Phase 5 Step 5.4.1-5.4.3 | .sheet/.alert/.confirmationDialog |
| **加载** | Phase 5 Step 5.1 (LoadingModifier) | 确定/不确定进度, 骨架屏 |
| **引导** | Phase 5 Step 5.5.1 | 可选/可跳过, 互动教学 |
| **启动** | Phase 5 Step 5.5.2 | UILaunchScreen, 与首屏相同 |
| **反馈** | Phase 5 Step 5.4.6 + Step 5.1 (HapticFeedback) | Alert/Toast/触觉 分级映射 |
| **通知管理** | Phase 7 Step 7.3 | 中断级别, Focus 模式 |
| **设置** | Phase 5 Step 5.5.3 | 最小化, 不重复系统设置 |
| **拖放** | (按需实现) | Drag and Drop API |
| **手势** | Phase 7 Step 7.3 | 不冲突, 有替代方式 |
| **隐私** | Phase 7 Step 7.3 + Phase 8 Step 8.3 | 仅请求必要权限, Privacy Manifest |
| **内容不可用** | Phase 5 Step 5.5.4 | ContentUnavailableView (iOS 17+) |
| **图片** | Phase 9 Step 9.1 | @2x/@3x, PNG/PDF/SVG |
| **包容性** | Phase 8 Step 8.2.8 + Phase 9 Step 9.5 | 语言/图标/文案 全面审查 |

---

> **SOP 版本**: 2.0.0 (HIG 集成版)
> **最后更新**: 2026-06-08
> **维护者**: Claude Code
> **适用范围**: 所有 iOS 项目 (可裁剪)
