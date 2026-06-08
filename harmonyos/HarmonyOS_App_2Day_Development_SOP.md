# 🦋 Claude Code + DevEco Studio 两天全自动鸿蒙 App 开发 SOP

> **适用对象**: Claude Code (AI Agent) 全自动执行
> **目标**: 从零到 AppGallery 上架，两天 (16 小时) 完成
> **技术栈**: ArkTS + ArkUI + Stage 模型 + RelationalStore
> **最低要求**: DevEco Studio 5.0+, HarmonyOS NEXT API 12+, 华为开发者账号
> **参考规范**: `HarmonyOS_Development_Guide.md`

---

## 🧠 Claude Code 技能调用矩阵

| 技能标识 | 技能名称 | 说明 | Claude Code 工具 |
|---------|---------|------|-----------------|
| `[SHELL]` | Shell 执行 | 执行 bash、hvigorw、hdc、git | `Bash` |
| `[WRITE]` | 文件写入 | 创建/修改 .ets .json5 .json 等文件 | `Edit` |
| `[READ]` | 文件读取 | 读取现有代码、日志、配置 | `Read` |
| `[DIALOG]` | 用户交互 | 向用户提问、请求确认、展示结果 | 对话输出 |
| `[GENERATE]` | 代码生成 | 生成 ArkTS 业务逻辑、UI、测试代码 | `Edit` |
| `[REVIEW]` | 代码审查 | 审查生成代码质量、ArkTS 规范合规 | `Read` → 分析 |
| `[DEBUG]` | 调试分析 | 分析编译错误/测试失败日志并修复 | `Read` + `Bash` + `Edit` |
| `[RESEARCH]` | 知识检索 | 查阅鸿蒙开发规范、API 文档 | 内置知识 + `Read` HOS_Guide |
| `[VALIDATE]` | 验证检查 | 对照清单逐项验证 | `Bash` + `Read` + 分析 |
| `[GIT]` | 版本控制 | git add/commit/tag/push | `Bash` |

### 技能调用原则

```
1. 每个 Phase 开始前 → [RESEARCH] 查阅 HarmonyOS_Development_Guide.md 对应章节
2. 每次 WRITE 代码后 → 立即 ./hvigorw assembleHap → [DEBUG] 自动分析和修复编译错误
3. 每个 Phase 结束前 → [VALIDATE] 对照该 Phase 验收条件逐项确认
4. 每个 Phase 结束后 → [GIT] 提交代码
5. 涉及不可逆操作 → [DIALOG] 必须获得用户确认
6. hvigorw 失败 → 读日志 → 定位文件 → Edit 修复 → 重编译, 最多循环 5 次
7. 无法自动修复 → [DIALOG] 报告具体错误文件和行号, 请求用户协助
```

### 全局编译验证模式 (每个 Phase 结束前执行)

```bash
# [SHELL] 标准编译验证块 — 每个 Phase 结束前复制此块
source /tmp/sop_harmony.env 2>/dev/null
cd ${PROJECT_DIR}
echo "=== 编译验证 ==="
./hvigorw assembleHap 2>&1 | tail -20
BUILD_EXIT=$?
if [ $BUILD_EXIT -ne 0 ]; then
  echo "❌ 编译失败! [DEBUG] 自动分析错误日志..."
  # [DEBUG] Claude Code 读取 hvigorw 输出, 定位 .ets 文件和行号, 用 Edit 修复
  ./hvigorw assembleHap 2>&1 | grep -E "ERROR|\.ets:[0-9]+"
else
  echo "✅ 编译通过"
fi
```

---

## 📋 总览时间线

```
Day 1 (8h)                              Day 2 (8h)
├─ [0.0h] 环境检查 & 项目初始化          ├─ [0.0h] Day 2 启动校验
├─ [0.5h] 产品需求确认                   ├─ [0.1h] 详情页 & 路由集成
├─ [1.0h] 高保真原型设计 & 设计系统       ├─ [1.0h] 设置页 & 搜索 & 手势
├─ [2.0h] Stage 架构搭建                ├─ [2.0h] 无障碍 & 设计规范验证
├─ [2.5h] 数据层 (Model + Store)        ├─ [2.5h] AppGallery 素材准备
│    └─ [NetworkService 可选]           ├─ [3.5h] 签名配置 + IAP(可选)
├─ [4.5h] ViewModel + [DI 容器]         ├─ [4.5h] 发布构建 & 云测试
├─ [6.0h] UI 层 (ArkUI) ← 按原型实现     ├─ [6.0h] AGC 提交审核 (24项清单)
└─ [7.5h] 自测 & 代码 Review             └─ [8.0h] 归档 & 文档
```

---

## ⚠️ 执行前提 — 环境准备 (先阅读)

> **重要**: hvigorw 不是全局命令，是项目目录下的 `./hvigorw` 包装脚本。
> DevEco Studio 创建项目时会自动生成此脚本。所有构建命令都在项目根目录执行 `./hvigorw`。

```bash
# [SHELL] 1. 确认 DevEco Studio 已安装
if [ -d "$HOME/Applications/DevEco-Studio.app" ] || [ -d "/Applications/DevEco-Studio.app" ]; then
    echo "✅ DevEco Studio 已安装"
else
    echo "❌ 未找到 DevEco Studio"
    echo "   下载: https://developer.huawei.com/consumer/cn/download/"
    echo "   安装: 双击 .dmg → 拖到 Applications"
    exit 1
fi

# [SHELL] 2. 确认 Git 已配置
git --version || { echo "❌ Git 未安装"; exit 1; }
git config user.name 2>/dev/null && git config user.email 2>/dev/null || {
    echo "❌ Git 未配置, 执行:"
    echo "   git config --global user.name 'Your Name'"
    echo "   git config --global user.email 'your@email.com'"
    exit 1
}

# [SHELL] 3. 确认华为开发者账号
echo "请确认:"
echo "  1. 已在 https://developer.huawei.com 注册账号"
echo "  2. 已实名认证 (个人/企业)"
echo "  3. 已签署开发者协议"
# [DIALOG] 等待用户确认 "已确认"
```

> **hvigorw 命令规范**: 全 SOP 中所有 `./hvigorw` 都在 `cd ${PROJECT_DIR}` 后执行。

---

# 🗓️ DAY 1 — 从零到可运行 MVP

---

## Phase 0: 环境初始化 (0:00 - 0:30)

> **[SHELL]** + **[DIALOG]** 检测环境 → 创建 DevEco Studio 项目 → 初始化 Git

### Step 0.1 — 创建鸿蒙项目 & 目录结构

```bash
# [DIALOG] 确认项目信息
PROJECT_NAME="MyHarmonyApp"
BUNDLE_NAME="com.example.myharmonyapp"
DISPLAY_NAME="我的鸿蒙App"

# 持久化项目变量
cat > /tmp/sop_harmony.env << HARMENV
PROJECT_NAME="$PROJECT_NAME"
BUNDLE_NAME="$BUNDLE_NAME"
DISPLAY_NAME="$DISPLAY_NAME"
PROJECT_DIR="$(pwd)/$PROJECT_NAME"
HARMENV
echo "✅ 项目变量已持久化"

# [DIALOG] 指导用户在 DevEco Studio 中创建项目:
echo "
请按以下步骤创建项目:
1. 打开 DevEco Studio → New Project → Empty Ability
2. Project name: ${PROJECT_NAME}
3. Bundle name: ${BUNDLE_NAME}
4. Save location: $(pwd)
5. API Level: 12+
6. Model: Stage
7. Language: ArkTS
8. 点击 Finish

创建完成后回复 '项目已创建' 继续
"
```

### Step 0.2 — 验证项目结构

> **[SHELL]** 确认项目文件完整，首次构建验证

```bash
source /tmp/sop_harmony.env 2>/dev/null
cd ${PROJECT_DIR}

# 验证关键文件
echo "=== 项目结构验证 ==="
ls AppScope/app.json5 && echo "✅ app.json5"
ls entry/src/main/module.json5 && echo "✅ module.json5"
ls entry/src/main/ets/pages/Index.ets && echo "✅ Index.ets"
ls entry/src/main/ets/entryability/EntryAbility.ets && echo "✅ EntryAbility.ets"
ls build-profile.json5 && echo "✅ build-profile.json5"
ls oh-package.json5 && echo "✅ oh-package.json5"

# 首次构建
echo "=== 首次构建验证 ==="
./hvigorw assembleHap 2>&1 | tail -10
# [DEBUG] 如有错误, 自动分析并修复
```

### Step 0.3 — 初始化 Git & 创建目录

```bash
cd ${PROJECT_DIR}

# [SHELL] 创建扩展目录结构
mkdir -p entry/src/main/ets/model
mkdir -p entry/src/main/ets/viewmodel
mkdir -p entry/src/main/ets/service
mkdir -p entry/src/main/ets/components
mkdir -p entry/src/main/ets/common
mkdir -p entry/src/main/resources/base/profile
mkdir -p prototype/screens
mkdir -p prototype/components

# [SHELL] 初始化 Git
git init && git checkout -b main

# [WRITE] .gitignore
cat > .gitignore << 'EOF'
/build/
/entry/build/
/.hvigor/
/.idea/
*.iml
local.properties
node_modules/
oh_modules/
.DS_Store
EOF

# [GIT]
git add -A && git commit -m "Initial HarmonyOS project structure"
```

---

## Phase 1: 产品需求澄清 (0:30 - 1:00)

> **[DIALOG]** + **[WRITE]** 结构化问卷 → 输出 SPECS.md

### Step 1.1 — 需求确认对话

```
[DIALOG] 确认以下信息:

1. App 核心功能 (一句话):
   [示例: "一个鸿蒙端智能备忘录"]

2. 目标设备:
   [ ] 手机 (phone)    [ ] 平板 (tablet)    [ ] 2in1    [ ] 全选

3. 是否需要后端:
   [ ] 纯本地    [ ] 云端 API    [ ] 华为云服务

4. 是否需要登录:
   [ ] 不需要    [ ] 华为账号登录    [ ] 手机号/邮箱

5. 盈利模式:
   [ ] 免费    [ ] 华为应用内支付 (IAP)    [ ] 付费下载

6. 是否需要元服务:
   [ ] 仅应用    [ ] 应用 + 元服务

7. UI 风格:
   [ ] 鸿蒙原生风格    [ ] 自定义品牌风格
```

### Step 1.2 — 输出 SPECS.md

> **[GENERATE]** + **[WRITE]** + **[DIALOG]** 整理用户回答，生成并请用户确认

