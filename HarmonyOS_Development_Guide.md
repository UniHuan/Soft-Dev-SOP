# 🦋 鸿蒙 (HarmonyOS NEXT) 开发规范文档

> **来源**: 华为开发者联盟 developer.huawei.com + ArkTS/ArkUI 官方文档
> **适用版本**: HarmonyOS NEXT (API 12+) / HarmonyOS 5
> **适用对象**: 鸿蒙应用开发者、Claude Code AI Agent
> **核心理念**: ArkTS 声明式 UI + Stage 模型 + 一次开发多设备部署

---

## 目录

1. [HarmonyOS NEXT 架构概览](#1-harmonyos-next-架构概览)
2. [ArkTS 语言规范](#2-arkts-语言规范)
3. [ArkUI 声明式 UI 开发](#3-arkui-声明式-ui-开发)
4. [Stage 模型 & 应用架构](#4-stage-模型--应用架构)
5. [组件 & 布局规范](#5-组件--布局规范)
6. [状态管理](#6-状态管理)
7. [导航 & 路由](#7-导航--路由)
8. [数据持久化](#8-数据持久化)
9. [网络请求规范](#9-网络请求规范)
10. [多设备适配 & 响应式布局](#10-多设备适配--响应式布局)
11. [安全 & 隐私规范](#11-安全--隐私规范)
12. [性能优化规范](#12-性能优化规范)
13. [测试规范](#13-测试规范)
14. [AppGallery 上架规范](#14-appgallery-上架规范)
15. [开发检查清单](#15-开发检查清单)

---

## 1. HarmonyOS NEXT 架构概览

### 1.1 系统分层

```
┌─────────────────────────────────────────────┐
│              应用层 (ArkTS/ArkUI)             │
│  ┌─────────┐ ┌─────────┐ ┌───────────────┐  │
│  │ 元服务   │ │ 应用     │ │ 原子化服务     │  │
│  └─────────┘ └─────────┘ └───────────────┘  │
├─────────────────────────────────────────────┤
│           应用框架层 (Stage 模型)             │
│  ┌─────────┐ ┌─────────┐ ┌───────────────┐  │
│  │ Ability │ │ UIAbility│ │ ExtensionAbility│ │
│  │ Stage   │ │ Component│ │                │  │
│  └─────────┘ └─────────┘ └───────────────┘  │
├─────────────────────────────────────────────┤
│       系统服务层 (Kit 化能力开放)              │
│  ArkUI | 网络 | 数据 | 媒体 | 安全 | AI   │
├─────────────────────────────────────────────┤
│            内核层 (鸿蒙微内核)                 │
└─────────────────────────────────────────────┘
```

### 1.2 核心概念

| 概念 | 说明 | iOS 对比 |
|------|------|---------|
| **ArkTS** | 鸿蒙官方开发语言，TypeScript 超集 | Swift |
| **ArkUI** | 声明式 UI 框架 | SwiftUI |
| **Stage 模型** | 应用组件模型，管理 Ability 生命周期 | App/Scene |
| **UIAbility** | 包含 UI 界面的 Ability，用户交互入口 | UIApplication |
| **ExtensionAbility** | 无 UI 的后台 Ability | Background Task |
| **HAP** | HarmonyOS Ability Package，应用包 | .app bundle |
| **HAR** | HarmonyOS Archive，静态共享库 | .framework |
| **HSP** | HarmonyOS Shared Package，动态共享库 | .xcframework |
| **元服务** | 免安装、即用即走的轻量服务 | App Clips |
| **原子化服务** | 可独立部署的最小功能单元 | Widget |

### 1.3 开发工具链

| 工具 | 用途 |
|------|------|
| **DevEco Studio** | 官方 IDE (基于 IntelliJ) |
| **hdc** | 鸿蒙设备连接工具 (类似 adb) |
| **ArkCompiler** | ArkTS 编译器 (方舟编译器) |
| **Previewer** | 实时预览器 |
| **Inspector** | UI 层级调试工具 |
| **Profiler** | 性能分析工具 |

---

## 2. ArkTS 语言规范

### 2.1 ArkTS 特性

ArkTS 是 TypeScript 的超集，在此基础上增加了声明式 UI 和状态管理的语法扩展。

```
ArkTS = TypeScript + 声明式 UI 语法 + 状态管理装饰器 + 方舟编译优化
```

### 2.2 类型系统规范

```typescript
// ✅ 推荐: 显式类型声明
let count: number = 0
let name: string = 'HarmonyOS'
let items: Array<string> = ['a', 'b', 'c']
let config: Record<string, number> = { width: 100, height: 200 }

// ✅ 推荐: 使用 interface 定义数据结构
interface TaskItem {
  id: string
  title: string
  priority: Priority
  isCompleted: boolean
  createdAt: Date
}

// ✅ 推荐: 使用 enum 定义枚举
enum Priority {
  LOW = 0,
  MEDIUM = 1,
  HIGH = 2,
  URGENT = 3
}

// ❌ 禁止: any (破坏类型安全)
// let data: any = fetchData()  // 禁止!

// ✅ 替代: unknown + 类型守卫
let data: unknown = fetchData()
if (typeof data === 'object' && data !== null) {
  // 安全使用
}
```

### 2.3 命名规范

```typescript
// 文件和目录: snake_case
// 类、接口、枚举、类型别名: PascalCase
// 变量、函数、方法、属性: camelCase
// 常量: UPPER_SNAKE_CASE
// 私有属性: 下划线前缀 _privateProperty

// ✅ 示例
const MAX_RETRY_COUNT = 3

interface UserProfile {
  userId: string
  displayName: string
  avatarUrl: string
}

class TaskService {
  private _cache: Map<string, TaskItem> = new Map()

  fetchTaskById(id: string): Promise<TaskItem> {
    // ...
  }
}

function calculatePriorityScore(tasks: TaskItem[]): number {
  // ...
}
```

### 2.4 装饰器使用规范

```typescript
// ArkTS 核心装饰器
@Component    // 自定义组件
@Entry        // 入口组件 (页面)
@State        // 组件内状态
@Prop         // 父→子单向传递
@Link         // 父↔子双向绑定
@Provide      // 跨层级提供
@Consume      // 跨层级消费
@Observed     // 观察类
@ObjectLink   // 观察对象引用
@StorageLink  // AppStorage 双向绑定
@StorageProp  // AppStorage 单向绑定
@Watch        // 状态变化监听

// 排列顺序规范 (在 struct 前):
// 1. @Entry (如有)
// 2. @Component
// 3. @Preview (如有)
```

### 2.5 禁止使用的 TypeScript 特性

```
ArkTS 限制 (方舟编译器不支持):
❌ any / unknown 的随意使用 → 严格类型
❌ eval() → 禁止
❌ with 语句 → 禁止
❌ Function 构造函数 → 禁止
❌ Reflect → 部分限制
❌ 动态导入 import() → 使用静态 import
❌ Symbol → 部分限制
❌ 原型链修改 → 禁止
❌ 删除对象属性 delete → 设为 null/undefined
```

---

## 3. ArkUI 声明式 UI 开发

### 3.1 组件声明规范

```typescript
// ✅ 标准组件结构
@Component
export struct TaskCard {
  // 1. 属性参数
  @Prop task: TaskItem
  @Prop isEditable: boolean = false

  // 2. 内部状态
  @State isExpanded: boolean = false
  @State localTitle: string = ''

  // 3. 生命周期
  aboutToAppear(): void {
    this.localTitle = this.task.title
  }

  aboutToDisappear(): void {
    // 清理资源
  }

  // 4. 计算属性 (用函数)
  getPriorityColor(): ResourceColor {
    switch (this.task.priority) {
      case Priority.URGENT: return '#FF3B30'
      case Priority.HIGH:   return '#FF9500'
      case Priority.MEDIUM: return '#007AFF'
      default:              return '#8E8E93'
    }
  }

  // 5. build 方法 (唯一必需)
  build() {
    Column() {
      Row() {
        Text(this.task.title)
          .fontSize(17)
          .fontWeight(FontWeight.Medium)
          .layoutWeight(1)
          .maxLines(2)
          .textOverflow({ overflow: TextOverflow.Ellipsis })

        Text(this.task.priority.toString())
          .fontSize(12)
          .fontColor(this.getPriorityColor())
          .padding({ left: 8, right: 8, top: 2, bottom: 2 })
          .backgroundColor(this.getPriorityColor() + '20')
          .borderRadius(6)
      }
      .width('100%')
      .padding(16)
    }
    .width('100%')
    .backgroundColor(Color.White)
    .borderRadius(12)
    .shadow({ radius: 8, color: '#00000010', offsetY: 2 })
  }
}
```

### 3.2 样式规范

```typescript
// ✅ 使用常量管理设计 Token
class DesignTokens {
  // 颜色
  static readonly COLOR_PRIMARY = '#007AFF'
  static readonly COLOR_BG = '#F2F2F7'
  static readonly COLOR_TEXT_PRIMARY = '#000000'
  static readonly COLOR_TEXT_SECONDARY = '#8E8E93'
  static readonly COLOR_SEPARATOR = '#C6C6C8'

  // 间距 (vp = virtual pixel)
  static readonly SPACING_XS = 4
  static readonly SPACING_SM = 8
  static readonly SPACING_MD = 16
  static readonly SPACING_LG = 24
  static readonly SPACING_XL = 32

  // 圆角
  static readonly RADIUS_SM = 8
  static readonly RADIUS_MD = 12
  static readonly RADIUS_LG = 16

  // 字号 (fp = font pixel, 自适应缩放)
  static readonly FONT_CAPTION = 12
  static readonly FONT_BODY = 16
  static readonly FONT_TITLE = 20
  static readonly FONT_LARGE_TITLE = 28
}

// ✅ 统一链式调用顺序 (可读性)
// 1. 尺寸 (width/height/size)
// 2. 间距 (padding/margin)
// 3. 外观 (backgroundColor/border/opacity)
// 4. 字体 (fontSize/fontColor/fontWeight)
// 5. 布局 (layoutWeight/align)
// 6. 事件 (onClick/onChange/gesture)
// 7. 状态 (enabled/visibility)
```

### 3.3 条件渲染 & 循环渲染

```typescript
// ✅ 条件渲染
build() {
  Column() {
    if (this.isLoading) {
      LoadingProgress()
    } else if (this.hasError) {
      ErrorView({ message: this.errorMessage })
    } else if (this.items.length === 0) {
      EmptyView({ hint: '暂无数据' })
    } else {
      ListView({ items: this.items })
    }
  }
}

// ✅ 循环渲染 — ForEach (必须提供 keyGenerator)
ForEach(this.filteredTasks,
  (task: TaskItem) => {
    ListItem() {
      TaskCard({ task: task })
    }
  },
  (task: TaskItem) => task.id  // ← 唯一 key, 必须!
)

// ✅ LazyForEach — 懒加载大列表
class TaskDataSource implements IDataSource {
  // 实现 totalCount(), getData(index), registerDataChangeListener()
}

LazyForEach(this.dataSource,
  (task: TaskItem, index: number) => {
    ListItem() {
      TaskCard({ task: task })
    }
  },
  (task: TaskItem) => task.id
)
```

---

## 4. Stage 模型 & 应用架构

### 4.1 Stage 模型目录结构

```
entry/                          # 主模块
├── src/main/
│   ├── ets/                    # ArkTS 源码
│   │   ├── entryability/       # UIAbility
│   │   │   └── EntryAbility.ets
│   │   ├── pages/              # 页面
│   │   │   ├── Index.ets
│   │   │   ├── DetailPage.ets
│   │   │   └── SettingsPage.ets
│   │   ├── components/         # 自定义组件
│   │   │   ├── TaskCard.ets
│   │   │   └── EmptyView.ets
│   │   ├── model/              # 数据模型
│   │   │   └── TaskItem.ets
│   │   ├── viewmodel/          # 视图模型
│   │   │   └── TaskViewModel.ets
│   │   ├── service/            # 服务层
│   │   │   ├── DatabaseService.ets
│   │   │   └── NetworkService.ets
│   │   └── common/             # 公共工具
│   │       ├── constants.ets
│   │       └── utils.ets
│   ├── resources/              # 资源文件
│   │   ├── base/               # 默认资源
│   │   │   ├── element/        # 字符串/颜色/数字
│   │   │   ├── media/          # 图片/音视频
│   │   │   └── profile/        # 配置文件
│   │   └── rawfile/            # 原始文件
│   └── module.json5            # 模块配置
├── build-profile.json5         # 构建配置
├── hvigorfile.ts               # 构建脚本
└── oh-package.json5            # 依赖管理

AppScope/                       # 应用全局配置
└── app.json5                   # 应用 bundleName/versionCode/icon
```

### 4.2 UIAbility 生命周期

```typescript
// entryability/EntryAbility.ets
export default class EntryAbility extends UIAbility {
  // 创建时 (加载页面)
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    hilog.info(0x0000, 'EntryAbility', 'onCreate')
    // 初始化全局服务、数据库等
  }

  // 销毁时 (进程被杀)
  onDestroy(): void {
    hilog.info(0x0000, 'EntryAbility', 'onDestroy')
    // 清理资源
  }

  // 显示在屏幕 (进入前台)
  onForeground(): void {
    hilog.info(0x0000, 'EntryAbility', 'onForeground')
    // 恢复 UI、刷新数据
  }

  // 隐藏到后台
  onBackground(): void {
    hilog.info(0x0000, 'EntryAbility', 'onBackground')
    // 保存状态、释放大内存
  }

  // 窗口创建
  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.loadContent('pages/Index',
      (err, data) => {
        if (err.code) {
          hilog.error(0x0000, 'EntryAbility', 'Failed to load content')
          return
        }
        // 加载成功
      })
  }
}
```

### 4.3 module.json5 关键配置

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "srcEntry": "./ets/entryability/EntryAbility.ets",
    "description": "$string:module_desc",
    "mainElement": "EntryAbility",
    "deviceTypes": [
      "phone",      // 手机
      "tablet",     // 平板
      "2in1"        // 二合一
    ],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",  // 页面路由配置
    "abilities": [{
      "name": "EntryAbility",
      "srcEntry": "./ets/entryability/EntryAbility.ets",
      "launchType": "singleton",
      "orientation": "auto_rotation",
      "backgroundModes": ["audioPlayback"]
    }]
  }
}
```

### 4.4 MVVM 架构推荐

```typescript
// Model — 纯数据
export class TaskItem {
  id: string = ''
  title: string = ''
  priority: Priority = Priority.MEDIUM
  isCompleted: boolean = false
  createdAt: Date = new Date()
}

// ViewModel — 业务逻辑 + 状态
export class TaskViewModel {
  @Trace tasks: TaskItem[] = []
  @Trace isLoading: boolean = false
  @Trace selectedFilter: FilterOption = FilterOption.ALL
  private service: DatabaseService = new DatabaseService()

  async loadTasks(): Promise<void> {
    this.isLoading = true
    try {
      this.tasks = await this.service.fetchAll()
    } catch (e) {
      hilog.error(0x0000, 'TaskVM', `Load failed: ${e}`)
    } finally {
      this.isLoading = false
    }
  }

  get filteredTasks(): TaskItem[] {
    return this.tasks.filter(t => {
      switch (this.selectedFilter) {
        case FilterOption.ACTIVE: return !t.isCompleted
        case FilterOption.COMPLETED: return t.isCompleted
        default: return true
      }
    })
  }
}

// View — 纯 UI
@Entry
@Component
struct TaskListPage {
  @State viewModel: TaskViewModel = new TaskViewModel()

  aboutToAppear(): void {
    this.viewModel.loadTasks()
  }

  build() {
    Navigation() {
      List() {
        ForEach(this.viewModel.filteredTasks,
          (task: TaskItem) => {
            ListItem() { TaskCard({ task: task }) }
          },
          (task: TaskItem) => task.id
        )
      }
    }
    .title('任务列表')
  }
}
```

---

## 5. 组件 & 布局规范

### 5.1 核心布局组件

```typescript
// Column — 垂直布局
Column({ space: DesignTokens.SPACING_MD }) {
  Text('标题').fontSize(20)
  Text('副标题').fontSize(14).fontColor('#999')
}
.width('100%')
.padding(16)

// Row — 水平布局
Row({ space: DesignTokens.SPACING_SM }) {
  Image($r('app.media.icon')).width(24).height(24)
  Text('标签').layoutWeight(1)  // 弹性填充
  Image($r('app.media.arrow_right')).width(16).height(16)
}
.width('100%')
.alignItems(VerticalAlign.Center)

// Flex — 弹性布局
Flex({ direction: FlexDirection.Row, wrap: FlexWrap.Wrap }) {
  // 自动换行的标签组
}

// Grid — 网格布局
Grid() {
  GridItem() { /* ... */ }
  GridItem() { /* ... */ }
}
.columnsTemplate('1fr 1fr')  // 两列等宽
.columnsGap(12)
.rowsGap(12)

// Stack — 层叠布局
Stack({ alignContent: Alignment.BottomEnd }) {
  Image($r('app.media.background')).width('100%')  // 底层
  Text('浮层文字')                                   // 顶层
    .margin(16)
}

// RelativeContainer — 相对布局 (类似 AutoLayout)
RelativeContainer() {
  Text('A').alignRules({ left: { anchor: '__container__', align: HorizontalAlign.Start } })
  Text('B').alignRules({ left: { anchor: 'A', align: HorizontalAlign.End } })
}
```

### 5.2 组件开发规范

```typescript
// ✅ 组件必须:
// 1. 有明确的 @Prop/@Link/@State 参数声明
// 2. 单一职责 (一个组件只做一件事)
// 3. 可复用 (不依赖全局状态, 通过参数传递)
// 4. 有 @Preview 用于 DevEco Studio 预览

@Preview({
  title: 'TaskCard - 高优先级',
  width: 360,
  height: 80
})
@Component
export struct TaskCard {
  @Prop task: TaskItem

  build() {
    // ...
  }
}

// ❌ 禁止:
// - 组件内直接访问数据库/网络 (都应通过 ViewModel/Service)
// - 组件超过 200 行
// - build() 方法中有复杂逻辑 (>10 行)
// - 硬编码的颜色/字号 (应使用 DesignTokens 或 $r())
```

### 5.3 常用系统组件速查

| 组件 | 用途 | 关键属性 |
|------|------|---------|
| `Text` | 文本显示 | fontSize, fontColor, maxLines, textOverflow |
| `Image` | 图片 | source, objectFit, borderRadius |
| `Button` | 按钮 | type(ButtonType.Capsule), stateEffect |
| `TextInput` | 输入框 | placeholder, type(InputType.Password), maxLength |
| `TextArea` | 多行输入 | placeholder, maxLength |
| `List` | 列表 | space, stickyHeader, scrollBar |
| `Swiper` | 轮播 | autoPlay, interval, indicator |
| `Scroll` | 可滚动容器 | scrollBar, scrollable |
| `Refresh` | 下拉刷新 | refreshing, onRefreshing |
| `Progress` | 进度条 | value, type(ProgressType.Linear) |
| `LoadingProgress` | 加载指示 | color |
| `AlertDialog` | 弹窗 | title, message, confirm, cancel |
| `ActionSheet` | 操作菜单 | sheets |
| `Toggle` | 开关 | type(ToggleType.Switch), isOn |
| `Slider` | 滑块 | value, min, max, step |
| `Checkbox` | 复选框 | select, mark |
| `Radio` | 单选框 | value, group |
| `Blank` | 弹性空白 | (占位, 类似 SwiftUI Spacer) |
| `Divider` | 分割线 | vertical, color, strokeWidth |

---

## 6. 状态管理

### 6.1 状态装饰器选择决策树

```
组件内部状态 → @State
父→子单向传递 → @Prop
父↔子双向绑定 → @Link
跨层级共享 (祖先→后代) → @Provide + @Consume
应用全局状态 → AppStorage / LocalStorage
对象/类级别观察 → @Observed + @ObjectLink
V2 响应式 (API 12+) → @ObservedV2 + @Trace
```

### 6.2 状态管理最佳实践

```typescript
// ✅ V1 装饰器 (API 9-11)
@Component
struct CounterV1 {
  @State count: number = 0       // 私有状态
  @Prop title: string             // 父传子 (只读)
  @Link config: AppConfig         // 双向绑定
  @Provide('theme') theme: string = 'light'  // 向下提供
  @Consume('theme') currentTheme: string     // 消费上层提供的

  build() {
    Column() {
      Text(`${this.title}: ${this.count}`)
      Button('+1').onClick(() => { this.count++ })
    }
  }
}

// ✅ V2 装饰器 (API 12+, 推荐)
@ObservedV2
class TaskStore {
  @Trace tasks: TaskItem[] = []
  @Trace selectedId: string = ''
  @Trace isLoading: boolean = false

  // @Trace 自动追踪属性变化, 无需手动 @State
}

@ComponentV2
struct TaskListV2 {
  @Local taskStore: TaskStore = new TaskStore()   // 组件内状态
  @Param title: string = ''                        // 父传子
  @Param @Once icon: ResourceStr = ''              // 仅初始化一次

  build() {
    List() {
      ForEach(this.taskStore.tasks,
        (task: TaskItem) => { ListItem() { /*...*/ } },
        (task: TaskItem) => task.id
      )
    }
  }
}
```

### 6.3 状态更新规则

```
✅ 可以修改: @State, @Link, @Provide 修饰的变量
❌ 不可修改: @Prop 修饰的变量 (只读)
✅ 数组/对象: 使用 class + @Observed 或 V2 @Trace
⚠️ 异步更新: 必须在 UI 线程更新状态
   错误: Promise.then(() => this.count++)  (可能不在UI线程)
   正确: 使用 TaskPool 或 @Concurrent 标记异步任务
```

---

## 7. 导航 & 路由

### 7.1 Navigation 组件 (推荐, API 9+)

```typescript
// ✅ 声明式导航 (Navigation + NavPathStack)
@Entry
@Component
struct MainPage {
  @Provide('navPathStack') pageStack: NavPathStack = new NavPathStack()

  build() {
    Navigation(this.pageStack) {
      List() {
        ListItem() {
          Text('任务详情')
            .onClick(() => {
              // push 到详情页
              this.pageStack.pushPathByName('DetailPage',
                { taskId: '123', title: '任务标题' })
            })
        }
      }
    }
    .title('首页')
    .mode(NavigationMode.Stack)  // 单页面堆栈
  }
}

// 目标页面
@Entry
@Component
struct DetailPage {
  @Consume('navPathStack') pageStack: NavPathStack
  @State taskId: string = ''
  @State title: string = ''

  build() {
    NavDestination() {
      Column() {
        Text(this.title).fontSize(24)
        Text(`ID: ${this.taskId}`)
      }
    }
    .title(this.title)
    .onReady((context: NavDestinationContext) => {
      // 接收路由参数
      this.taskId = context.pathInfo?.param?.taskId ?? ''
      this.title = context.pathInfo?.param?.title ?? ''
    })
  }
}
```

### 7.2 Router 路由 (旧版, 兼容)

```typescript
import router from '@ohos.router'

// push
router.pushUrl({
  url: 'pages/DetailPage',
  params: { taskId: '123' }
})

// 接收参数
const params = router.getParams() as Record<string, Object>
const taskId = params['taskId'] as string

// pop (返回)
router.back()

// replace (替换当前页)
router.replaceUrl({ url: 'pages/LoginPage' })

// 路由配置 (main_pages.json)
// {
//   "src": ["pages/Index", "pages/DetailPage", "pages/SettingsPage"]
// }
```

### 7.3 Tab 导航

```typescript
@Entry
@Component
struct MainTabs {
  @State currentIndex: number = 0
  private tabsController: TabsController = new TabsController()

  build() {
    Tabs({ barPosition: BarPosition.End, controller: this.tabsController }) {
      TabContent() {
        HomePage()
      }.tabBar(this.tabBuilder('首页', $r('sys.symbol.house_fill'), 0))

      TabContent() {
        DiscoverPage()
      }.tabBar(this.tabBuilder('发现', $r('sys.symbol.magnifyingglass'), 1))

      TabContent() {
        ProfilePage()
      }.tabBar(this.tabBuilder('我的', $r('sys.symbol.person_fill'), 2))
    }
    .onChange((index: number) => { this.currentIndex = index })
  }

  @Builder tabBuilder(text: string, icon: Resource, index: number) {
    Column() {
      Image(icon).width(24).height(24)
        .fillColor(index === this.currentIndex ? '#007AFF' : '#8E8E93')
      Text(text).fontSize(10)
        .fontColor(index === this.currentIndex ? '#007AFF' : '#8E8E93')
    }
  }
}
```

---

## 8. 数据持久化

### 8.1 存储方案选择

| 方案 | 适用场景 | 容量 |
|------|---------|------|
| **Preferences** | 简单键值对 (设置、配置) | 小 |
| **KV Store** | 分布式键值数据库 | 中 |
| **RelationalStore** | 关系型数据库 (SQLite) | 大 |
| **ObjectStore** | 对象数据库 (NoSQL) | 大 |
| **UDMF** | 统一数据管理框架 | — |
| **DataShare** | 跨应用数据共享 | — |

### 8.2 Preferences (键值存储)

```typescript
import preferences from '@ohos.data.preferences'

// 初始化
const PREFERENCES_NAME = 'app_settings'
let prefs: preferences.Preferences

async function initPreferences(context: Context): Promise<void> {
  prefs = preferences.getPreferencesSync(context, { name: PREFERENCES_NAME })
}

// 读写
async function saveUserPreference(key: string, value: string): Promise<void> {
  await prefs.put(key, value)
  await prefs.flush()  // 持久化到磁盘
}

async function loadUserPreference(key: string): Promise<string> {
  return await prefs.get(key, '') as string
}

// 监听变化
prefs.on('change', (key: string) => {
  hilog.info(0x0000, 'Prefs', `Key ${key} changed`)
})
```

### 8.3 RelationalStore (关系型数据库)

```typescript
import relationalStore from '@ohos.data.relationalStore'

// 表结构定义
const SQL_CREATE_TABLE = `
  CREATE TABLE IF NOT EXISTS tasks (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    priority INTEGER DEFAULT 0,
    is_completed INTEGER DEFAULT 0,
    created_at INTEGER NOT NULL
  )
`

// 初始化
let store: relationalStore.RdbStore

async function initDatabase(context: Context): Promise<void> {
  const config: relationalStore.StoreConfig = {
    name: 'task_database.db',
    securityLevel: relationalStore.SecurityLevel.S1
  }

  store = await relationalStore.getRdbStore(context, config)
  await store.executeSql(SQL_CREATE_TABLE)
}

// CRUD 操作
async function insertTask(task: TaskItem): Promise<number> {
  const valueBucket: relationalStore.ValuesBucket = {
    'id': task.id,
    'title': task.title,
    'priority': task.priority,
    'is_completed': task.isCompleted ? 1 : 0,
    'created_at': task.createdAt.getTime()
  }
  return await store.insert('tasks', valueBucket)
}

async function queryTasks(filterCompleted: boolean = false): Promise<TaskItem[]> {
  let predicates = new relationalStore.RdbPredicates('tasks')
  if (filterCompleted) {
    predicates.equalTo('is_completed', 1)
  }
  const resultSet = await store.query(predicates)
  const tasks: TaskItem[] = []

  while (resultSet.goToNextRow()) {
    tasks.push({
      id: resultSet.getString(resultSet.getColumnIndex('id')),
      title: resultSet.getString(resultSet.getColumnIndex('title')),
      priority: resultSet.getLong(resultSet.getColumnIndex('priority')),
      isCompleted: resultSet.getLong(resultSet.getColumnIndex('is_completed')) === 1,
      createdAt: new Date(resultSet.getLong(resultSet.getColumnIndex('created_at')))
    })
  }
  resultSet.close()
  return tasks
}

async function updateTask(task: TaskItem): Promise<number> {
  const valueBucket: relationalStore.ValuesBucket = {
    'title': task.title,
    'priority': task.priority,
    'is_completed': task.isCompleted ? 1 : 0
  }
  let predicates = new relationalStore.RdbPredicates('tasks')
  predicates.equalTo('id', task.id)
  return await store.update(valueBucket, predicates)
}

async function deleteTask(taskId: string): Promise<number> {
  let predicates = new relationalStore.RdbPredicates('tasks')
  predicates.equalTo('id', taskId)
  return await store.delete(predicates)
}
```

---

## 9. 网络请求规范

### 9.1 HTTP 请求

```typescript
import http from '@ohos.net.http'

class NetworkService {
  private static readonly BASE_URL = 'https://api.example.com/v1'
  private static readonly TIMEOUT = 30000

  static async request<T>(
    endpoint: string,
    options: http.HttpRequestOptions = {}
  ): Promise<T> {
    const httpRequest = http.createHttp()

    const defaultOptions: http.HttpRequestOptions = {
      method: http.RequestMethod.GET,
      header: { 'Content-Type': 'application/json' },
      connectTimeout: this.TIMEOUT,
      readTimeout: this.TIMEOUT,
      ...options
    }

    try {
      const response = await httpRequest.request(
        `${this.BASE_URL}/${endpoint}`,
        defaultOptions
      )

      if (response.responseCode === 200) {
        return JSON.parse(response.result as string) as T
      } else {
        throw new NetworkError(response.responseCode,
          `HTTP ${response.responseCode}`)
      }
    } catch (err) {
      throw new NetworkError(0, `Network error: ${JSON.stringify(err)}`)
    } finally {
      httpRequest.destroy()  // ⚠️ 必须销毁!
    }
  }

  // 封装常用方法
  static async get<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: http.RequestMethod.GET })
  }

  static async post<T>(endpoint: string, body: object): Promise<T> {
    return this.request<T>(endpoint, {
      method: http.RequestMethod.POST,
      extraData: JSON.stringify(body)
    })
  }

  static async upload(endpoint: string, filePath: string): Promise<string> {
    return this.request<string>(endpoint, {
      method: http.RequestMethod.POST,
      extraData: filePath  // 文件路径
    })
  }
}

class NetworkError extends Error {
  code: number
  constructor(code: number, message: string) {
    super(message)
    this.code = code
  }
}
```

### 9.2 网络请求规范

```
✅ 所有网络请求必须:
  - 设置超时 (connectTimeout + readTimeout)
  - 错误处理 (try-catch + 用户友好提示)
  - 线程安全 (异步操作不阻塞 UI 线程)
  - 请求销毁 (httpRequest.destroy())
  - 日志记录 (hilog)

✅ 敏感数据传输:
  - 使用 HTTPS (禁止 HTTP 明文)
  - 证书校验 (caPath 指定自定义证书)
  - Token 存储使用 HUKS (密钥管理)

❌ 禁止:
  - 在主线程执行同步网络请求
  - 硬编码 API Key / Token
  - 忽略 SSL 证书错误 (except 开发环境)
```

---

## 10. 多设备适配 & 响应式布局

### 10.1 断点系统

```typescript
// 使用 BreakpointSystem 实现响应式
import { BreakpointSystem, BreakpointConstants } from '@ohos.breakpoint_system'

// 设备断点
enum WindowSize {
  SM = 0,   // 手机竖屏 (< 600vp)
  MD = 1,   // 手机横屏 / 平板竖屏 (600-840vp)
  LG = 2,   // 平板横屏 / 桌面 (≥ 840vp)
}

@Entry
@Component
struct ResponsivePage {
  @State windowSize: WindowSize = WindowSize.SM
  @StorageLink('currentBreakpoint') currentBreakpoint: string = 'sm'

  build() {
    if (this.currentBreakpoint === 'sm') {
      // 手机: 单列布局
      SingleColumnLayout()
    } else if (this.currentBreakpoint === 'md') {
      // 平板竖屏: 双列布局
      TwoColumnLayout()
    } else {
      // 平板横屏/桌面: 侧边栏+内容
      ListAndDetailLayout()
    }
  }
}
```

### 10.2 媒体查询

```typescript
// ✅ 使用 @Styles + mediaQuery
@Component
struct AdaptiveComponent {
  build() {
    Column() {
      Text('响应式文本')
    }
    .width('100%')
    .padding(16)
    // 手机横屏/平板增大内边距
    .mediaQuery({ minWidth: 600 }, () => {
      this.padding(32)
    })
  }
}
```

### 10.3 横竖屏适配

```typescript
// module.json5 声明
"abilities": [{
  "orientation": "auto_rotation"  // 自动旋转
}]

// 监听方向变化
import display from '@ohos.display'

display.on('change', (data: display.Display) => {
  const isPortrait = display.getDefaultDisplaySync().width
    < display.getDefaultDisplaySync().height
  // 根据方向调整 UI
})
```

### 10.4 资源多设备适配

```
resources/
├── base/              # 默认 (手机竖屏)
│   ├── element/
│   │   └── string.json
│   └── media/
│       └── icon.png   # 基准图标
├── dark/              # 深色模式 ← 自动切换
│   └── element/
│       └── color.json
├── tablet/            # 平板
│   └── element/
│       └── string.json
└── zh_CN/             # 中文
    └── element/
        └── string.json

// 代码中使用
Text($r('app.string.welcome_message'))  // 自动选择正确资源
Image($r('app.media.icon'))              // 自动选择密度
```

---

## 11. 安全 & 隐私规范

### 11.1 权限声明

```json5
// module.json5
"requestPermissions": [
  {
    "name": "ohos.permission.INTERNET",        // 网络 (自动授权)
    "reason": "$string:internet_reason",
    "usedScene": { "abilities": ["EntryAbility"], "when": "always" }
  },
  {
    "name": "ohos.permission.CAMERA",           // 相机 (用户授权)
    "reason": "$string:camera_reason",          // ⚠️ 必须说明理由!
    "usedScene": { "abilities": ["EntryAbility"], "when": "inuse" }
  },
  {
    "name": "ohos.permission.LOCATION",
    "reason": "$string:location_reason",
    "usedScene": { "abilities": ["EntryAbility"], "when": "inuse" }
  }
]
```

### 11.2 隐私规范

```
✅ 必须:
  - 在 privacy statement 中声明所有数据收集用途
  - 权限请求时显示 reason (告知用户为什么需要)
  - 仅请求最小必要权限
  - 敏感数据使用 HUKS (HarmonyOS Universal KeyStore) 加密存储
  - AppGallery 上架时填写隐私声明

❌ 禁止:
  - 未经用户同意收集个人数据
  - 在后台使用相机/麦克风
  - 获取设备唯一标识符用于追踪 (OAID 除外)
  - 申请与功能无关的权限
```

### 11.3 数据加密

```typescript
import huks from '@ohos.security.huks'

// HUKS: 密钥管理
async function generateKey(): Promise<void> {
  const keyAlias = 'app_master_key'
  const huksOptions: huks.HuksOptions = {
    properties: [
      { tag: huks.HuksTag.HUKS_TAG_ALGORITHM, value: huks.HuksKeyAlg.HUKS_ALG_AES },
      { tag: huks.HuksTag.HUKS_TAG_KEY_SIZE, value: huks.HuksKeySize.HUKS_AES_KEY_SIZE_256 },
      { tag: huks.HuksTag.HUKS_TAG_PURPOSE, value: huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_ENCRYPT
        | huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_DECRYPT },
      { tag: huks.HuksTag.HUKS_TAG_DIGEST, value: huks.HuksKeyDigest.HUKS_DIGEST_SHA256 },
    ],
    inData: new Uint8Array(0)
  }

  await huks.generateKeyItem(keyAlias, huksOptions)
}

// 加密存储 (使用 HUKS 密钥)
async function encryptData(plainText: string): Promise<Uint8Array> {
  // 1. 从 HUKS 获取密钥
  // 2. 使用 AES-GCM 加密
  // 3. 存储密文到 RelationalStore/Preferences
}
```

---

## 12. 性能优化规范

### 12.1 渲染性能

```
✅ 列表优化:
  - 大列表 (> 100 项) 使用 LazyForEach
  - 为 ForEach 提供唯一 keyGenerator
  - 使用 .cachedCount() 预加载

✅ 组件优化:
  - 避免在 build() 中创建新对象 (提取到 aboutToAppear)
  - 使用 @Reusable 标记可复用组件
  - 使用 @Builder 提取重复 UI 结构
  - 控制组件嵌套深度 (≤ 10 层)

✅ 图片优化:
  - 使用 .renderMode(ImageRenderMode.Template) 减少内存
  - 列表缩略图使用 .syncLoad(false) (异步加载)
  - 大图使用 ImageKnife 库或分块加载

✅ 动画优化:
  - 使用系统属性动画 (.animation()) 而非自定义逐帧
  - 避免同时执行多个动画
  - 不可见时停止动画 (.onVisibleAreaChange())
```

### 12.2 内存管理

```typescript
// ✅ 及时释放资源
aboutToDisappear(): void {
  // 取消网络请求
  this.httpRequest?.destroy()

  // 移除事件监听
  emitter.off(eventId)

  // 清空大对象
  this.largeImageData = null

  // 关闭数据库连接
  this.resultSet?.close()
}

// ✅ 使用 WeakRef 避免循环引用
class ViewModel {
  private weakRef: WeakRef<Component>

  setComponent(comp: Component): void {
    this.weakRef = new WeakRef(comp)
  }
}
```

### 12.3 启动优化

```
✅ 冷启动优化:
  - 首页延迟加载非首屏内容 (约 2 秒后)
  - aboutToAppear() 中避免同步耗时操作
  - 使用 TaskPool 并行初始化

✅ 热启动优化:
  - onForeground() 中仅刷新必要数据
  - 缓存上次页面状态 (不在 onForeground 重建 UI)
```

---

## 13. 测试规范

### 13.1 单元测试

```typescript
// src/test/TaskViewModel.test.ets
import { describe, it, expect } from '@ohos/hypium'
import { TaskViewModel } from '../viewmodel/TaskViewModel'

export default function TaskViewModelTest() {
  describe('TaskViewModel', () => {
    it('should filter active tasks', () => {
      const vm = new TaskViewModel()
      vm.tasks = [
        { id: '1', title: 'Task 1', isCompleted: false, priority: 0, createdAt: new Date() },
        { id: '2', title: 'Task 2', isCompleted: true, priority: 0, createdAt: new Date() },
      ]
      vm.selectedFilter = 'active'

      expect(vm.filteredTasks.length).assertEqual(1)
      expect(vm.filteredTasks[0].id).assertEqual('1')
    })

    it('should handle empty task list', () => {
      const vm = new TaskViewModel()
      vm.tasks = []
      expect(vm.filteredTasks.length).assertEqual(0)
    })
  })
}
```

### 13.2 UI 测试

```typescript
// 使用 UiDriver 模拟用户操作
import { Driver, Component, UiDriver } from '@ohos.UiTest'

it('should navigate to detail page on task tap', async () => {
  const driver = await UiDriver.create()
  const taskItem = await driver.findComponent(ON.text('Task 1'))
  await taskItem.click()
  const detailPage = await driver.findComponent(ON.id('detail_page'))
  expect(detailPage).assertNotNull()
})
```

---

## 14. AppGallery 上架规范

### 14.1 上架准备清单

```
═══════════════════════════════════════════════
AppGallery Connect 上架要求
═══════════════════════════════════════════════

□ 1. 应用基本信息
   - 应用名称 (2-30 字符)
   - 应用描述 (50-8000 字符)
   - 应用图标 (512×512 px, PNG)
   - 至少 3 张应用截图 (分辨率 ≥ 1080p)

□ 2. 应用分类
   - 选择正确的应用分类
   - 年龄分级

□ 3. 隐私 & 合规
   - 隐私政策 URL (必须, 可访问)
   - 权限说明 (每个敏感权限的用途)
   - 用户协议 (如适用)

□ 4. 应用包
   - 签名 (使用 AGC 的发布证书)
   - .app 包 (hvigor 构建)
   - 版本号 (versionCode + versionName)

□ 5. 测试
   - 通过 云测试 (必须, AGC 免费提供)
   - 无崩溃、无 ANR
   - 核心功能正常

□ 6. 内容审查
   - 无违法违规内容
   - 无侵犯第三方权益
   - 应用名称/图标不侵权
```

### 14.2 构建 & 签名

```bash
# DevEco Studio 或命令行构建
hvigorw assembleApp

# 构建产物
# build/outputs/default/entry-default-signed.hap   # HAP 包
# build/outputs/default/app-default-signed.app       # APP 包

# 签名配置 (build-profile.json5)
"signingConfigs": [{
  "name": "release",
  "type": "HarmonyOS",
  "material": {
    "storeFile": "/path/to/release.p12",
    "storePassword": "******",
    "keyAlias": "release",
    "keyPassword": "******",
    "signAlg": "SHA256withECDSA",
    "profile": "/path/to/release.p7b",
    "certpath": "/path/to/release.cer"
  }
}]
```

### 14.3 云测试要求

```
AGC 云测试 (免费, 必须通过):
□ 1. 兼容性测试: 至少 5 款主流设备
□ 2. 稳定性测试: Monkey 测试 ≥ 30 分钟, 无崩溃
□ 3. 性能测试: 冷启动 < 2s, 内存 < 500MB
□ 4. 安全测试: 无明文密码、无日志泄露
□ 5. 权限测试: 权限声明与使用一致
```

### 14.4 元服务 (原子化服务) 特殊要求

```
如果发布为元服务:
□ 包大小 ≤ 10MB
□ 无需安装, 即用即走
□ 核心功能在 3 步内可完成
□ 支持分享和碰一碰分发
□ 有独立的元服务图标和描述
```

---

## 15. 开发检查清单

### 代码质量

```
□ ArkTS 无 any 类型
□ 所有 ForEach 有唯一 keyGenerator
□ 大列表使用 LazyForEach
□ 网络请求使用 try-catch
□ HTTP 请求后 destroy()
□ 组件 aboutToDisappear 中清理资源
□ 无硬编码字符串 (使用 $r() 或 DesignTokens)
□ 命名符合规范 (camelCase/PascalCase/UPPER_SNAKE_CASE)
□ 无 console.log (使用 hilog)
```

### 安全 & 隐私

```
□ 所有敏感权限声明 reason
□ 最小权限原则 (不申请多余权限)
□ HTTPS 通信 (无 HTTP 明文)
□ 敏感数据使用 HUKS 加密
□ 隐私政策 URL 可访问
□ 无硬编码密钥/Token
```

### 多设备适配

```
□ 支持手机 + 平板
□ 支持横竖屏 (orientation: auto_rotation)
□ 使用 vp/fp 单位 (非 px)
□ 资源分设备密度 (base/tablet/dark)
□ 深色模式适配 (dark 资源目录)
```

### 性能

```
□ 冷启动 < 2 秒
□ 列表滚动流畅 (60 fps)
□ 无内存泄漏
□ 大图异步加载
□ 无主线程同步 IO
```

### AppGallery 上架

```
□ 应用图标 512×512 PNG
□ 至少 3 张截图 (≥ 1080p)
□ 隐私政策 URL 可访问
□ 通过 AGC 云测试
□ 应用包签名正确
□ 版权信息完整 (如含第三方代码需声明)
□ 软件著作权证书 (如有, 非强制)
```

---

## 📚 附录: iOS ↔ HarmonyOS 概念映射 (SwiftUI 开发者速查)

| iOS (SwiftUI) | HarmonyOS (ArkUI) |
|---------------|-------------------|
| `@main struct App` | `@Entry @Component struct` + EntryAbility |
| `WindowGroup` | `Navigation` / `windowStage.loadContent` |
| `TabView` | `Tabs` + `TabContent` |
| `NavigationStack` | `Navigation` + `NavPathStack` |
| `NavigationLink` | `pageStack.pushPathByName()` |
| `List` | `List` + `ListItem` |
| `ForEach` | `ForEach` / `LazyForEach` |
| `@State` | `@State` / `@Local` (V2) |
| `@Binding` | `@Link` / `@Param` (V2) |
| `@ObservedObject` | `@Observed` + `@ObjectLink` |
| `@EnvironmentObject` | `@Provide` + `@Consume` |
| `@Environment(\.colorScheme)` | `@StorageLink('currentColorMode')` |
| `Color(.systemBackground)` | `$r('sys.color.ohos_id_color_background')` |
| `Font.system(.body)` | `fontSize(16).fontWeight(FontWeight.Regular)` |
| `.sheet()` | `bindSheet($$isShow)` / `Navigation` |
| `.alert()` | `AlertDialog` |
| `.contextMenu()` | `bindContextMenu()` |
| `ProgressView()` | `LoadingProgress()` |
| `.refreshable` | `Refresh({ onRefresh })` |
| `.task {}` | `aboutToAppear()` |
| `.onDisappear {}` | `aboutToDisappear()` |
| `@AppStorage` | `AppStorage` / `@StorageLink` |
| `UserDefaults` | `Preferences` |
| `CoreData / SwiftData` | `RelationalStore` |
| `URLSession` | `@ohos.net.http` |
| `Codable / JSONEncoder` | `JSON.parse()` / `JSON.stringify()` |
| `@Environment(\.scenePhase)` | `UIAbility.onForeground/onBackground` |
| `.ignoresSafeArea()` | `.expandSafeArea()` |
| `Spacer()` | `Blank()` |
| `#Preview` | `@Preview` |
| `Xcode` | `DevEco Studio` |
| `Simulator` | `Emulator` / `Previewer` |

---

> **文档版本**: 1.0.0
> **最后更新**: 2026-06-08
> **基于**: HarmonyOS NEXT API 12+ / ArkTS / ArkUI 官方文档
> **维护**: 随华为开发者联盟文档更新而更新
