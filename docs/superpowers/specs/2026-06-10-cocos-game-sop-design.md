# Cocos Creator 游戏开发完整流程 SOP 设计文档

> 创建日期：2026-06-10
> 状态：已批准

---

## 1. 概述

### 1.1 基本信息

| 项目 | 内容 |
|------|------|
| 名称 | Cocos Creator 游戏开发完整流程 SOP |
| 引擎版本 | Cocos Creator 3.8+ |
| 语言 | TypeScript |
| 目标平台 | 微信小游戏、抖音小游戏、快手小游戏 |
| 后端方案 | 单机优先 + 可选云服务（微信云开发/字节云开发） |
| 开发周期 | 5 阶段（策划→原型→开发→测试→上线），12-18天 |

### 1.2 设计目标

- 填补现有SOP体系在游戏开发领域的空白
- 覆盖游戏开发全生命周期（策划到上线）
- 提供多平台小游戏发布能力
- 设计通用游戏模板，可裁剪扩展

---

## 2. 阶段结构

| 阶段 | 名称 | 核心产出 | 预计时长 |
|------|------|----------|----------|
| **阶段一** | 游戏策划 | 游戏设计文档（GDD）、原型草图、资源清单 | 1-2天 |
| **阶段二** | 原型开发 | 可玩原型、核心玩法验证、技术可行性验证 | 2-3天 |
| **阶段三** | 完整开发 | 完整游戏功能、UI系统、关卡/内容、云服务集成 | 5-7天 |
| **阶段四** | 测试优化 | Bug修复、性能优化、多平台适配、真机测试 | 2-3天 |
| **阶段五** | 上线发布 | 小游戏审核、上架、版本管理、运营准备 | 2-3天 |

---

## 3. 架构设计

### 3.1 项目结构

```
项目结构
├── assets/
│   ├── scenes/             # 场景文件
│   │   ├── Launch.scene    # 启动场景
│   │   ├── Main.scene      # 主场景
│   │   └── Game.scene      # 游戏场景
│   ├── scripts/            # 脚本目录
│   │   ├── core/           # 核心框架
│   │   │   ├── GameManager.ts
│   │   │   ├── AudioManager.ts
│   │   │   ├── ResourceLoader.ts
│   │   │   └── EventSystem.ts
│   │   ├── ui/             # UI组件
│   │   ├── game/           # 游戏逻辑
│   │   └── platform/       # 平台适配
│   ├── resources/          # 动态加载资源
│   ├── textures/           # 图片资源
│   ├── audio/              # 音效资源
│   └── prefabs/            # 预制体
├── settings/               # 引擎配置
├── project.json            # 项目配置
└── tsconfig.json           # TypeScript配置
```

### 3.2 核心模块设计

| 模块 | 职责 | 说明 |
|------|------|------|
| **GameManager** | 游戏生命周期管理 | 初始化、暂停、恢复、退出 |
| **AudioManager** | 音效管理 | BGM、音效播放、音量控制 |
| **ResourceLoader** | 资源加载管理 | 异步加载、进度回调、资源缓存 |
| **EventSystem** | 事件系统 | 全局事件订阅/发布、解耦模块 |
| **SaveSystem** | 存档系统 | 本地存储、云存档同步 |
| **PlatformAdapter** | 平台适配 | 微信/抖音/快手 SDK 封装 |

---

## 4. 核心组件设计

### 4.1 通用游戏模板功能模块

| 模块 | 组件/功能 | 说明 |
|------|-----------|------|
| **启动系统** | 闪屏、资源预加载、初始化流程 | Logo展示→资源加载→主菜单 |
| **UI框架** | UI管理器、弹窗系统、引导系统 | UI栈管理、动画过渡、模态弹窗 |
| **角色系统** | 角色控制、动画状态机、碰撞检测 | 可扩展的角色基类 |
| **关卡系统** | 关卡配置、关卡加载、进度管理 | 数据驱动的关卡设计 |
| **音效系统** | BGM管理、音效池、音量设置 | 支持多音效同时播放 |
| **存档系统** | 本地存档、云存档、多存档槽 | JSON序列化、加密可选 |
| **成就系统** | 成就定义、解锁条件、通知展示 | 可配置的成就数据 |
| **广告系统** | 激励视频、插屏广告、Banner广告 | 多平台广告SDK封装 |
| **内购系统** | 商品配置、购买流程、发货验证 | 小游戏内购支持 |
| **数据统计** | 自定义事件、关卡埋点、用户行为 | 接入各平台数据分析 |

### 4.2 技能标注映射

| 标签 | 游戏SOP应用场景 |
|------|-----------------|
| `[SHELL]` | Cocos Creator 项目创建、构建发布 |
| `[WRITE]` | 创建场景、脚本、配置文件 |
| `[GENERATE]` | 生成组件模板、数据模型 |
| `[DEBUG]` | 引擎调试器、真机调试、性能分析 |
| `[DIALOG]` | 策划确认、设计评审、上架配置 |
| `[REVIEW]` | 代码审查、性能检查、设计文档评审 |
| `[VALIDATE]` | 多平台构建验证、真机测试 |
| `[RESEARCH]` | Cocos文档、各平台API、游戏设计参考 |
| `[GIT]` | 版本提交 |