```markdown
# 产品规格书 — ${PROJECT_NAME}

## 核心功能
- [ ] 功能1: xxx
- [ ] 功能2: xxx

## 技术选型
- UI: ArkUI 声明式
- 架构: Stage 模型 + MVVM
- 存储: RelationalStore
- 后端: ${BACKEND}

## 目标设备
- phone / tablet / 2in1

## 屏幕清单
1. 首页 (Index)
2. 详情页 (DetailPage)
3. 设置页 (SettingsPage)
```

---

## Phase 1.5: 高保真原型设计 (1:00 - 2:00)

> **[GENERATE]** + **[WRITE]** + **[DIALOG]** DevEco Studio Previewer 作为原型工具

### Step 1.5.1 — 设计系统定义

> **[RESEARCH]** + **[WRITE]** 生成 DESIGN_SPECS.md

```markdown
# 设计规格书

## 色彩系统
Primary: #007AFF    Secondary: #5856D6
Success: #34C759    Warning: #FF9500    Error: #FF3B30
Background: #F2F2F7 (grey) / Card: #FFFFFF

## 字体系统 (fp)
largeTitle: 28fp Bold    title: 20fp Medium
body: 16fp Regular       caption: 12fp Regular

## 间距 (8vp 网格)
xs:4  sm:8  md:16  lg:24  xl:32

## 圆角
组件: 12vp    按钮: 24vp (Capsule)    卡片: 16vp
```

### Step 1.5.2 — 生成原型文件

> **[WRITE]** 为每个屏幕创建原型 `prototype/screens/*.ets`

```typescript
// prototype/screens/HomePrototype.ets
// 首页原型 — DevEco Previewer 直接预览
@Entry
@Component
struct HomePrototype {
  @State selectedFilter: number = 0
  private filters: string[] = ['全部', '进行中', '已完成']

  build() {
    Column() {
      // 筛选标签
      Row({ space: 8 }) {
        ForEach(this.filters, (filter: string, index: number) => {
          Text(filter)
            .fontSize(14)
            .fontWeight(this.selectedFilter === index ? FontWeight.Bold : FontWeight.Normal)
            .fontColor(this.selectedFilter === index ? Color.White : '#333')
            .padding({ left: 16, right: 16, top: 6, bottom: 6 })
            .backgroundColor(this.selectedFilter === index ? '#007AFF' : '#E8E8ED')
            .borderRadius(20)
            .onClick(() => { this.selectedFilter = index })
        })
      }
      .padding(12)
      .width('100%')
      .justifyContent(FlexAlign.Start)

      // 任务列表
      List() {
        ForEach(['完成需求文档', '设计首页布局', '代码审查', '测试用例编写', '发布版本'],
          (item: string, index: number) => {
            ListItem() {
              Row({ space: 12 }) {
                Image($r('sys.media.ohos_ic_public_ok'))
                  .width(24).height(24)
                  .fillColor(index === 0 ? '#34C759' : '#C7C7CC')
                Column({ space: 4 }) {
                  Text(item).fontSize(16).fontColor('#1A1A1A')
                  if (index < 3) {
                    Text('截止 6月15日').fontSize(12).fontColor('#FF9500')
                  }
                }.alignItems(HorizontalAlign.Start).layoutWeight(1)
                Text(index < 2 ? '高' : '中')
                  .fontSize(11).fontColor(index < 2 ? '#FF3B30' : '#007AFF')
                  .padding({ left: 8, right: 8, top: 2, bottom: 2 })
                  .backgroundColor(index < 2 ? '#FF3B3010' : '#007AFF10')
                  .borderRadius(4)
              }
              .width('100%').padding(16)
              .backgroundColor(Color.White).borderRadius(12)
              .margin({ bottom: 8 })
            }
          })
      }
      .layoutWeight(1)
      .padding({ left: 16, right: 16 })
    }
    .width('100%').height('100%')
    .backgroundColor('#F2F2F7')
  }
}
```

> **[DIALOG]** 用户在 DevEco Studio 中打开原型文件 → Previewer 预览 → 回复确认

---

## Phase 2: Stage 架构搭建 (2:00 - 2:30)

> **[WRITE]** + **[GENERATE]** 生成 Stage 模型基础文件

### Step 2.1 — EntryAbility & 全局配置

> **[WRITE]** 更新 `EntryAbility.ets`

```typescript
// entry/src/main/ets/entryability/EntryAbility.ets
import { UIAbility, Want, AbilityConstant } from '@kit.AbilityKit'
import { window } from '@kit.ArkUI'

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    // 初始化全局服务
  }

  onDestroy(): void { /* 清理 */ }

  onForeground(): void { /* 刷新数据 */ }

  onBackground(): void { /* 保存状态 */ }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        console.error(`Failed to load: ${err.code}`)
      }
    })
  }
}
```

### Step 2.2 — 公共模块创建

```typescript
// entry/src/main/ets/common/DesignTokens.ets
export class DesignTokens {
  static readonly COLOR_PRIMARY = '#007AFF'
  static readonly COLOR_BG = '#F2F2F7'
  static readonly COLOR_CARD = '#FFFFFF'
  static readonly COLOR_TEXT_PRIMARY = '#1A1A1A'
  static readonly COLOR_TEXT_SECONDARY = '#8E8E93'
  static readonly COLOR_TEXT_TERTIARY = '#C7C7CC'
  static readonly COLOR_SUCCESS = '#34C759'
  static readonly COLOR_WARNING = '#FF9500'
  static readonly COLOR_ERROR = '#FF3B30'

  static readonly SPACING_XS = 4
  static readonly SPACING_SM = 8
  static readonly SPACING_MD = 16
  static readonly SPACING_LG = 24

  static readonly RADIUS_SM = 8
  static readonly RADIUS_MD = 12
  static readonly RADIUS_LG = 16
  static readonly RADIUS_FULL = 9999

  static readonly FONT_LARGE_TITLE = 28
  static readonly FONT_TITLE = 20
  static readonly FONT_BODY = 16
  static readonly FONT_CAPTION = 12
}

// entry/src/main/ets/common/Utils.ets
export class Utils {
  static formatDate(timestamp: number): string {
    const date = new Date(timestamp)
    return `${date.getFullYear()}-${(date.getMonth()+1).toString().padStart(2,'0')}-${date.getDate().toString().padStart(2,'0')}`
  }
}
```

### Step 2.3 — Index 页面更新

```typescript
// entry/src/main/ets/pages/Index.ets — 应用首页 (Stub)
import { DesignTokens } from '../common/DesignTokens'

@Entry
@Component
struct Index {
  @State message: string = 'Hello HarmonyOS'
  @State tasks: string[] = []

  aboutToAppear(): void {
    // Phase 4 将加载真实数据
  }

  build() {
    Column() {
      if (this.tasks.length === 0) {
        // 空状态
        Column({ space: DesignTokens.SPACING_MD }) {
          Image($r('sys.media.ohos_ic_public_notes'))
            .width(64).height(64)
            .fillColor(DesignTokens.COLOR_TEXT_TERTIARY)
          Text('暂无数据').fontSize(DesignTokens.FONT_BODY).fontColor(DesignTokens.COLOR_TEXT_SECONDARY)
          Text('点击下方按钮开始添加').fontSize(DesignTokens.FONT_CAPTION).fontColor(DesignTokens.COLOR_TEXT_TERTIARY)
        }
        .width('100%').layoutWeight(1)
        .justifyContent(FlexAlign.Center)
      } else {
        // Phase 5 实现列表 UI
        Text('待 Phase 5 实现').fontSize(16)
      }

      // 底部按钮
      Button('添加')
        .fontSize(17).fontWeight(FontWeight.Medium)
        .backgroundColor(DesignTokens.COLOR_PRIMARY)
        .borderRadius(DesignTokens.RADIUS_MD)
        .height(50).width('90%')
        .margin({ bottom: 32 })
    }
    .width('100%').height('100%')
    .backgroundColor(DesignTokens.COLOR_BG)
  }
}
```

```bash
# [SHELL] Phase 2 编译验证
source /tmp/sop_harmony.env 2>/dev/null
cd ${PROJECT_DIR}
./hvigorw assembleHap 2>&1 | tail -20
# [DEBUG] 如有错误: 读取错误行号 → Edit 目标 .ets 文件 → 修复 → 重编译
# [GIT]
git add -A && git commit -m "Phase 2: Stage architecture + DesignTokens + Index stub"
```

---

## Phase 3: 数据层开发 (2:30 - 4:30)

> **[GENERATE]** + **[WRITE]** + **[DEBUG]** Model + RelationalStore + Preferences

### Step 3.1 — 数据模型

```typescript
// entry/src/main/ets/model/TaskItem.ets
export enum Priority {
  LOW = 0,
  MEDIUM = 1,
  HIGH = 2,
  URGENT = 3
}

export class TaskItem {
  id: string = ''
  title: string = ''
  description: string = ''
  priority: Priority = Priority.MEDIUM
  isCompleted: boolean = false
  dueDate: number = 0       // timestamp
  createdAt: number = Date.now()
  completedAt: number = 0

  static create(title: string, priority: Priority = Priority.MEDIUM): TaskItem {
    const item = new TaskItem()
    item.id = `${Date.now()}_${Math.random().toString(36).slice(2, 9)}`
    item.title = title
    item.priority = priority
    item.createdAt = Date.now()
    return item
  }
}
```

### Step 3.2 — 数据库服务

```typescript
// entry/src/main/ets/service/DatabaseService.ets
import { relationalStore } from '@kit.ArkData'
import { TaskItem, Priority } from '../model/TaskItem'

const SQL_CREATE_TABLE = `
  CREATE TABLE IF NOT EXISTS tasks (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT DEFAULT '',
    priority INTEGER DEFAULT 1,
    is_completed INTEGER DEFAULT 0,
    due_date INTEGER DEFAULT 0,
    created_at INTEGER NOT NULL,
    completed_at INTEGER DEFAULT 0
  )
