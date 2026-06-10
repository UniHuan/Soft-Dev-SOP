# Cocos Creator 游戏开发完整流程 SOP 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 编写完整的 Cocos Creator 游戏开发 SOP 文档，覆盖从策划到上线的5个阶段

**Architecture:** 采用5阶段结构（策划→原型→开发→测试→上线），每个阶段包含多个步骤，提供完整的游戏开发流程指南

**Tech Stack:** Cocos Creator 3.8+ + TypeScript + 小游戏平台 SDK

---

## 文件结构

```
game/
├── CocosCreator_Game_Development_SOP.md  # 主文档（约6000+行）
├── COCOS_CREATOR_GUIDE.md                 # Cocos Creator 开发指南
└── GAME_DESIGN_TEMPLATES.md               # 游戏设计文档模板
```

---

## Task 1: 创建 SOP 文档框架和元信息

**Files:**
- Create: `game/CocosCreator_Game_Development_SOP.md`

- [ ] **Step 1.1: 创建文档头部和元信息**

```markdown
# 🎮 Claude Code + Cocos Creator 游戏开发完整流程 SOP

> **适用对象**: Claude Code (AI Agent) 全自动执行
> **目标**: 从游戏策划到小游戏上架，完整流程覆盖
> **技术栈**: Cocos Creator 3.8+ + TypeScript
> **目标平台**: 微信小游戏、抖音小游戏、快手小游戏
> **开发周期**: 5 阶段，12-18 天
> **最低要求**: Node.js 18+, Cocos Creator 3.8+, 微信开发者工具

---

## 🧠 Claude Code 技能调用矩阵

| 技能标识 | 技能名称 | 说明 | Claude Code 工具 |
|---------|---------|------|-----------------|
| `[SHELL]` | Shell 执行 | 执行 bash 命令、Cocos CLI、构建脚本 | `Bash` |
| `[WRITE]` | 文件写入 | 创建/修改 .ts .scene .prefab 文件 | `Edit` / `Write` |
| `[READ]` | 文件读取 | 读取现有代码、场景、配置 | `Read` |
| `[DIALOG]` | 用户交互 | 向用户提问、请求确认、策划讨论 | 对话输出 |
| `[GENERATE]` | 代码生成 | 生成游戏组件、系统、配置 | `Edit` / `Write` |
| `[REVIEW]` | 代码审查 | 审查游戏代码、性能、架构设计 | `Read` → 分析 |
| `[DEBUG]` | 调试分析 | 分析运行时错误、性能问题并修复 | `Read` + `Bash` + `Edit` |
| `[RESEARCH]` | 知识检索 | 查阅 Cocos 文档、游戏设计最佳实践 | `Read` 参考文档 |
| `[VALIDATE]` | 验证检查 | 对照清单逐项验证 | `Bash` + `Read` + 分析 |
| `[GIT]` | 版本控制 | git add/commit/tag/push | `Bash` |

### 技能调用原则

\`\`\`
1. 每个阶段开始前 → [RESEARCH] 查阅本 SOP 的设计规范和最佳实践
2. 每次生成代码后 → [REVIEW] 检查代码质量和性能影响
3. 每次构建后 → [DEBUG] 分析输出、修复问题
4. 每个阶段结束前 → [VALIDATE] 对照验收条件确认
5. 每个阶段结束后 → [GIT] 提交代码
6. 策划讨论 → [DIALOG] 与用户确认设计方案
\`\`\`
```

- [ ] **Step 1.2: 添加总览时间线**