---

## 5. 数据流设计

### 5.1 数据流向

```
玩家操作 → 游戏逻辑 → 存档系统 → 本地存储
                              ↓
                          云存档（可选）
                              ↓
                          云开发数据库
```

### 5.2 核心数据模型

#### 玩家数据

```typescript
interface PlayerData {
  playerId: string
  nickname: string
  avatar: string
  level: number
  exp: number
  coins: number
  gems: number
  unlockLevels: number[]
  achievements: string[]
  settings: GameSettings
  createTime: number
  updateTime: number
}
```

#### 游戏设置

```typescript
interface GameSettings {
  bgmVolume: number      // 0-1
  sfxVolume: number      // 0-1
  vibration: boolean
  language: string
}
```

#### 关卡数据

```typescript
interface LevelData {
  levelId: number
  config: LevelConfig    // 关卡配置
  stars: number          // 0-3星
  bestScore: number
  playCount: number
  unlocked: boolean
}
```

#### 成就数据

```typescript
interface AchievementData {
  achievementId: string
  name: string
  description: string
  icon: string
  unlocked: boolean
  unlockTime?: number
  progress: number       // 0-1
  target: number
}
```

### 5.3 存档策略

| 场景 | 触发时机 | 存储位置 |
|------|----------|----------|
| 自动存档 | 关卡结束、获得成就、退出游戏 | 本地 + 云端（如已登录） |
| 手动存档 | 玩家点击保存按钮 | 本地 + 云端 |
| 云同步 | 登录成功、切换设备 | 云端拉取合并 |

---

## 6. 错误处理设计

### 6.1 错误分类

| 类型 | 场景 | 处理方式 |
|------|------|----------|
| **资源加载错误** | 图片/音频加载失败 | 显示占位图、静默跳过、重试机制 |
| **存档错误** | 存档损坏、写入失败 | 提示用户、尝试恢复、新建存档 |
| **网络错误** | 云同步失败、广告加载失败 | 静默重试、降级为单机模式 |
| **平台错误** | SDK调用失败、权限拒绝 | Toast提示、引导用户设置 |
| **游戏逻辑错误** | 配置缺失、边界情况 | 日志记录、安全默认值 |

### 6.2 错误处理示例

```typescript
// 全局错误捕获
director.on(Director.EVENT_ERROR, (error) => {
  console.error('[Game Error]', error)
  // 上报错误日志
  if (typeof wx !== 'undefined') {
    wx.reportEvent('game_error', { message: error.message })
  }
})

// 资源加载失败处理
resources.load(path, type, (err, asset) => {
  if (err) {
    console.warn(`资源加载失败: ${path}`)
    // 使用默认资源
    return this.getDefaultAsset(type)
  }
  return asset
})

// 存档恢复
function loadSaveData(): PlayerData {
  try {
    const data = localStorage.getItem('save')
    return JSON.parse(data)
  } catch (e) {
    console.warn('存档损坏，创建新存档')
    return createNewPlayerData()
  }
}
```

---

## 7. 测试策略设计

### 7.1 测试层级

| 层级 | 工具 | 覆盖范围 |
|------|------|----------|
| **单元测试** | Jest | 工具函数、数据模型、游戏逻辑 |
| **组件测试** | Cocos Test Framework | 组件行为、事件响应 |
| **集成测试** | 自动化测试 | 完整游戏流程、关卡通关 |
| **真机测试** | 微信开发者工具、真机 | 多平台兼容、性能验证 |

### 7.2 测试覆盖率目标

| 类型 | 目标 | 说明 |
|------|------|------|
| 核心逻辑 | ≥80% | 存档系统、关卡逻辑、数据计算 |
| 组件 | ≥60% | 核心组件的生命周期和交互 |
| 工具函数 | ≥90% | 数学计算、数据转换 |

### 7.3 测试示例

```typescript
// __tests__/save-system.test.ts
describe('SaveSystem', () => {
  beforeEach(() => {
    localStorage.clear()
  })

  it('应该正确保存玩家数据', () => {
    const saveSystem = new SaveSystem()
    const data = createMockPlayerData()

    saveSystem.save(data)
    const loaded = saveSystem.load()

    expect(loaded.playerId).toBe(data.playerId)
    expect(loaded.coins).toBe(data.coins)
  })

  it('存档损坏时应该返回新存档', () => {
    localStorage.setItem('save', 'invalid json')

    const saveSystem = new SaveSystem()
    const data = saveSystem.load()

    expect(data).toBeDefined()
    expect(data.playerId).toBeDefined()
  })
})

// __tests__/level-manager.test.ts
describe('LevelManager', () => {
  it('应该正确计算星级', () => {
    const manager = new LevelManager()

    expect(manager.calculateStars(100, 150)).toBe(1)
    expect(manager.calculateStars(120, 150)).toBe(2)
    expect(manager.calculateStars(150, 150)).toBe(3)
  })
})
```

---

## 8. 阶段详细设计