`

export class DatabaseService {
  private static store: relationalStore.RdbStore | null = null

  static async init(context: Context): Promise<void> {
    const config: relationalStore.StoreConfig = {
      name: 'app_database.db',
      securityLevel: relationalStore.SecurityLevel.S1
    }
    this.store = await relationalStore.getRdbStore(context, config)
    await this.store.executeSql(SQL_CREATE_TABLE)
  }

  static getStore(): relationalStore.RdbStore {
    if (!this.store) throw new Error('Database not initialized')
    return this.store
  }

  static async insertTask(task: TaskItem): Promise<number> {
    const vb: relationalStore.ValuesBucket = {
      'id': task.id, 'title': task.title,
      'description': task.description, 'priority': task.priority,
      'is_completed': task.isCompleted ? 1 : 0,
      'due_date': task.dueDate, 'created_at': task.createdAt,
      'completed_at': task.completedAt
    }
    return await this.store!.insert('tasks', vb)
  }

  static async queryAll(): Promise<TaskItem[]> {
    const predicates = new relationalStore.RdbPredicates('tasks')
    predicates.orderByDesc('created_at')
    const rs = await this.store!.query(predicates)
    const tasks: TaskItem[] = []
    while (rs.goToNextRow()) {
      tasks.push(DatabaseService.rowToTask(rs))
    }
    rs.close()
    return tasks
  }

  static async updateTask(task: TaskItem): Promise<number> {
    const vb: relationalStore.ValuesBucket = {
      'title': task.title, 'description': task.description,
      'priority': task.priority,
      'is_completed': task.isCompleted ? 1 : 0,
      'due_date': task.dueDate, 'completed_at': task.completedAt
    }
    const predicates = new relationalStore.RdbPredicates('tasks')
    predicates.equalTo('id', task.id)
    return await this.store!.update(vb, predicates)
  }

  static async deleteTask(id: string): Promise<number> {
    const predicates = new relationalStore.RdbPredicates('tasks')
    predicates.equalTo('id', id)
    return await this.store!.delete(predicates)
  }

  private static rowToTask(rs: relationalStore.ResultSet): TaskItem {
    const task = new TaskItem()
    task.id = rs.getString(rs.getColumnIndex('id'))
    task.title = rs.getString(rs.getColumnIndex('title'))
    task.description = rs.getString(rs.getColumnIndex('description'))
    task.priority = rs.getLong(rs.getColumnIndex('priority')) as Priority
    task.isCompleted = rs.getLong(rs.getColumnIndex('is_completed')) === 1
    task.dueDate = rs.getLong(rs.getColumnIndex('due_date'))
    task.createdAt = rs.getLong(rs.getColumnIndex('created_at'))
    task.completedAt = rs.getLong(rs.getColumnIndex('completed_at'))
    return task
  }
}
```

### Step 3.3 — Preferences (设置存储)

```typescript
// entry/src/main/ets/service/PreferenceService.ets
import { preferences } from '@kit.ArkData'

const PREF_NAME = 'app_settings'

export class PreferenceService {
  private static prefs: preferences.Preferences | null = null

  static async init(context: Context): Promise<void> {
    this.prefs = await preferences.getPreferences(context, PREF_NAME)
  }

  static async getString(key: string, defaultValue: string = ''): Promise<string> {
    if (!this.prefs) return defaultValue
    return await this.prefs.get(key, defaultValue) as string
  }

  static async setString(key: string, value: string): Promise<void> {
    if (!this.prefs) return
    await this.prefs.put(key, value)
    await this.prefs.flush()
  }

  static async getBoolean(key: string, defaultValue: boolean = false): Promise<boolean> {
    if (!this.prefs) return defaultValue
    return await this.prefs.get(key, defaultValue) as boolean
  }

  static async setBoolean(key: string, value: boolean): Promise<void> {
    if (!this.prefs) return
    await this.prefs.put(key, value)
    await this.prefs.flush()
  }
}
```

### Step 3.4 — 更新 EntryAbility (完整文件)

> **[READ]** 打开 `entry/src/main/ets/entryability/EntryAbility.ets` → **[EDIT]** 替换整个文件:

```typescript
// entry/src/main/ets/entryability/EntryAbility.ets — 完整版 (含数据库初始化)
import { UIAbility, Want, AbilityConstant } from '@kit.AbilityKit'
import { window } from '@kit.ArkUI'
import { DatabaseService } from '../service/DatabaseService'
import { PreferenceService } from '../service/PreferenceService'

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    // 初始化全局服务 (必须在 onCreate 中获取 context)
    DatabaseService.init(this.context)
    PreferenceService.init(this.context)
  }

  onDestroy(): void { /* 清理 */ }

  onForeground(): void { /* 刷新数据 */ }

  onBackground(): void { /* 保存状态 */ }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        console.error(`Failed to load content: ${err.code}`)
      }
    })
  }
}
```

```bash
# [SHELL] Phase 3 编译验证 (验证 Model + DatabaseService + PreferenceService 集成)
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
./hvigorw assembleHap 2>&1 | tail -20
# [DEBUG] 常见错误: import 路径不对 / Context 类型未声明 / 缺少 @kit.ArkData 依赖
# [GIT]
git add -A && git commit -m "Phase 3: Data layer - Model + RelationalStore + Preferences"
```

---

### Phase 3.5: 网络服务封装 (可选, 如 SPECS 需要后端)

> **[WRITE]** + **[GENERATE]** HTTP 客户端 + 华为账号登录 (如适用)

```typescript
// entry/src/main/ets/service/NetworkService.ets
import { http } from '@kit.NetworkKit'

export class NetworkError extends Error {
  code: number
  constructor(code: number, message: string) {
    super(message)
    this.code = code
  }
}

export class NetworkService {
  private static readonly BASE_URL = 'https://api.example.com/v1'
  private static readonly TIMEOUT = 30000

  static async request<T>(endpoint: string, options: http.HttpRequestOptions = {}): Promise<T> {
    const httpRequest = http.createHttp()
    const defaultOptions: http.HttpRequestOptions = {
      method: http.RequestMethod.GET,
      header: { 'Content-Type': 'application/json' },
      connectTimeout: this.TIMEOUT,
      readTimeout: this.TIMEOUT,
      ...options
    }
    try {
      const response = await httpRequest.request(`${this.BASE_URL}/${endpoint}`, defaultOptions)
      if (response.responseCode === 200) {
        return JSON.parse(response.result as string) as T
      }
      throw new NetworkError(response.responseCode, `HTTP ${response.responseCode}`)
    } finally {
      httpRequest.destroy()
    }
  }

  static async get<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint)
  }

  static async post<T>(endpoint: string, body: object): Promise<T> {
    return this.request<T>(endpoint, {
      method: http.RequestMethod.POST,
      extraData: JSON.stringify(body)
    })
  }
}
```

```bash
# [SHELL] 编译验证
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
./hvigorw assembleHap 2>&1 | tail -10
# [GIT]
git add -A && git commit -m "Phase 3.5: NetworkService (optional)"
```

### Phase 3.6: 华为账号登录 (可选, 如 SPECS 需要登录)

> **[WRITE]** + **[DIALOG]** 集成华为账号 Kit (Account Kit)

```typescript
// entry/src/main/ets/service/AuthService.ets
import { authentication } from '@kit.AccountKit'
import { BusinessError } from '@kit.BasicServicesKit'

export class AuthService {
  private static readonly HUAWEI_ACCOUNT_SCOPE = ['profile', 'openid']

  // 华为账号登录
  static async signIn(): Promise<{ success: boolean; openId?: string }> {
    try {
      const request: authentication.HuaweiIDProvider = {
        scopes: this.HUAWEI_ACCOUNT_SCOPE,
        state: Math.random().toString(36)
      }
      const result = await authentication.signIn(request)
      if (result && result.openId) {
        console.info(`Sign in success: ${result.openId}`)
        return { success: true, openId: result.openId }
      }
      return { success: false }
    } catch (err) {
      const error = err as BusinessError
      console.error(`Sign in failed: ${error.code} ${error.message}`)
      return { success: false }
    }
  }

  // 静默登录 (已授权用户)
  static async silentSignIn(): Promise<{ success: boolean; openId?: string }> {
    try {
      const request: authentication.HuaweiIDProvider = {
        scopes: this.HUAWEI_ACCOUNT_SCOPE,
        state: Math.random().toString(36)
      }
      const result = await authentication.silentSignIn(request)
      if (result && result.openId) {
        return { success: true, openId: result.openId }
      }
      return { success: false }
    } catch (err) {
      return { success: false }
    }
  }

  // 退出登录
  static async signOut(): Promise<void> {
    try {
      await authentication.signOut()
    } catch (err) {
      console.error(`Sign out failed: ${JSON.stringify(err)}`)
    }
  }
}
```

```json5
// module.json5 中添加 Account Kit 权限
"requestPermissions": [
  { "name": "ohos.permission.GET_BUNDLE_INFO" }
]
// 同时需要在 AGC → 我的应用 → 开发 → API 管理 中开通 Account Kit
```

> **[DIALOG]** 使用华为账号登录需要在 AGC 中开通 Account Kit 服务，配置 OAuth 回调。

```bash
# [GIT]
git add -A && git commit -m "Phase 3.6: AuthService - Huawei Account Kit (optional)"
```

---

## Phase 4: ViewModel 层 (4:30 - 6:00)

> **[GENERATE]** + **[WRITE]** + **[REVIEW]** 基于 Model 创建 ViewModel

```typescript
// entry/src/main/ets/viewmodel/TaskViewModel.ets
import { TaskItem, Priority } from '../model/TaskItem'
import { DatabaseService } from '../service/DatabaseService'

export enum FilterOption {
  ALL = 'all',
  ACTIVE = 'active',
  COMPLETED = 'completed',
  HIGH_PRIORITY = 'high'
}

export class TaskViewModel {
  tasks: TaskItem[] = []
  isLoading: boolean = false
  searchText: string = ''
  selectedFilter: FilterOption = FilterOption.ALL

  async loadTasks(): Promise<void> {
    this.isLoading = true
    try {
      this.tasks = await DatabaseService.queryAll()
    } catch (e) {
      console.error(`TaskVM load error: ${JSON.stringify(e)}`)
    } finally {
      this.isLoading = false
    }
  }

  get filteredTasks(): TaskItem[] {
    let result = this.tasks

    // 筛选
    switch (this.selectedFilter) {
      case FilterOption.ACTIVE:
        result = result.filter(t => !t.isCompleted)
        break
      case FilterOption.COMPLETED:
        result = result.filter(t => t.isCompleted)
        break
      case FilterOption.HIGH_PRIORITY:
        result = result.filter(t => t.priority >= Priority.HIGH)
        break
    }

    // 搜索
    if (this.searchText.trim()) {
      const keyword = this.searchText.toLowerCase()
      result = result.filter(t => t.title.toLowerCase().includes(keyword))
    }

    return result
  }

  async addTask(title: string, priority: Priority = Priority.MEDIUM): Promise<void> {
    const task = TaskItem.create(title, priority)
    await DatabaseService.insertTask(task)
    await this.loadTasks()
  }

  async toggleComplete(task: TaskItem): Promise<void> {
    task.isCompleted = !task.isCompleted
    task.completedAt = task.isCompleted ? Date.now() : 0
    await DatabaseService.updateTask(task)
    await this.loadTasks()
  }

  async updateTask(task: TaskItem): Promise<void> {
    await DatabaseService.updateTask(task)
    await this.loadTasks()
  }

  async deleteTask(id: string): Promise<void> {
    await DatabaseService.deleteTask(id)
    await this.loadTasks()
  }
}
```