```markdown
---

## 📋 总览时间线

\`\`\`
阶段一：游戏策划 (1-2天)
├─ [0.0h] 游戏概念确定
├─ [0.5h] 玩法设计文档
├─ [1.0h] 原型草图绘制
├─ [1.5h] 资源清单整理
└─ [2.0h] 技术方案评审

阶段二：原型开发 (2-3天)
├─ [0.0h] 项目初始化
├─ [0.5h] 核心框架搭建
├─ [1.0h] 场景搭建
├─ [1.5h] 核心玩法实现
└─ [2.5h] 原型验证测试

阶段三：完整开发 (5-7天)
├─ [0.0h] UI系统开发
├─ [1.0h] 角色系统开发
├─ [2.0h] 关卡系统开发
├─ [3.0h] 音效系统开发
├─ [4.0h] 存档系统开发
├─ [5.0h] 成就系统开发
├─ [5.5h] 广告系统开发
├─ [6.0h] 云服务集成
└─ [7.0h] 平台适配

阶段四：测试优化 (2-3天)
├─ [0.0h] 单元测试
├─ [0.5h] 集成测试
├─ [1.0h] 性能优化
├─ [2.0h] 多平台测试
└─ [2.5h] Bug修复

阶段五：上线发布 (2-3天)
├─ [0.0h] 构建发布
├─ [0.5h] 小游戏审核
├─ [1.0h] 版本管理
├─ [2.0h] 运营准备
└─ [2.5h] 正式上线
\`\`\`
```

- [ ] **Step 1.3: 添加执行前提检查**

```markdown
---

## ⚠️ 执行前提 (Claude Code 自动检查)

\`\`\`bash
# [SHELL] 1. 确认 Node.js 版本 >= 18.0
NODE_VERSION=$(node -v | sed 's/v//' | cut -d. -f1)
if [ "$NODE_VERSION" -lt 18 ]; then
    echo "❌ 需要 Node.js 18.0+"
    exit 1
fi

# [SHELL] 2. 确认 Cocos Creator 已安装
if [ ! -d "/Applications/CocosCreator.app" ]; then
    echo "⚠️ Cocos Creator 未安装在默认位置"
    echo "   请手动指定 Cocos Creator 路径"
fi

# [SHELL] 3. 检查微信开发者工具
WX_CLI="/Applications/wechatwebdevtools.app/Contents/MacOS/cli"
if [ -f "$WX_CLI" ]; then
    echo "✅ 微信开发者工具 CLI 可用"
fi

# [SHELL] 4. 保存环境变量
cat > /tmp/sop_cocos.env << 'EOF'
PROJECT_NAME="MyGame"
PROJECT_DIR="/Users/xurui/Projects/SOP/MyGame"
COCOS_VERSION="3.8"
EOF
\`\`\`
```

- [ ] **Step 1.4: 提交框架代码**

```bash
git add game/CocosCreator_Game_Development_SOP.md
git commit -m "docs: add Cocos Creator game SOP framework

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 2: 编写阶段一：游戏策划

**Files:**
- Modify: `game/CocosCreator_Game_Development_SOP.md`

- [ ] **Step 2.1: 编写步骤 1.1-1.5**

添加游戏策划阶段内容，包含：
- 游戏概念确定 (DIALOG)
- 玩法设计文档 (WRITE + RESEARCH)
- 原型草图绘制 (WRITE)
- 资源清单整理 (WRITE)
- 技术方案评审 (REVIEW)

每个步骤提供完整的模板和指导。

- [ ] **Step 2.2: 提交阶段一文档**

```bash
git add game/CocosCreator_Game_Development_SOP.md
git commit -m "docs: add Cocos Creator SOP Stage 1 - Game Design

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 3: 编写阶段二：原型开发

**Files:**
- Modify: `game/CocosCreator_Game_Development_SOP.md`

- [ ] **Step 3.1: 编写步骤 2.1 项目初始化**