### 8.1 阶段一：游戏策划（1-2天）

| 步骤 | 名称 | 主要内容 | 技能标注 | 产出物 |
|------|------|----------|----------|--------|
| 1.1 | 游戏概念 | 确定游戏类型、核心玩法、目标用户 | `[DIALOG]` | 游戏概念文档 |
| 1.2 | 玩法设计 | 核心循环、操作方式、关卡设计 | `[WRITE][RESEARCH]` | 玩法设计文档 |
| 1.3 | 原型草图 | UI布局、场景结构、交互流程 | `[WRITE]` | 原型草图 |
| 1.4 | 资源清单 | 美术资源、音效资源、配置数据 | `[WRITE]` | 资源清单 |
| 1.5 | 技术方案 | 架构设计、技术选型、风险分析 | `[REVIEW]` | 技术方案文档 |

### 8.2 阶段二：原型开发（2-3天）

| 步骤 | 名称 | 主要内容 | 技能标注 | 产出物 |
|------|------|----------|----------|--------|
| 2.1 | 项目初始化 | 创建项目、配置环境、目录结构 | `[SHELL][WRITE]` | 项目骨架 |
| 2.2 | 核心框架 | GameManager、EventSystem、ResourceLoader | `[GENERATE]` | 核心框架 |
| 2.3 | 场景搭建 | 启动场景、主菜单、游戏场景 | `[WRITE]` | 基础场景 |
| 2.4 | 核心玩法 | 实现核心玩法原型（可玩） | `[GENERATE][DEBUG]` | 可玩原型 |
| 2.5 | 原型验证 | 玩法测试、调整参数 | `[DEBUG][DIALOG]` | 验证报告 |

### 8.3 阶段三：完整开发（5-7天）

| 步骤 | 名称 | 主要内容 | 技能标注 | 产出物 |
|------|------|----------|----------|--------|
| 3.1 | UI系统 | UI管理器、弹窗、动画 | `[GENERATE]` | UI框架 |
| 3.2 | 角色系统 | 角色控制、动画、碰撞 | `[GENERATE]` | 角色模块 |
| 3.3 | 关卡系统 | 关卡配置、加载、进度 | `[GENERATE][WRITE]` | 关卡模块 |
| 3.4 | 音效系统 | BGM、音效、音量控制 | `[GENERATE]` | 音效模块 |
| 3.5 | 存档系统 | 本地存档、云存档 | `[GENERATE]` | 存档模块 |
| 3.6 | 成就系统 | 成就配置、解锁、通知 | `[GENERATE]` | 成就模块 |
| 3.7 | 广告系统 | 激励视频、插屏、Banner | `[GENERATE][DIALOG]` | 广告模块 |
| 3.8 | 云服务集成 | 云存档、排行榜、数据分析 | `[GENERATE]` | 云服务模块 |
| 3.9 | 平台适配 | 微信/抖音/快手 SDK 封装 | `[WRITE][VALIDATE]` | 平台适配层 |

### 8.4 阶段四：测试优化（2-3天）

| 步骤 | 名称 | 主要内容 | 技能标注 | 产出物 |
|------|------|----------|----------|--------|
| 4.1 | 单元测试 | 核心逻辑测试、覆盖率 | `[VALIDATE]` | 测试报告 |
| 4.2 | 集成测试 | 完整流程测试、边界情况 | `[DEBUG]` | 测试用例 |
| 4.3 | 性能优化 | 帧率优化、内存优化、包体优化 | `[DEBUG][REVIEW]` | 性能报告 |
| 4.4 | 多平台测试 | 微信/抖音/快手真机测试 | `[VALIDATE]` | 兼容性报告 |
| 4.5 | Bug修复 | 问题定位、修复、回归测试 | `[DEBUG]` | Bug清单 |

### 8.5 阶段五：上线发布（2-3天）

| 步骤 | 名称 | 主要内容 | 技能标注 | 产出物 |
|------|------|----------|----------|--------|
| 5.1 | 构建发布 | 多平台构建、包体生成 | `[SHELL]` | 发布包 |
| 5.2 | 小游戏审核 | 微信/抖音/快手提交审核 | `[DIALOG][VALIDATE]` | 审核提交 |
| 5.3 | 版本管理 | 版本号、更新日志、灰度发布 | `[GIT][WRITE]` | 版本记录 |
| 5.4 | 运营准备 | 数据埋点、客服配置、运营素材 | `[WRITE]` | 运营资料 |
| 5.5 | 正式上线 | 发布公告、监控告警、用户反馈 | `[REVIEW]` | 上线完成 |

---

## 9. 与现有SOP的关系

```
Cocos Creator 游戏SOP (独立)
├── 可复用: Git工作流、测试策略
├── 可引用: 小程序SOP（云开发部分）
└── 独立模块: 游戏引擎开发、小游戏审核、游戏运营
```

---

## 10. 后续工作

- [ ] 创建实现计划（writing-plans）
- [ ] 编写 Cocos Creator 游戏SOP文档
- [ ] 创建通用游戏模板项目
- [ ] 编写 Unity SOP 设计文档