```bash
# [SHELL] Phase 4 编译验证
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
./hvigorw assembleHap 2>&1 | tail -20
# [DEBUG] 常见错误: FilterOption 导入路径 / TaskItem 属性不存在
# [GIT]
git add -A && git commit -m "Phase 4: ViewModel - TaskViewModel with filter/search/CRUD"
```

### Phase 4.1: 依赖注入容器 (可选, ViewModel ≥ 3 时推荐)

> **[WRITE]** 轻量 DI 容器，便于测试时注入 Mock

```typescript
// entry/src/main/ets/common/AppContainer.ets
import { DatabaseService } from '../service/DatabaseService'
import { PreferenceService } from '../service/PreferenceService'
import { TaskViewModel } from '../viewmodel/TaskViewModel'

export class AppContainer {
  // ViewModel 工厂
  static makeTaskViewModel(): TaskViewModel {
    return new TaskViewModel()
  }

  // Service 初始化 (在 EntryAbility.onCreate 中调用)
  static async initServices(context: Context): Promise<void> {
    await DatabaseService.init(context)
    await PreferenceService.init(context)
  }
}
```

> Claude Code 更新 EntryAbility: 用 `AppContainer.initServices(this.context)` 替换直接调用。

```bash
# [GIT]
git add -A && git commit -m "Phase 4.1: DI container"
```

---

## Phase 5: UI 层开发 ← 按原型实现 (6:00 - 7:30)

> **[GENERATE]** + **[WRITE]** + **[REVIEW]** + **[DEBUG]** 1:1 还原原型

### Step 5.0 — 原型映射 & 清理

> **[READ]** + **[EDIT]** Phase 2 创建了 Stub Index 页面，Phase 5 用完整实现替换。
> 原型中的设计值必须与 DesignTokens 一一对应。

```
原型 → 生产代码映射:
  原型硬编码颜色 → DesignTokens.COLOR_XXX
  原型硬编码字号 → DesignTokens.FONT_XXX
  原型硬编码间距 → DesignTokens.SPACING_XXX
  原型硬编码圆角 → DesignTokens.RADIUS_XXX
  原型假数据 → TaskViewModel 真实数据
  原型 @Entry struct → @Component export struct (组件化)
```

**[READ]** 打开 `prototype/screens/HomePrototype.ets` → 确认设计细节 → **[EDIT]** 开始实现。

### Step 5.1 — 任务卡片组件

```typescript
// entry/src/main/ets/components/TaskCard.ets
import { TaskItem, Priority } from '../model/TaskItem'
import { DesignTokens } from '../common/DesignTokens'

@Component
export struct TaskCard {
  @Prop task: TaskItem
  onToggle?: () => void
  onTap?: () => void

  getPriorityColor(): string {
    switch (this.task.priority) {
      case Priority.URGENT: return DesignTokens.COLOR_ERROR
      case Priority.HIGH:   return DesignTokens.COLOR_WARNING
      case Priority.MEDIUM: return DesignTokens.COLOR_PRIMARY
      default:              return DesignTokens.COLOR_TEXT_TERTIARY
    }
  }

  getPriorityLabel(): string {
    switch (this.task.priority) {
      case Priority.URGENT: return '紧急'
      case Priority.HIGH:   return '高'
      case Priority.MEDIUM: return '中'
      default:              return '低'
    }
  }

  build() {
    Row({ space: DesignTokens.SPACING_MD }) {
      // 完成状态图标
      Image($r('sys.media.ohos_ic_public_ok'))
        .width(24).height(24)
        .fillColor(this.task.isCompleted ? DesignTokens.COLOR_SUCCESS : DesignTokens.COLOR_TEXT_TERTIARY)
        .onClick(() => { if (this.onToggle) this.onToggle() })

      // 内容
      Column({ space: DesignTokens.SPACING_XS }) {
        Text(this.task.title)
          .fontSize(DesignTokens.FONT_BODY)
          .fontColor(this.task.isCompleted ? DesignTokens.COLOR_TEXT_TERTIARY : DesignTokens.COLOR_TEXT_PRIMARY)
          .decoration({ type: this.task.isCompleted ? TextDecorationType.LineThrough : TextDecorationType.None })
          .maxLines(2)
          .textOverflow({ overflow: TextOverflow.Ellipsis })

        if (this.task.dueDate > 0 && !this.task.isCompleted) {
          Row({ space: 4 }) {
            Image($r('sys.media.ohos_ic_public_calendar'))
              .width(12).height(12)
              .fillColor(this.task.dueDate < Date.now() ? DesignTokens.COLOR_ERROR : DesignTokens.COLOR_WARNING)
            Text(this.formatDueDate())
              .fontSize(DesignTokens.FONT_CAPTION)
              .fontColor(this.task.dueDate < Date.now() ? DesignTokens.COLOR_ERROR : DesignTokens.COLOR_WARNING)
          }
        }
      }
      .alignItems(HorizontalAlign.Start)
      .layoutWeight(1)

      // 优先级标签
      Text(this.getPriorityLabel())
        .fontSize(11)
        .fontColor(this.getPriorityColor())
        .padding({ left: 8, right: 8, top: 2, bottom: 2 })
        .backgroundColor(this.getPriorityColor() + '10')
        .borderRadius(DesignTokens.RADIUS_SM)
    }
    .width('100%')
    .padding(DesignTokens.SPACING_MD)
    .backgroundColor(DesignTokens.COLOR_CARD)
    .borderRadius(DesignTokens.RADIUS_MD)
    .onClick(() => { if (this.onTap) this.onTap() })
  }

  formatDueDate(): string {
    const d = new Date(this.task.dueDate)
    return `${d.getMonth()+1}月${d.getDate()}日`
  }
}
```

### Step 5.2 — 完整首页

```typescript
// entry/src/main/ets/pages/Index.ets — 完整实现
import { TaskViewModel, FilterOption } from '../viewmodel/TaskViewModel'
import { TaskItem, Priority } from '../model/TaskItem'
import { TaskCard } from '../components/TaskCard'
import { DesignTokens } from '../common/DesignTokens'
import { router } from '@kit.ArkUI'

@Entry
@Component
struct Index {
  @State viewModel: TaskViewModel = new TaskViewModel()
  @State refreshFlag: number = 0

  private filters: Array<{ label: string, value: FilterOption }> = [
    { label: '全部', value: FilterOption.ALL },
    { label: '进行中', value: FilterOption.ACTIVE },
    { label: '已完成', value: FilterOption.COMPLETED },
    { label: '高优先级', value: FilterOption.HIGH_PRIORITY }
  ]

  aboutToAppear(): void {
    this.viewModel.loadTasks().then(() => { this.refreshFlag++ })
  }

  build() {
    Column() {
      // 顶部标题
      Text('我的任务')
        .fontSize(DesignTokens.FONT_LARGE_TITLE)
        .fontWeight(FontWeight.Bold)
        .width('100%')
        .padding({ left: DesignTokens.SPACING_MD, top: DesignTokens.SPACING_MD })

      // 筛选标签
      Scroll() {
        Row({ space: DesignTokens.SPACING_SM }) {
          ForEach(this.filters, (item: { label: string, value: FilterOption }) => {
            Text(item.label)
              .fontSize(14)
              .fontWeight(this.viewModel.selectedFilter === item.value ? FontWeight.Bold : FontWeight.Normal)
              .fontColor(this.viewModel.selectedFilter === item.value ? Color.White : '#333')
              .padding({ left: 16, right: 16, top: 6, bottom: 6 })
              .backgroundColor(this.viewModel.selectedFilter === item.value
                ? DesignTokens.COLOR_PRIMARY : '#E8E8ED')
              .borderRadius(DesignTokens.RADIUS_FULL)
              .onClick(() => {
                this.viewModel.selectedFilter = item.value
                this.refreshFlag++
              })
          })
        }
        .padding(DesignTokens.SPACING_MD)
      }
      .scrollable(ScrollDirection.Horizontal)
      .scrollBar(BarState.Off)

      // 任务列表
      if (this.viewModel.isLoading) {
        Column() {
          LoadingProgress().color(DesignTokens.COLOR_PRIMARY)
        }
        .width('100%').layoutWeight(1)
        .justifyContent(FlexAlign.Center)
      } else if (this.viewModel.filteredTasks.length === 0) {
        Column({ space: DesignTokens.SPACING_MD }) {
          Image($r('sys.media.ohos_ic_public_notes'))
            .width(64).height(64).fillColor(DesignTokens.COLOR_TEXT_TERTIARY)
          Text('暂无任务').fontSize(DesignTokens.FONT_BODY).fontColor(DesignTokens.COLOR_TEXT_SECONDARY)
        }
        .width('100%').layoutWeight(1).justifyContent(FlexAlign.Center)
      } else {
        List({ space: DesignTokens.SPACING_SM }) {
          ForEach(this.viewModel.filteredTasks,
            (task: TaskItem) => {
              ListItem() {
                TaskCard({
                  task: task,
                  onToggle: () => {
                    this.viewModel.toggleComplete(task).then(() => { this.refreshFlag++ })
                  }
                })
              }
            },
            (task: TaskItem) => task.id
          )
        }
        .layoutWeight(1)
        .padding({ left: DesignTokens.SPACING_MD, right: DesignTokens.SPACING_MD })
        .scrollBar(BarState.Off)
      }

      // 添加按钮
      Button('添加任务')
        .fontSize(17).fontWeight(FontWeight.Medium)
        .backgroundColor(DesignTokens.COLOR_PRIMARY)
        .borderRadius(DesignTokens.RADIUS_MD)
        .height(50).width('90%')
        .margin({ bottom: 32 })
        .onClick(() => {
          this.viewModel.addTask('新任务 ' + new Date().toLocaleTimeString())
            .then(() => { this.refreshFlag++ })
        })
    }
    .width('100%').height('100%')
    .backgroundColor(DesignTokens.COLOR_BG)
  }
}
```

### Step 5.3 — 导航模式 (鸿蒙 HIG)