```markdown
---

# 阶段二：原型开发

## Step 2.1 — 项目初始化

> **[SHELL]** + **[WRITE]** 创建 Cocos Creator 项目

### 创建项目

\`\`\`bash
# [SHELL] 使用 Cocos Creator CLI 创建项目
# 注意：需要先配置 Cocos Creator 路径
COCOS_PATH="/Applications/CocosCreator.app/Contents/MacOS/CocosCreator"
PROJECT_NAME="MyGame"
PROJECT_DIR="/Users/xurui/Projects/SOP"

$COCOS_PATH --project $PROJECT_DIR/$PROJECT_NAME --create

# [SHELL] 创建目录结构
cd $PROJECT_DIR/$PROJECT_NAME
mkdir -p assets/scripts/core
mkdir -p assets/scripts/ui
mkdir -p assets/scripts/game
mkdir -p assets/scripts/platform
mkdir -p assets/scenes
mkdir -p assets/prefabs
mkdir -p assets/resources
mkdir -p assets/textures
mkdir -p assets/audio
\`\`\`

### 配置 TypeScript

> **[WRITE]** 创建 tsconfig.json

\`\`\`json
{
  "compilerOptions": {
    "target": "ES2015",
    "module": "ESNext",
    "strict": true,
    "skipLibCheck": true,
    "importHelpers": false,
    "noImplicitAny": false,
    "moduleResolution": "node",
    "experimentalDecorators": true,
    "forceConsistentCasingInFileNames": true,
    "allowSyntheticDefaultImports": true,
    "declaration": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "outDir": "./temp/declarations",
    "baseUrl": ".",
    "paths": {
      "cc": ["./node_modules/cc"],
      "cc/env": ["./node_modules/cc/env"]
    }
  },
  "include": [
    "assets/**/*.ts"
  ],
  "exclude": [
    "node_modules",
    "library",
    "local",
    "temp",
    "build"
  ]
}
\`\`\`
```

- [ ] **Step 3.2: 编写步骤 2.2-2.5**

添加原型开发剩余步骤：
- 核心框架搭建 (GameManager, EventSystem, ResourceLoader)
- 场景搭建 (Launch, Main, Game)
- 核心玩法实现
- 原型验证测试

- [ ] **Step 3.3: 提交阶段二文档**

```bash
git add game/CocosCreator_Game_Development_SOP.md
git commit -m "docs: add Cocos Creator SOP Stage 2 - Prototype Development

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 4: 编写阶段三：完整开发

**Files:**
- Modify: `game/CocosCreator_Game_Development_SOP.md`

- [ ] **Step 4.1: 编写步骤 3.1-3.4**

添加核心系统开发：
- UI系统 (UIManager, 弹窗, 动画)
- 角色系统 (角色控制, 动画状态机, 碰撞检测)
- 关卡系统 (关卡配置, 加载, 进度管理)
- 音效系统 (BGM, 音效, 音量控制)

- [ ] **Step 4.2: 编写步骤 3.5-3.9**

添加辅助系统开发：
- 存档系统 (本地存档, 云存档)
- 成就系统 (成就配置, 解锁, 通知)
- 广告系统 (激励视频, 插屏, Banner)
- 云服务集成 (云存档, 排行榜, 数据分析)
- 平台适配 (微信/抖音/快手 SDK 封装)

- [ ] **Step 4.3: 提交阶段三文档**

```bash
git add game/CocosCreator_Game_Development_SOP.md
git commit -m "docs: add Cocos Creator SOP Stage 3 - Full Development

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 5: 编写阶段四和五：测试优化与上线发布

**Files:**
- Modify: `game/CocosCreator_Game_Development_SOP.md`

- [ ] **Step 5.1: 编写阶段四：测试优化**

添加测试优化内容：
- 单元测试配置与执行
- 集成测试流程
- 性能优化指南 (帧率、内存、包体)
- 多平台测试 (微信/抖音/快手)
- Bug修复流程

- [ ] **Step 5.2: 编写阶段五：上线发布**

添加上线发布内容：
- 构建发布流程
- 小游戏审核指南 (微信/抖音/快手)
- 版本管理规范
- 运营准备清单
- 正式上线流程

- [ ] **Step 5.3: 提交阶段四和五文档**

