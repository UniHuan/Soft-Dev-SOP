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

## 完整 Phase 模板

```markdown
## Phase N: 阶段名称 (Day X, H:MM-H:MM)

> **[SKILL1]** + **[SKILL2]** 该 Phase 的核心任务一句话描述

### Step N.1 — 步骤标题

> **[SKILL]** 该步骤的执行指引

**具体操作**:

```bash
# [SHELL] 命令说明
cd ${PROJECT_DIR}
./hvigorw assembleHap 2>&1 | tail -10
```

```typescript
// [WRITE] 文件路径/文件名.ets
// 完整的、可编译的代码
export class Example { }
```

```bash
# [SHELL] Phase N 编译验证
source /tmp/sop_harmony.env 2>/dev/null && cd ${PROJECT_DIR}
./hvigorw assembleHap 2>&1 | tail -20
# [DEBUG] 常见错误: xxx / xxx
# [GIT]
git add -A && git commit -m "Phase N: description of changes"
```
```

## 项目覆盖度

| 平台 | SOP 行数 | 规范文档 | Phase 数 | 技能标注 |
|------|---------|---------|---------|---------|
| iOS | 5,410 | HIG KB (493 行) | 16 | ✅ |
| HarmonyOS | 2,544 | Dev Guide (1,883 行) | 13 | ✅ |
| 软著 | 1,269 | — | 8 | ✅ |
| 流水线 | 748 | — | — | ✅ |

**新增 SOP Checklist**:
```
□ 1. 创建 SOP 骨架 (Phase 0 → Phase N)
□ 2. 每个 Phase 写入 [SKILL] 标注
□ 3. 每个 Phase 添加编译验证 bash block
□ 4. 添加 [DEBUG] 覆盖常见错误
□ 5. 添加附录 (命令速查/错误速查/依赖图)
□ 6. 小白可执行验证 (零基础模拟)
□ 7. 更新 CLAUDE.md 平台规则
□ 8. 更新 README.md 导航
□ 9. 更新 CHANGELOG.md
□ 10. 更新 pipeline/ (如涉及交叉复用)
```