```typescript
// ✅ Tab 导航 — 鸿蒙标准: 3-5 个标签, 底部定位
@Entry
@Component
struct MainTabs {
  @State currentIndex: number = 0
  private tabsController: TabsController = new TabsController()

  build() {
    Tabs({ barPosition: BarPosition.End, controller: this.tabsController }) {
      TabContent() { HomePage() }
        .tabBar(this.tabBuilder('首页', 0))
      TabContent() { DiscoverPage() }
        .tabBar(this.tabBuilder('发现', 1))
      TabContent() { ProfilePage() }
        .tabBar(this.tabBuilder('我的', 2))
    }.onChange((index: number) => { this.currentIndex = index })
  }

  @Builder tabBuilder(text: string, index: number) {
    Column() {
      Text(index === 0 ? '📋' : index === 1 ? '🔍' : '👤').fontSize(22)
      Text(text).fontSize(10)
        .fontColor(this.currentIndex === index ? DesignTokens.COLOR_PRIMARY : DesignTokens.COLOR_TEXT_SECONDARY)
    }
  }
}

// ✅ router 跳转 — pushUrl 带参数, router.back() 返回
// ✅ Navigation + NavPathStack — 大项目推荐 (见 HarmonyOS_Development_Guide.md 7.1)
```

### Step 5.4 — 模态 & 反馈模式

```typescript
// ✅ AlertDialog — 关键确认 (删除/不可逆操作前)
AlertDialog.show({
  title: '删除任务',
  message: '此操作不可撤销，确定删除？',
  primaryButton: { value: '取消', action: () => {} },
  secondaryButton: {
    value: '删除',
    fontColor: DesignTokens.COLOR_ERROR,
    action: () => { this.viewModel.deleteTask(task.id) }
  }
})

// ✅ CustomDialog — 自定义弹窗 (复杂表单/详情)
@CustomDialog
struct AddTaskDialog {
  controller: CustomDialogController
  @State inputText: string = ''

  build() {
    Column() {
      Text('新建任务').fontSize(20).fontWeight(FontWeight.Bold)
      TextInput({ placeholder: '任务标题', text: this.inputText })
        .onChange((v: string) => { this.inputText = v })
      Row({ space: 12 }) {
        Button('取消').onClick(() => { this.controller.close() })
        Button('确定').onClick(() => {
          // 保存任务
          this.controller.close()
        }).backgroundColor(DesignTokens.COLOR_PRIMARY)
      }
    }.padding(24)
  }
}

// ✅ Toast/提示 — 轻量反馈 (非阻断性)
import { promptAction } from '@kit.ArkUI'
promptAction.showToast({ message: '任务已添加', duration: 2000 })

// ✅ 鸿蒙反馈分级:
// 操作成功 → Toast (2秒自动消失)
// 关键确认 → AlertDialog (需用户明确操作)
// 多选项 → ActionSheet (底部弹出)
// 错误提示 → 内嵌红色文本 + Toast
// 加载中 → LoadingProgress
```

### Step 5.5 — 引导页 & 空状态

```typescript
// ✅ 引导页 (首次启动时显示, 可跳过)
@Entry
@Component
struct OnboardingPage {
  @StorageProp('has_onboarded') hasOnboarded: boolean = false
  @State currentPage: number = 0

  build() {
    Column() {
      Swiper() {
        // 引导页 1
        Column({ space: 24 }) {
          Image($r('sys.media.ohos_ic_public_notes')).width(80).height(80)
          Text('欢迎使用').fontSize(28).fontWeight(FontWeight.Bold)
          Text('高效管理你的日常任务').fontSize(16).fontColor(DesignTokens.COLOR_TEXT_SECONDARY)
        }
        // 引导页 2
        Column({ space: 24 }) {
          Image($r('sys.media.ohos_ic_public_calendar')).width(80).height(80)
          Text('智能提醒').fontSize(28).fontWeight(FontWeight.Bold)
          Text('不再错过任何重要事项').fontSize(16).fontColor(DesignTokens.COLOR_TEXT_SECONDARY)
        }
      }
      .indicator(true).loop(false).layoutWeight(1)

      Row({ space: 16 }) {
        Button('跳过').fontColor(DesignTokens.COLOR_TEXT_SECONDARY).backgroundColor(Color.Transparent)
          .onClick(() => { this.completeOnboarding() })
        Button('开始使用').backgroundColor(DesignTokens.COLOR_PRIMARY).borderRadius(24).height(50)
          .onClick(() => { this.completeOnboarding() })
      }.padding(32).width('100%')
    }
  }

  completeOnboarding(): void {
    AppStorage.setOrCreate('has_onboarded', true)
    // 跳转到主页 (需要 EntryAbility 判断)
  }
}

// ✅ 空状态 (列表为空时显示)
// 已在 Step 5.2 Index.ets 中包含: LoadingProgress / 暂无数据 / 有数据 三态切换

// ✅ ContentUnavailableView (鸿蒙等价: 自定义空状态组件)
@Component
struct EmptyStateView {
  @Prop icon: Resource = $r('sys.media.ohos_ic_public_notes')
  @Prop title: string = '暂无内容'
  @Prop message: string = ''
  action?: () => void
  actionTitle?: string

  build() {
    Column({ space: DesignTokens.SPACING_MD }) {
      Image(this.icon).width(64).height(64).fillColor(DesignTokens.COLOR_TEXT_TERTIARY)
      Text(this.title).fontSize(DesignTokens.FONT_BODY).fontColor(DesignTokens.COLOR_TEXT_SECONDARY)
      if (this.message) {
        Text(this.message).fontSize(DesignTokens.FONT_CAPTION).fontColor(DesignTokens.COLOR_TEXT_TERTIARY)
      }
    }
  }
}
```

### Step 5.6 — HIG 适配 (鸿蒙设计规范)

```
✅ 所有文本使用 fp 单位 (自适应缩放)
✅ 所有间距使用 vp 单位
✅ 所有颜色使用 DesignTokens 常量
✅ 列表使用 LazyForEach (数据 > 100 时)
✅ 触摸目标 ≥ 48vp (鸿蒙标准)
✅ 深色模式自动适配 (系统处理)
✅ 支持横竖屏 (module.json5: "orientation": "auto_rotation")
□ Tab 导航 3-5 个标签, 底部定位
□ 模态弹窗有明确关闭方式
□ 引导页可选/可跳过
```

```bash
# [SHELL] Phase 5 编译验证
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
./hvigorw assembleHap 2>&1 | tail -20
# [DEBUG] 常见错误: TaskCard @Prop 类型不匹配 / @Entry 页面缺失
# [DIALOG] 在 DevEco Studio Previewer 中打开 Index.ets → 视觉确认与原型一致
# 用户回复 "视觉确认通过" 后继续
```
```bash
# [GIT]
git add -A && git commit -m "Phase 5: UI layer - TaskCard + Index + HIG compliance"
```

---

## Phase 6: 自测 & Day 1 收尾 (7:30 - 8:00)

> **[SHELL]** + **[VALIDATE]** + **[WRITE]** + **[GIT]**

### Step 6.1 — 编译 & 代码统计

```bash
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
./hvigorw assembleHap 2>&1 | tail -10

echo "=== ArkTS 代码统计 ==="
find entry/src/main/ets -name "*.ets" | wc -l | xargs echo "源文件数:"
find entry/src/main/ets -name "*.ets" -exec wc -l {} \; | awk '{sum+=$1} END {print "总行数: " sum}'
```

### Step 6.2 — 单元测试 (Hypium)

> **[WRITE]** `entry/src/test/TaskViewModel.test.ets` — 4 条核心用例

```typescript
import { describe, it, expect } from '@ohos/hypium'
import { TaskViewModel, FilterOption } from '../../main/ets/viewmodel/TaskViewModel'
import { TaskItem, Priority } from '../../main/ets/model/TaskItem'

export default function TaskViewModelTest() {
  describe('TaskViewModel', () => {
    it('filter_should_return_all_when_filter_is_all', () => {
      const vm = new TaskViewModel()
      vm.tasks = [
        { id:'1', title:'A', isCompleted:false, priority:0, description:'', dueDate:0, createdAt:0, completedAt:0 },
        { id:'2', title:'B', isCompleted:true, priority:0, description:'', dueDate:0, createdAt:0, completedAt:0 }
      ]
      vm.selectedFilter = FilterOption.ALL
      expect(vm.filteredTasks.length).assertEqual(2)
    })

    it('filter_should_return_active_only', () => {
      const vm = new TaskViewModel()
      vm.tasks = [
        { id:'1', title:'A', isCompleted:false, priority:0, description:'', dueDate:0, createdAt:0, completedAt:0 },
        { id:'2', title:'B', isCompleted:true, priority:0, description:'', dueDate:0, createdAt:0, completedAt:0 }
      ]
      vm.selectedFilter = FilterOption.ACTIVE
      expect(vm.filteredTasks.length).assertEqual(1)
    })

    it('search_should_filter_by_title', () => {
      const vm = new TaskViewModel()
      vm.tasks = [
        { id:'1', title:'Hello', isCompleted:false, priority:0, description:'', dueDate:0, createdAt:0, completedAt:0 },
        { id:'2', title:'World', isCompleted:false, priority:0, description:'', dueDate:0, createdAt:0, completedAt:0 }
      ]
      vm.searchText = 'Hello'
      expect(vm.filteredTasks.length).assertEqual(1)
    })

    it('empty_tasks_should_return_empty_filtered', () => {
      const vm = new TaskViewModel()
      vm.tasks = []
      expect(vm.filteredTasks.length).assertEqual(0)
    })
  })
}
```

> **[DIALOG]** 在 DevEco Studio: 右键 `entry/src/test` → Run 'Tests' → 4 条全部通过

### Step 6.3 — Day 1 报告 & 提交

```bash
cat > DAY1_REPORT.md << 'EOF'
# Day 1 完成报告
- Stage 模型: ✅ EntryAbility + module.json5
- 数据层: ✅ RelationalStore(CRUD) + Preferences + NetworkService
- 认证: ✅ AuthService(华为账号, 可选)
- ViewModel: ✅ TaskViewModel(筛选/搜索/CRUD)
- DI 容器: ✅ AppContainer
- UI 层: ✅ TaskCard + Index + 导航 + 模态 + 引导 + 空状态
- HIG: ✅ fp/vp, 48vp触摸, 深色模式, 横竖屏
- 编译: ✅ ./hvigorw assembleHap 通过
- 测试: ✅ 4条Hypium用例通过
EOF

git add -A && git commit -m "Day 1 complete: Stage + MVVM + RelationalStore + full HIG UI patterns"
```

---

# 🗓️ DAY 2 — 从 MVP 到可发布

### Day 2 启动校验 (0:00-0:05)

```bash
# [SHELL] + [DEBUG] 验证 Day 1 代码完好
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
git status --short
echo "=== Day 2 启动编译验证 ==="
./hvigorw assembleHap 2>&1 | tail -20
# [DEBUG] 如有编译失败 → 修复再继续; 如通过 → 进入 Day 2
echo "✅ Day 2 启动校验通过"
```

---