```bash
git add game/CocosCreator_Game_Development_SOP.md
git commit -m "docs: add Cocos Creator SOP Stage 4-5

- Stage 4: Testing and Optimization
- Stage 5: Release and Publishing

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 6: 创建参考文档

**Files:**
- Create: `game/COCOS_CREATOR_GUIDE.md`
- Create: `game/GAME_DESIGN_TEMPLATES.md`

- [ ] **Step 6.1: 创建 Cocos Creator 开发指南**

```markdown
# Cocos Creator 开发指南

## 组件生命周期

\`\`\`typescript
import { _decorator, Component } from 'cc'
const { ccclass, property } = _decorator

@ccclass('MyComponent')
export class MyComponent extends Component {
    onLoad() {
        // 组件加载时调用
    }

    start() {
        // 组件第一次激活时调用
    }

    update(dt: number) {
        // 每帧调用
    }

    onDestroy() {
        // 组件销毁时调用
    }
}
\`\`\`

## 资源加载

\`\`\`typescript
import { resources, SpriteFrame } from 'cc'

// 加载单个资源
resources.load('textures/hero', SpriteFrame, (err, spriteFrame) => {
    if (err) {
        console.error(err)
        return
    }
    // 使用 spriteFrame
})

// 加载文件夹下所有资源
resources.loadDir('textures', SpriteFrame, (err, assets) => {
    // assets 是 SpriteFrame 数组
})
\`\`\`

## 场景管理

\`\`\`typescript
import { director } from 'cc'

// 切换场景
director.loadScene('GameScene')

// 预加载场景
director.preloadScene('GameScene', () => {
    console.log('场景预加载完成')
})
\`\`\`

## 平台适配

\`\`\`typescript
// 判断当前平台
if (sys.platform === sys.Platform.WECHAT_GAME) {
    // 微信小游戏
    wx.login()
} else if (sys.platform === sys.Platform.BYTEDANCE_MINI_GAME) {
    // 抖音小游戏
    tt.login()
}
\`\`\`
```

- [ ] **Step 6.2: 创建游戏设计文档模板**

```markdown
# 游戏设计文档模板

## 1. 游戏概述

- 游戏名称：
- 游戏类型：
- 目标平台：
- 目标用户：

## 2. 核心玩法

### 2.1 游戏循环

玩家 → 操作 → 反馈 → 奖励 → 玩家

### 2.2 操作方式

- 触摸
- 重力感应
- 虚拟摇杆

## 3. 关卡设计

| 关卡 | 目标 | 难度 | 奖励 |
|------|------|------|------|
| 1-1 | 教学关卡 | ★ | 100金币 |
| 1-2 | 基础挑战 | ★ | 150金币 |

## 4. 经济系统

- 金币：通过关卡获得
- 钻石：充值获得
- 能量：时间恢复

## 5. 变现方式

- 激励视频广告
- 内购道具
- 订阅会员
```

- [ ] **Step 6.3: 最终提交**

```bash
git add .
git commit -m "docs: complete Cocos Creator game SOP documentation

- Add reference guides
- Add game design templates
- 5 Stages, complete documentation

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Task 7: 更新 README 和项目文档

**Files:**
- Modify: `README.md`

- [ ] **Step 7.1: 更新 README 添加游戏开发 SOP**

在 README.md 中添加游戏开发部分。

- [ ] **Step 7.2: 提交 README 更新**

```bash
git add README.md
git commit -m "docs: add Cocos Creator game SOP to README

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Self-Review Checklist

- [ ] **Spec Coverage**: 所有5个阶段都已实现
- [ ] **Placeholder Scan**: 无占位符
- [ ] **Type Consistency**: TypeScript 接口在各阶段保持一致
- [ ] **Code Completeness**: 所有代码步骤都有完整代码
- [ ] **Command Accuracy**: 所有 bash 命令可直接执行
