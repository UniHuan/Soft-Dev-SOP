# Contributing to SOP Hub

## SOP 编写规范

### Phase 结构
```markdown
## Phase N: 标题 (时间)

> **[SKILL1]** + **[SKILL2]** 一句话描述

### Step N.1 — 标题

> **[SKILL]** 执行指引

[具体命令或代码]

> **验收条件**或 **[GIT] commit 命令**
```

### 技能标注
每个 Phase/Step 必须标注至少 1 个技能标签：
- `[SHELL]` — bash 命令
- `[WRITE]` — 文件创建
- `[GENERATE]` — 代码生成
- `[DIALOG]` — 用户确认
- `[DEBUG]` — 编译修复
- `[VALIDATE]` — 清单验证
- `[RESEARCH]` — 查阅参考文档
- `[GIT]` — 版本控制

### 编译验证
- 每个 Phase 结束必须有编译验证步骤
- iOS: `xcodebuild build` → 检查输出 → `[DEBUG]`
- 鸿蒙: `./hvigorw assembleHap` → 检查输出 → `[DEBUG]`

### 审查流程
1. 自查: 逐 Phase 模拟执行，确认无编译错误
2. 小白测试: 能否让零基础用户按步骤完成
3. Cross-ref: 确保交叉引用的文件路径正确
4. 技能密度: 每个 Step 至少 1 个 `[SKILL]`

## 提交规范
- 格式: `[类型] 简短描述`
- 类型: `Add` `Fix` `Review` `Doc` `Refactor`

## 审核清单
```
□ Phase 头部有 > [SKILL] 标注
□ 编译验证 bash 命令正确 (./hvigorw, xcodebuild)
□ [DIALOG] 用户确认点合理
□ [DEBUG] 覆盖常见编译错误
□ 引用路径正确 (相对路径)
□ README.md 行数统计已更新
```