## Phase 7: 详情页 & 路由集成 (Day 2, 0:05-1:00)

> **[GENERATE]** + **[WRITE]** + **[DEBUG]** 创建编辑详情页, 集成 Navigation 路由

### Step 7.1 — DetailPage (完整实现)

**[WRITE]** `entry/src/main/ets/pages/DetailPage.ets`:

```typescript
import { router } from '@kit.ArkUI'
import { TaskItem, Priority } from '../model/TaskItem'
import { TaskViewModel } from '../viewmodel/TaskViewModel'
import { DesignTokens } from '../common/DesignTokens'

@Entry
@Component
struct DetailPage {
  @State task: TaskItem = new TaskItem()
  @State editTitle: string = ''
  @State editDescription: string = ''
  @State editPriority: number = Priority.MEDIUM
  @State editDueDate: string = ''
  private viewModel: TaskViewModel = new TaskViewModel()
  private isNewTask: boolean = true

  aboutToAppear(): void {
    const params = router.getParams() as Record<string, Object>
    if (params && params['taskId']) {
      // 编辑已有任务: 从 ViewModel 查找
      this.viewModel.loadTasks().then(() => {
        const found = this.viewModel.tasks.find(t => t.id === params['taskId'])
        if (found) {
          this.task = found
          this.editTitle = found.title
          this.editDescription = found.description
          this.editPriority = found.priority
          this.editDueDate = found.dueDate > 0 ? new Date(found.dueDate).toISOString().slice(0, 10) : ''
          this.isNewTask = false
        }
      })
    }
  }

  build() {
    Column() {
      // 标题栏
      Row() {
        Button('取消').fontSize(16).backgroundColor(Color.Transparent)
          .onClick(() => { router.back() })
        Text(this.isNewTask ? '新建任务' : '编辑任务')
          .fontSize(18).fontWeight(FontWeight.Bold).layoutWeight(1).textAlign(TextAlign.Center)
        Button('保存').fontSize(16).fontColor(DesignTokens.COLOR_PRIMARY)
          .backgroundColor(Color.Transparent)
          .onClick(() => { this.saveTask() })
      }
      .width('100%').padding(16)

      // 表单
      Column({ space: DesignTokens.SPACING_LG }) {
        TextInput({ placeholder: '任务标题', text: this.editTitle })
          .fontSize(16).width('100%')
          .onChange((v: string) => { this.editTitle = v })

        TextArea({ placeholder: '详细描述 (可选)', text: this.editDescription })
          .fontSize(14).width('100%').height(100)
          .onChange((v: string) => { this.editDescription = v })

        Row({ space: DesignTokens.SPACING_MD }) {
          Text('优先级').fontSize(16)
          Blank()
          ForEach(['低', '中', '高', '紧急'], (label: string, index: number) => {
            Text(label)
              .fontSize(14).fontColor(this.editPriority === index ? Color.White : DesignTokens.COLOR_TEXT_PRIMARY)
              .padding({ left: 12, right: 12, top: 4, bottom: 4 })
              .backgroundColor(this.editPriority === index ? DesignTokens.COLOR_PRIMARY : DesignTokens.COLOR_BG)
              .borderRadius(DesignTokens.RADIUS_SM)
              .onClick(() => { this.editPriority = index })
          })
        }.width('100%')

        TextInput({ placeholder: '截止日期 (YYYY-MM-DD)', text: this.editDueDate })
          .fontSize(16).width('100%')
          .onChange((v: string) => { this.editDueDate = v })
      }
      .padding(16).layoutWeight(1)
    }
    .width('100%').height('100%')
    .backgroundColor(DesignTokens.COLOR_BG)
  }

  saveTask(): void {
    if (!this.editTitle.trim()) return  // 标题必填
    this.task.title = this.editTitle
    this.task.description = this.editDescription
    this.task.priority = this.editPriority
    if (this.editDueDate) {
      this.task.dueDate = new Date(this.editDueDate).getTime()
    }
    if (this.isNewTask) {
      this.viewModel.addTask(this.task.title, this.task.priority)
    } else {
      this.viewModel.updateTask(this.task)
    }
    router.back()
  }
}
```

### Step 7.2 — 注册路由 & 首页跳转 (精确编辑)

**[READ]** → **[EDIT]** 打开 `entry/src/main/resources/base/profile/main_pages.json`，替换整个文件内容为:

```json
{
  "src": [
    "pages/Index",
    "pages/DetailPage",
    "pages/SettingsPage"
  ]
}
```

**[READ]** 打开 `entry/src/main/ets/pages/Index.ets` → **[EDIT]**: 精确替换以下两处:

**替换 1**: 找到 TaskCard 的使用位置 (搜索 `TaskCard({`)，在 `onTap` 属性处，将原来的空白回调替换为:

```typescript
// 原代码: TaskCard({ task: task, onToggle: () => {...} })
// 改为:
TaskCard({
  task: task,
  onToggle: () => {
    this.viewModel.toggleComplete(task).then(() => { this.refreshFlag++ })
  },
  onTap: () => {
    router.pushUrl({ url: 'pages/DetailPage', params: { taskId: task.id } })
  }
})
```

**替换 2**: 找到底部的 "添加任务" Button (搜索 `添加任务`)，将 `onClick` 回调改为:

```typescript
// 原代码:
Button('添加任务')
  // ... 样式 ...
  .onClick(() => {
    this.viewModel.addTask('新任务 ' + new Date().toLocaleTimeString())
      .then(() => { this.refreshFlag++ })
  })

// 改为:
Button('添加任务')
  // ... 样式保持不变 ...
  .onClick(() => {
    router.pushUrl({ url: 'pages/DetailPage' })
  })
```

```bash
# [SHELL] Phase 7 编译验证
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
./hvigorw assembleHap 2>&1 | tail -20
# [GIT]
git add -A && git commit -m "Phase 7: DetailPage + router integration"
```

---

## Phase 8: 设置页 & 搜索 & 手势 (Day 2, 1:00-2:00)

> **[GENERATE]** + **[WRITE]** 设置页 + 搜索功能 + ContextMenu

**[WRITE]** `entry/src/main/ets/pages/SettingsPage.ets`:

```typescript
import { DesignTokens } from '../common/DesignTokens'
import { PreferenceService } from '../service/PreferenceService'
import { router } from '@kit.ArkUI'

@Entry
@Component
struct SettingsPage {
  @State hapticEnabled: boolean = true
  @State darkModeEnabled: boolean = false

  aboutToAppear(): void {
    PreferenceService.getBoolean('haptic_enabled', true).then(v => { this.hapticEnabled = v })
  }

  build() {
    Column() {
      Row() {
        Button('← 返回').fontSize(16).backgroundColor(Color.Transparent)
          .onClick(() => { router.back() })
        Text('设置').fontSize(20).fontWeight(FontWeight.Bold).layoutWeight(1).textAlign(TextAlign.Center)
        Blank().width(60)
      }.width('100%').padding(16)

      List() {
        ListItem() {
          Row() {
            Text('触觉反馈').fontSize(16)
            Blank()
            Toggle({ type: ToggleType.Switch, isOn: this.hapticEnabled })
              .onChange((v: boolean) => {
                this.hapticEnabled = v
                PreferenceService.setBoolean('haptic_enabled', v)
              })
          }.width('100%').padding(16).backgroundColor(Color.White).borderRadius(12)
        }.margin({ bottom: 8 })

        ListItem() {
          Row() {
            Text('关于').fontSize(16).layoutWeight(1)
            Text('v1.0.0').fontSize(14).fontColor(DesignTokens.COLOR_TEXT_SECONDARY)
            Image($r('sys.media.ohos_ic_public_arrow_right')).width(16).height(16)
              .fillColor(DesignTokens.COLOR_TEXT_TERTIARY)
          }.width('100%').padding(16).backgroundColor(Color.White).borderRadius(12)
        }
      }.layoutWeight(1).padding(16)
    }.width('100%').height('100%').backgroundColor(DesignTokens.COLOR_BG)
  }
}
```

**[READ]** → **[EDIT]** 打开 `entry/src/main/ets/pages/Index.ets`，做两处编辑:

**编辑 1**: 在 `build()` 方法中，在筛选标签 `Scroll()` 之前插入搜索栏:

```typescript
// 在 Column() 内, Text('我的任务') 之后, Scroll() 之前, 插入:
Search({ placeholder: '搜索任务', value: this.viewModel.searchText })
  .onChange((value: string) => {
    this.viewModel.searchText = value
    this.refreshFlag++
  })
  .width('90%')
  .margin({ top: 8, bottom: 8 })
```

**编辑 2**: 在 `ListItem()` 内的 `TaskCard` 组件上添加长按删除手势。找到 `TaskCard({` 所在处，在闭合 `})` 后添加:

```typescript
// 在 TaskCard 的 ListItem 上添加:
ListItem() {
  TaskCard({
    task: task,
    onToggle: () => { this.viewModel.toggleComplete(task).then(() => { this.refreshFlag++ }) },
    onTap: () => { router.pushUrl({ url: 'pages/DetailPage', params: { taskId: task.id } }) }
  })
}
.gesture(
  LongPressGesture({ repeat: false, duration: 500 })
    .onAction(() => {
      this.viewModel.deleteTask(task.id).then(() => { this.refreshFlag++ })
    })
)
// ⚠️ 注意: .gesture 添加在 ListItem() 上, 不在 TaskCard 上
```

```bash
# [SHELL] Phase 8 编译验证
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
./hvigorw assembleHap 2>&1 | tail -20
# [GIT]
git add -A && git commit -m "Phase 8: Settings + search + gesture"
```

### Phase 8.1: 无障碍 & 鸿蒙设计规范验证

> **[VALIDATE]** + **[REVIEW]** 对照鸿蒙设计规范逐项检查

```
✅ 文本 & 字体
  □ 所有字号使用 fp 单位 (自适应系统字体缩放)
  □ 最小字号 ≥ 12fp (鸿蒙无障碍标准)
  □ 字体加粗使用 FontWeight 枚举

✅ 颜色 & 对比度
  □ 文字与背景对比度 ≥ 4.5:1 (小文本) / 3:1 (大文本)
  □ 不依赖颜色作为唯一的信息传达方式
  □ 深色模式适配 (系统自动)

✅ 触摸 & 交互
  □ 所有可点击元素 ≥ 48vp × 48vp (鸿蒙标准)
  □ 重要操作有确认机制 (如删除前弹窗)
  □ 手势不冲突 (长按/滑动/点击)

✅ 内容 & 状态
  □ 有空状态提示 (列表为空时显示引导)
  □ 有加载状态 (LoadingProgress)
  □ 有错误状态提示
  □ 文字支持多语言 (使用 $r() 引用资源)

✅ 鸿蒙特性
  □ module.json5 声明 "orientation": "auto_rotation"
  □ 支持手机 + 平板 + 2in1
  □ app.json5 声明 deviceTypes: ["phone", "tablet"]
```

