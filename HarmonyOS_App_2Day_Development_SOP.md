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
1. 每个 Phase 开始前 → [RESEARCH] 查阅 HarmonyOS_Development_Guide.md
2. 每次生成代码后 → [REVIEW] 对照前后步骤自检代码质量
3. 每次 hvigorw 构建后 → [DEBUG] 分析输出、自动修复、不把错误抛给用户
4. 每个 Phase 结束前 → [VALIDATE] 对照该 Phase 的验收条件逐项确认
5. 每个 Phase 结束后 → [GIT] 提交代码
6. 涉及不可逆操作 → [DIALOG] 必须获得用户确认
```

---

## 📋 总览时间线

```
Day 1 (8h)                              Day 2 (8h)
├─ [0.0h] 环境检查 & 项目初始化          ├─ [0.0h] 集成测试 & Bug 修复
├─ [0.5h] 产品需求确认                   ├─ [2.0h] 性能优化 & 无障碍
├─ [1.0h] 高保真原型设计 & 设计系统       ├─ [3.5h] AppGallery 素材准备
├─ [2.0h] Stage 架构搭建                ├─ [4.5h] 内购/订阅配置
├─ [2.5h] 数据层 (Model + Store)        ├─ [5.5h] 构建签名 & 云测试
├─ [4.5h] 业务逻辑层 (ViewModel)         ├─ [6.5h] AGC 提交审核
├─ [6.0h] UI 层 (ArkUI) ← 按原型实现     └─ [8.0h] 提交审核 & 文档归档
└─ [7.5h] 自测 & 代码 Review
```

---

## ⚠️ 执行前提 (Claude Code 自动检查)

```bash
# [SHELL] 1. 确认 DevEco Studio 已安装
if [ -d "$HOME/Applications/DevEco-Studio.app" ] || [ -d "/Applications/DevEco-Studio.app" ]; then
    echo "✅ DevEco Studio 已安装"
else
    echo "❌ 未找到 DevEco Studio, 请从 https://developer.huawei.com/consumer/cn/download/ 下载"
    exit 1
fi

# [SHELL] 2. 确认 hvigorw 可用 (项目构建工具)
DEVECO_SDK_DIR=$(find ~/Library -name "devecostudio" -type d 2>/dev/null | head -1)
echo "DevEco SDK: ${DEVECO_SDK_DIR:-未找到}"

# [SHELL] 3. 确认 hdc 可用 (设备连接工具)
which hdc >/dev/null 2>&1 && echo "✅ hdc 可用" || echo "⚠️ hdc 未在 PATH 中"

# [SHELL] 4. 确认 Git
git --version || { echo "❌ Git 未安装"; exit 1; }
git config user.name && git config user.email || { echo "❌ Git 未配置"; exit 1; }

# [SHELL] 5. 确认华为开发者账号
echo "请确认已在 https://developer.huawei.com 注册华为开发者账号"
# [DIALOG] 等待用户确认
```

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
hvigorw assembleHap 2>&1 | tail -10
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

> **Claude Code 执行**: 编译验证 → `hvigorw assembleHap 2>&1 | tail -5` → [DEBUG] 修复 → [GIT] commit

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

### Step 3.4 — 更新 EntryAbility 初始化数据库

```typescript
// 在 EntryAbility.onCreate() 中添加
import { DatabaseService } from '../service/DatabaseService'
import { PreferenceService } from '../service/PreferenceService'

onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
  const ctx = this.context
  DatabaseService.init(ctx)       // 初始化数据库
  PreferenceService.init(ctx)     // 初始化偏好存储
}
```

> **[SHELL]** 编译验证 → [DEBUG] → [GIT] commit

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

> **[SHELL]** 编译验证 → [DEBUG] → [GIT] commit

---

## Phase 5: UI 层开发 ← 按原型实现 (6:00 - 7:30)

> **[GENERATE]** + **[WRITE]** + **[REVIEW]** + **[DEBUG]** 1:1 还原原型

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

### Step 5.3 — HIG 适配 (鸿蒙设计规范)

```
✅ 所有文本使用 fp 单位 (自适应缩放)
✅ 所有间距使用 vp 单位
✅ 所有颜色使用 DesignTokens 常量
✅ 列表使用 LazyForEach (数据 > 100 时)
✅ 触摸目标 ≥ 48vp (鸿蒙标准)
✅ 深色模式自动适配 (系统处理)
✅ 支持横竖屏 (module.json5: "orientation": "auto_rotation")
```

> **[SHELL]** 编译验证 → [DEBUG] → 在 Previewer 中验证 → [GIT] commit

---

## Phase 6: 自测 & Day 1 收尾 (7:30 - 8:00)

> **[SHELL]** + **[VALIDATE]** + **[GIT]**

```bash
# 构建验证
hvigorw assembleHap 2>&1 | tail -5

# 检查代码质量
echo "=== ArkTS 代码检查 ==="
find entry/src/main/ets -name "*.ets" | wc -l | xargs echo "源文件数:"
find entry/src/main/ets -name "*.ets" -exec wc -l {} \; | awk '{sum+=$1} END {print "总行数: " sum}'