---

## Phase 9: AppGallery 素材 & 元数据 (Day 2, 2:00-3:30)

> **[SHELL]** + **[WRITE]** + **[DIALOG]**

### Step 9.1 — AGC App 创建

```
[DIALOG] 指导用户创建 AGC 应用:

1. 打开 https://developer.huawei.com → AppGallery Connect
2. 我的项目 → 添加项目 → 填写项目名称
3. 项目内 → 添加应用 → 选择平台 (HarmonyOS)
4. 填写应用包名: ${BUNDLE_NAME}
5. 创建完成后回复 'AGC 应用已创建'
```

### Step 9.2 — 图标 & 截图

```bash
# [SHELL] 准备图标 (需要用户提供 1024×1024 源图)
# 鸿蒙图标规范: 512×512 px, PNG, 无透明背景
# 放置位置: AppScope/resources/base/media/app_icon.png

# [SHELL] 截图 — 使用 hdc 从真机截取, 或用 Previewer 截图
# 截图尺寸要求: ≥ 1080×1920 (竖屏)
# 至少 3 张: 首页、功能页、设置页
```

### Step 9.3 — 元数据自动生成

> **[GENERATE]** + **[WRITE]** Claude Code 从 SPECS.md + DESIGN_SPECS.md 自动提取并生成 AGC 元数据

```bash
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
mkdir -p agc_metadata

# [WRITE] 生成应用描述 (从 SPECS.md 核心功能提取)
cat > agc_metadata/app_description.txt << DESC
【应用简介】
${DISPLAY_NAME} 是一款专注于 [核心功能描述] 的鸿蒙原生应用。

【核心功能】
$(grep -A5 "核心功能" SPECS.md 2>/dev/null | sed 's/^/- /' || echo "- [请补充功能描述]")

【适用场景】
$(grep -A3 "适用场景\|目标用户" SPECS.md 2>/dev/null | sed 's/^/- /' || echo "- [请补充适用场景]")

【特点】
- 鸿蒙原生 ArkUI 设计，流畅体验
- 支持手机、平板、2in1 多设备
- 本地数据安全存储
DESC
echo "✅ 应用描述已生成: agc_metadata/app_description.txt"

# [WRITE] 生成关键词 (从 SPECS.md + DESIGN_SPECS.md 提取)
cat > agc_metadata/keywords.txt << KW
$(grep "功能\|Feature" SPECS.md 2>/dev/null | head -5 | sed 's/.*: *//' | tr '\n' ',')
鸿蒙,ArkUI,效率,工具
KW
echo "✅ 关键词已生成"

# [WRITE] 生成隐私政策模板
cat > agc_metadata/privacy_policy.md << 'PRIVACY'
# ${DISPLAY_NAME} 隐私政策

## 信息收集
本应用仅收集必要的用户数据以提供核心功能。

## 数据存储
所有数据存储在用户设备本地，不上传至服务器。

## 权限说明
$(grep -A3 "permission" entry/src/main/module.json5 2>/dev/null | sed 's/.*name.*://' || echo "- 仅申请功能必需权限")

## 联系方式
如有隐私相关问题，请联系: [用户邮箱]
PRIVACY
echo "✅ 隐私政策已生成: agc_metadata/privacy_policy.md"
```

```bash
# [GIT]
git add -A && git commit -m "Phase 9: AppGallery assets & auto-generated metadata"
```

---

## Phase 10: 签名配置 & 发布构建 (Day 2, 3:30-4:30)

> **[SHELL]** + **[DIALOG]**

### Step 10.1 — 签名证书获取

```
[DIALOG] 指导用户获取签名证书:

1. 打开 DevEco Studio → Build → Generate Key and CSR
2. 填写: Alias=release, 密码, 组织信息
3. 生成 .p12 密钥库 + .csr 证书请求文件
4. 登录 AGC → 我的项目 → 签名 → 上传 .csr → 下载 .p7b 证书链
5. 文件清单:
   - release.p12 (密钥库, 自己保管)
   - release.p7b (证书链, AGC 签发)
   - release.cer (证书, 可选)
```

### Step 10.2 — 配置 build-profile.json5

```json5
// [EDIT] entry/build-profile.json5 — 添加签名配置
{
  "apiType": "stageMode",
  "buildOption": {},
  "targets": [{
    "name": "default",
    "runtimeOS": "HarmonyOS"
  }],
  "signingConfigs": [{
    "name": "release",
    "type": "HarmonyOS",
    "material": {
      "storeFile": "release.p12",
      "storePassword": "YOUR_PASSWORD",
      "keyAlias": "release",
      "keyPassword": "YOUR_KEY_PASSWORD",
      "signAlg": "SHA256withECDSA",
      "profile": "release.p7b",
      "certpath": "release.cer"
    }
  }]
}
```

### Step 10.3 — 华为应用内支付 (IAP, 可选)

> 仅当 SPECS.md 选择了盈利模式 (华为 IAP) 时执行

```typescript
// entry/src/main/ets/service/IAPService.ets
import { iap } from '@kit.IAPKit'
import { BusinessError } from '@kit.BasicServicesKit'

export class IAPService {
  private static readonly PRODUCT_IDS = [
    'premium_monthly',
    'premium_yearly',
    'remove_ads'
  ]

  // 查询可购买商品
  static async queryProducts(): Promise<iap.ProductInfo[]> {
    try {
      const result = await iap.queryProductInfo({ productIds: this.PRODUCT_IDS })
      return result.productInfos
    } catch (err) {
      const error = err as BusinessError
      console.error(`IAP query failed: ${error.code} ${error.message}`)
      return []
    }
  }

  // 购买商品
  static async purchase(productId: string): Promise<boolean> {
    try {
      const result = await iap.purchaseProduct({ productId: productId })
      // 验证购买结果
      if (result.purchaseResultCode === 0) {
        console.info(`Purchase success: ${productId}`)
        return true
      }
      return false
    } catch (err) {
      console.error(`Purchase failed: ${JSON.stringify(err)}`)
      return false
    }
  }

  // 恢复购买
  static async restorePurchases(): Promise<void> {
    // 查询已购非消耗型商品
    try {
      const result = await iap.queryPurchasedProductInfo({
        productType: iap.ProductType.NONCONSUMABLE
      })
      // 恢复用户权益
    } catch (err) {
      console.error(`Restore failed: ${JSON.stringify(err)}`)
    }
  }
}
```

```json5
// 在 module.json5 声明 IAP 权限
"requestPermissions": [
  { "name": "ohos.permission.INTERNET" },
  { "name": "ohos.permission.IN_APP_PURCHASE" }
]
```

> **[DIALOG]** IAP 商品需先在 AGC → 增长 → 应用内支付 中配置商品 ID

### Step 10.4 — 发布构建

```bash
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
# 清理后构建发布包
./hvigorw clean
./hvigorw assembleApp 2>&1 | tail -20

# 产物路径
echo "=== 构建产物 ==="
find build -name "*.app" -o -name "*.hap" 2>/dev/null
# [DEBUG] 确认 .app 文件存在且 > 1MB
```

```bash
# [GIT]
git tag v1.0.0
git add -A && git commit -m "Phase 10: Release build v1.0.0 signed"
```

---

## Phase 11: AGC 云测试 (Day 2, 4:30-5:30)

> **[DIALOG]** 云测试必须通过才能上架

```
[DIALOG] AGC 云测试流程:

1. 打开 AGC → 我的应用 → 版本 → 上传 .app 包
2. 上传后 → 云测试 → 新建测试
3. 选择测试类型:
   □ 兼容性测试 (≥ 5 款设备)
   □ 稳定性测试 (Monkey ≥ 30min)
   □ 性能测试 (冷启动 < 2s)
   □ 安全测试
   □ 权限测试
4. 等待测试完成 (约 30 分钟)
5. 全部通过后 → 进入 Phase 12

如有失败:
- [DEBUG] 查看测试报告 → 定位问题 → 修复代码 → 重新构建 → 重新测试
```

---

## Phase 12: AGC 提交审核 (Day 2, 5:30-6:30)

> **[DIALOG]** + **[VALIDATE]**

```
[VALIDATE] Claude Code 逐项确认 AGC 提交清单:

```
[VALIDATE] Claude Code 逐项确认 — AGC 提交完整检查清单:

═══════════════════════════════════════
一、应用信息
═══════════════════════════════════════
□ 1.1 应用名称: 2-30 字符, 不含禁用词
□ 1.2 应用描述: 50-8000 字符, 无夸大宣传
□ 1.3 应用图标: 512×512 px PNG, 清晰可辨
□ 1.4 应用截图: ≥ 3 张, ≥ 1080p, 真实界面

═══════════════════════════════════════
二、构建 & 签名
═══════════════════════════════════════
□ 2.1 .app 包已上传到 AGC
□ 2.2 签名证书正确 (p12 + p7b)
□ 2.3 versionCode 递增, versionName 正确
□ 2.4 minAPIVersion ≤ 目标设备 API 版本

═══════════════════════════════════════
三、云测试 (必须全部通过)
═══════════════════════════════════════
□ 3.1 兼容性: ≥ 5 款设备通过
□ 3.2 稳定性: Monkey ≥ 30min 无崩溃
□ 3.3 性能: 冷启动 < 2s, 内存 < 500MB
□ 3.4 安全: 无明文密码, 无日志敏感信息
□ 3.5 权限: 声明的权限与实际使用一致

═══════════════════════════════════════
四、隐私 & 合规
═══════════════════════════════════════
□ 4.1 隐私政策 URL 可访问
□ 4.2 所有敏感权限声明 { reason }
□ 4.3 不获取无关权限 (最小权限原则)
□ 4.4 用户数据收集说明清晰
□ 4.5 年龄分级正确

═══════════════════════════════════════
五、内容审核
═══════════════════════════════════════
□ 5.1 无违法违规内容
□ 5.2 无侵犯第三方权益 (商标/著作权)
□ 5.3 无隐藏功能或恶意行为
□ 5.4 应用功能与描述一致 (不夸大)

═══════════════════════════════════════
六、上架后维护
═══════════════════════════════════════
□ 6.1 记录上架日期 (用于软著首次发表日期)
□ 6.2 配置 AGC 崩溃监控
□ 6.3 配置 AGC 分析服务 (DAU/留存/崩溃率)
□ 6.4 准备软著申请材料 (参照 copyright/ SOP)

[DIALOG] 全部确认后:
1. AGC → 我的应用 → 版本 → "提交审核"
2. 审核周期: 3-7 个工作日
```

---

## Phase 13: 归档 & 后续维护 (Day 2, 6:30-8:00)

> **[GIT]** + **[GENERATE]** 文档归档

```bash
# [SHELL] 最终提交
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}

# 归档产出
mkdir -p docs
cp SPECS.md DESIGN_SPECS.md docs/ 2>/dev/null
cp DAY1_REPORT.md docs/ 2>/dev/null

# 生成 README
cat > README.md << READMEEOF
# ${PROJECT_NAME}

## 技术栈
- ArkTS + ArkUI + Stage 模型
- RelationalStore 本地存储
- HarmonyOS NEXT API 12+

## 项目结构
\`\`\`
entry/src/main/ets/
├── entryability/EntryAbility.ets
├── pages/ (Index, DetailPage, SettingsPage)
├── components/TaskCard.ets
├── model/TaskItem.ets
├── viewmodel/TaskViewModel.ets
├── service/ (DatabaseService, PreferenceService)
└── common/ (DesignTokens, Utils)
\`\`\`

## 构建
\`\`\`bash
./hvigorw assembleHap    # 调试
./hvigorw assembleApp    # 发布
\`\`\`
READMEEOF

# [GIT] 最终提交 & tag
git add -A
git commit -m "Release v1.0.0 - AppGallery submission ready"
git tag -a "v1.0.0" -m "Release v1.0.0"

echo "🎉 鸿蒙 App 开发完成!"
echo "   下一步: AGC 审核通过后 → 上架 → 进入软著申请流程"
```

---

## 📚 附录

### A. 关键命令速查

```bash
# 构建
./hvigorw assembleHap                         # 调试 HAP
./hvigorw assembleApp                         # 发布 APP (含签名)
./hvigorw clean                               # 清理构建产物

# 真机调试
hdc shell                                   # 连接设备 shell
hdc app install build/.../xxx.hap            # 安装应用
hdc app uninstall ${BUNDLE_NAME}             # 卸载
hdc hilog                                   # 实时日志
hdc hilog | grep "MyApp"                    # 过滤日志
hdc file recv /data/app/.../xxx.hap ./       # 从设备拉取文件

# 创建目录
mkdir -p entry/src/main/ets/{model,viewmodel,service,components,common,pages}
mkdir -p prototype/{screens,components}
```

### B. 常见 hvigorw 编译错误速查

| 错误 | 原因 | 修复 |
|------|------|------|
| `Cannot find module '@kit.xxx'` | Kit 导入路径错误或 API 版本不匹配 | 检查 oh-package.json5 依赖; API 11 使用 `@ohos.xxx` |
| `ArkTS:ERROR: Use explicit types` | 变量未声明类型 | 添加类型注解: `let x: string = ''` |
| `ForEach missing third parameter` | 缺少 keyGenerator | 添加 `(item) => item.id` |
| `Property 'xxx' does not exist` | 引用了不存在的属性 | 检查 Model 类定义 |
| `Object literal must include all properties` | 创建对象时缺少必需属性 | 补全对象属性或使用可选 `?` |
| `Struct 'xxx' has no 'build' method` | @Component 缺少 build() | 添加 `build() {}` |
| `Cannot find name 'Context'` | 缺少类型导入 | `import { Context } from '@kit.AbilityKit'` 或在 Ability 中使用 `this.context` |

### C. Phase 文件依赖关系

```
Phase 0   创建项目骨架
Phase 1   SPECS.md
Phase 1.5 DESIGN_SPECS.md + 原型文件
Phase 2   ┌─ common/DesignTokens.ets
         ├─ common/Utils.ets
         ├─ entryability/EntryAbility.ets (更新)
         └─ pages/Index.ets (Stub)
Phase 3   ┌─ model/TaskItem.ets
         ├─ service/DatabaseService.ets  ← 依赖 TaskItem
         ├─ service/PreferenceService.ets
         └─ entryability/EntryAbility.ets (添加 DB init)
Phase 4   └─ viewmodel/TaskViewModel.ets  ← 依赖 TaskItem + DatabaseService
Phase 5   ┌─ components/TaskCard.ets       ← 依赖 TaskItem + DesignTokens
         └─ pages/Index.ets (完整实现)      ← 依赖 TaskViewModel + TaskCard
Phase 6   Day 1 收尾验证
Phase 7   ┌─ pages/DetailPage.ets           ← 依赖 TaskItem + TaskViewModel
         └─ main_pages.json (更新路由)
Phase 8   ┌─ pages/SettingsPage.ets         ← 依赖 PreferenceService
         └─ pages/Index.ets (添加搜索+手势)
Phase 9   AppGallery 素材
Phase 10  签名 + 发布构建
Phase 11  AGC 云测试
Phase 12  AGC 提交审核
Phase 13  归档
```

### D. Claude Code 执行检查点

```
每次 ./hvigorw assembleHap 后必须:
□ 读取输出 → 判断成功/失败
□ 失败 → 提取错误文件+行号 → Edit 修复 → 重编译 (最多 5 次)
□ 成功 → 继续下一个 Step

每个 Phase 结束前必须:
□ ./hvigorw assembleHap 通过 ✅
□ git add -A + git commit ✅
□ 用户确认 (如有 [DIALOG] 标记)

Phase 5 额外的视觉验证:
□ DevEco Studio 打开 entry/src/main/ets/pages/Index.ets
□ 点击右上角 Previewer 标签 (View → Tool Windows → Previewer)
□ 等待预览加载 → 用户对比原型 → 回复 "视觉确认通过"
```

### E. 小白操作指南 (DevEco Studio 关键操作)

```
┌─ 如何打开 Previewer ──────────────────────────┐
│ 1. 打开任意 .ets 文件                          │
│ 2. 点击编辑器右上角的 "Previewer" 标签          │
│ 3. 或: View → Tool Windows → Previewer         │
│ 4. 预览自动刷新, 无需手动构建                    │
└──────────────────────────────────────────────┘

┌─ 如何在模拟器中运行 App ───────────────────────┐
│ 1. DevEco Studio 顶部工具栏 → 设备下拉框       │
│ 2. 选择 "Phone" 模拟器 (首次需创建)             │
│ 3. 点击绿色 ▶ 按钮运行                          │
│ 4. 如无模拟器: Tools → Device Manager → 新建    │
└──────────────────────────────────────────────┘

┌─ 如何连接真机调试 ─────────────────────────────┐
│ 1. 手机: 设置 → 关于手机 → 连续点击版本号 7次   │
│ 2. 系统 → 开发者选项 → 开启 USB 调试             │
│ 3. USB 连接电脑                                  │
│ 4. 终端: hdc shell → 确认连接                   │
│ 5. DevEco Studio 设备下拉框选择真机 → Run       │
└──────────────────────────────────────────────┘

┌─ ./hvigorw 命令速记 ──────────────────────────┐
│ cd 项目根目录   ← 必须先进入!                    │
│ ./hvigorw assembleHap      调试构建 (快速)      │
│ ./hvigorw assembleApp      发布构建 (签名)      │
│ ./hvigorw clean            清理构建产物          │
│ ⚠️ 必须加 ./ ! hvigorw 不是全局命令              │
└──────────────────────────────────────────────┘
```

---

### F. 开发故障排除 FAQ

```
Q: 执行 ./hvigorw 报 "command not found"
A: 确认在项目根目录 (有 hvigorw 文件的那一层) 执行;
   确认文件有执行权限: chmod +x hvigorw;
   DevEco Studio 创建的 hvigorw 是 Gradle wrapper，需 Java 环境。

Q: ./hvigorw assembleHap 报 "SDK not found"
A: DevEco Studio 安装后需配置 SDK 路径:
   1. DevEco Studio → Preferences → SDK
   2. 确认 HarmonyOS SDK 已下载 (API 12+)
   3. 确认 local.properties 文件中的 sdk.dir 指向正确路径

Q: Previewer 不显示，一片空白
A: 1. 等待构建完成 (Previewer 需要编译)
   2. 确认 .ets 文件无编译错误
   3. 检查 @Entry 装饰器存在
   4. 尝试: Build → Clean Project → 重新打开 Previewer

Q: 模拟器启动失败
A: 1. DevEco Studio → Tools → Device Manager
   2. 确认已创建 Phone 模拟器 (API 12+)
   3. 检查系统虚拟化是否开启 (Intel VT-x / AMD-V)
   4. macOS: 确认 /etc/hosts 包含 127.0.0.1 localhost

Q: 数据库操作崩溃
A: 1. 确认 DatabaseService.init(context) 在 EntryAbility.onCreate 中调用
   2. Context 必须从 Ability 获取，不能自己 new
   3. 所有 DB 操作需在 init 完成后 (await 异步)
   4. 检查表名/字段名与 SQL CREATE TABLE 一致

Q: List 不显示数据
A: 1. 确认 ForEach 第三个参数 keyGenerator 返回唯一值
   2. 确认 @State 变量变化会触发 UI 刷新
   3. 查看 logcat/hilog 有无错误信息
   4. 对于 async 数据加载，确保在 then() 回调中更新 @State

Q: 签名配置后构建失败
A: 1. 确认 .p12 密钥库密码正确
   2. 确认 .p7b 证书链文件存在
   3. 确认签名算法为 SHA256withECDSA
   4. 检查 .p12 文件路径 (相对于 entry/ 目录)
   5. 或直接在 DevEco Studio → Build → Generate Key 重新生成

Q: 如何调试 ArkTS 代码?
A: 1. DevEco Studio → 设置断点 → Debug 模式运行
   2. console.info() 输出到 logcat
   3. 真机: hdc hilog | grep "TAG"
   4. 模拟器: 底部 Log 面板查看
```

### G. 全流程小白自测 (提交前 5 分钟检查)

```
□ 1. ./hvigorw assembleHap 编译通过 (无 ERROR)
□ 2. Previewer 中界面显示正常 (非白屏/非崩溃)
□ 3. 列表有数据时显示列表, 无数据时显示空状态
□ 4. 添加按钮能跳转详情页
□ 5. 详情页能保存并返回
□ 6. 筛选按钮切换后列表内容变化
□ 7. 设置页开关能正常切换
□ 8. AGC 云测试全部通过
□ 9. app_icon.png 已放 AppScope/resources/base/media/
□ 10. main_pages.json 包含所有页面路径
```

> **SOP 版本**: 1.1.0 | **最后更新**: 2026-06-08
> **关联文档**: `HarmonyOS_Development_Guide.md`
> **技术栈**: ArkTS + ArkUI + Stage 模型 + RelationalStore + Huawei Account Kit