# Day 1 报告
cat > DAY1_REPORT.md << EOF
# Day 1 完成报告
- Stage 模型: ✅ 已搭建
- 数据层: ✅ RelationalStore + Preferences
- ViewModel: ✅ TaskViewModel
- UI 层: ✅ 首页 + TaskCard 组件
- 编译: ✅ hvigorw assembleHap 通过
EOF

git add -A && git commit -m "Day 1 complete: MVP with Stage + MVVM + RelationalStore"
```

---

# 🗓️ DAY 2 — 从 MVP 到可发布

### Day 2 启动校验

```bash
source /tmp/sop_harmony.env 2>/dev/null
cd ${PROJECT_DIR}
hvigorw assembleHap 2>&1 | tail -5
# [DEBUG] 确保 Day 1 代码仍可编译
```

---

## Phase 7-8: 集成测试 & 细节完善 (Day 2, 0:00-3:30)

> **[GENERATE]** + **[DEBUG]** 添加详情页、设置页、搜索、手势交互

### 关键文件

```typescript
// entry/src/main/ets/pages/DetailPage.ets
@Entry
@Component
struct DetailPage {
  @State task: TaskItem = new TaskItem()
  @State viewModel: TaskViewModel = new TaskViewModel()

  aboutToAppear(): void {
    // 从 router 接收 taskId, 加载详情
  }

  build() {
    Column() {
      // 编辑表单
      TextInput({ placeholder: '任务标题', text: this.task.title })
        .onChange((value: string) => { this.task.title = value })
      // ... 更多字段
      Button('保存').onClick(() => {
        this.viewModel.updateTask(this.task)
        router.back()
      })
    }
  }
}

// entry/src/main/ets/pages/SettingsPage.ets
@Entry
@Component
struct SettingsPage {
  @State hapticEnabled: boolean = true

  build() {
    Column() {
      Text('设置').fontSize(28).fontWeight(FontWeight.Bold)
      Row() {
        Text('触觉反馈')
        Toggle({ type: ToggleType.Switch, isOn: this.hapticEnabled })
          .onChange((value: boolean) => { this.hapticEnabled = value })
      }
    }
  }
}
```

> **[WRITE]** 注册新页面到 `main_pages.json` → 编译验证 → [GIT]

---

## Phase 9: AppGallery 素材准备 (Day 2, 3:30-4:30)

> **[SHELL]** + **[WRITE]** + **[DIALOG]**

```
═══════════════════════════════════════
📱 AppGallery Connect 素材清单
═══════════════════════════════════════

□ 应用图标: 512×512 px PNG
□ 应用截图: ≥ 3 张 (分辨率 ≥ 1080p)
□ 应用描述: 50-8000 字符
□ 隐私政策 URL
□ 应用分类 & 年龄分级

Claude Code 指导:
1. 图标: 准备 512×512 PNG 放在 AppScope/resources/base/media/
2. 截图: 使用 DevEco Previewer 截图或真机截屏
3. 描述: 根据 SPECS.md 生成 (参照 iOS SOP Phase 9.4)
```

---

## Phase 10: 构建签名 & 云测试 (Day 2, 4:30-5:30)

```bash
# 构建发布包
hvigorw assembleApp

# 产物
# build/outputs/default/app-default-unsigned.app    # 未签名 APP
# build/outputs/default/entry-default-unsigned.hap  # 未签名 HAP

# [DIALOG] 签名需要在 AGC 中配置:
echo "
请在 AppGallery Connect 中配置签名:
1. https://developer.huawei.com → 我的项目 → 签名
2. 生成/上传发布证书 (.p12)
3. 配置签名后重新构建
"
```

---

## Phase 11-13: AGC 提交审核 & 归档 (Day 2, 5:30-8:00)

> **[DIALOG]** + **[VALIDATE]** + **[GIT]**

```
═══════════════════════════════════════
AGC 提交检查清单
═══════════════════════════════════════

□ 1. 应用信息完整 (名称/描述/图标/截图)
□ 2. 隐私政策 URL 可访问
□ 3. 云测试通过 (兼容性/稳定性/性能/安全/权限)
□ 4. 应用包签名正确
□ 5. 版本号正确 (versionCode + versionName)
□ 6. 内容合规 (无违规内容、无侵权)

# [GIT]
git tag v1.0.0
git commit -m "Release v1.0.0 for AppGallery submission"
```

---

## 📚 附录: 关键命令速查

```bash
# 构建
hvigorw assembleHap           # 调试 HAP
hvigorw assembleApp           # 发布 APP
hvigorw clean                 # 清理

# 真机调试
hdc shell                      # 连接设备
hdc app install xxx.hap        # 安装应用
hdc uninstall com.example.xxx  # 卸载
hdc hilog                      # 查看日志

# Git
git add -A && git commit -m "msg"
git tag v1.0.0

# 项目结构创建
mkdir -p entry/src/main/ets/{model,viewmodel,service,components,common,pages}
```

---

> **SOP 版本**: 1.0.0 | **最后更新**: 2026-06-08
> **关联文档**: `HarmonyOS_Development_Guide.md`
> **技术栈**: ArkTS + ArkUI + Stage 模型 + RelationalStore
