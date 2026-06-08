# 📋 [平台/类型] SOP 模板

> **适用对象**: Claude Code (AI Agent) + 用户协作
> **目标**: [一句话描述此 SOP 的目标]
> **适用版本**: [平台/工具版本]
> **参考规范**: [关联的规范文档]

---

## 🧠 Claude Code 技能调用矩阵

| 技能标识 | 技能名称 | 说明 | Claude Code 工具 |
|---------|---------|------|-----------------|
| `[SHELL]` | Shell 执行 | 执行 bash 命令 | `Bash` |
| `[WRITE]` | 文件写入 | 创建/修改文件 | `Edit` |
| `[READ]` | 文件读取 | 读取现有代码/日志/配置 | `Read` |
| `[DIALOG]` | 用户交互 | 向用户提问/确认 | 对话输出 |
| `[GENERATE]` | 代码生成 | 生成代码 | `Edit` |
| `[REVIEW]` | 代码审查 | 审查生成代码质量 | `Read` → 分析 |
| `[DEBUG]` | 调试分析 | 分析错误并修复 | `Read` + `Bash` + `Edit` |
| `[VALIDATE]` | 验证检查 | 对照清单逐项验证 | `Bash` + `Read` |
| `[RESEARCH]` | 知识检索 | 查阅规范文档 | `Read` |
| `[GIT]` | 版本控制 | git add/commit/tag | `Bash` |

### 技能调用原则

```
1. 每个 Phase 开始前 → [RESEARCH] 查阅关联规范文档
2. 每次生成代码后 → [REVIEW] 自检代码质量
3. 每次构建后 → [DEBUG] 自动分析修复、不抛错误给用户
4. 每个 Phase 结束前 → [VALIDATE] 对照验收条件逐项确认
5. 每个 Phase 结束后 → [GIT] 提交代码
6. 涉及不可逆操作 → [DIALOG] 必须获得用户确认
```

### 全局编译验证模式

```bash
# [SHELL] 标准编译验证块
source /tmp/sop_project.env 2>/dev/null
cd ${PROJECT_DIR}
echo "=== 编译验证 ==="
[构建命令] 2>&1 | tail -20
BUILD_EXIT=$?
if [ $BUILD_EXIT -ne 0 ]; then
  echo "❌ 编译失败! [DEBUG] 自动分析错误日志..."
  [构建命令] 2>&1 | grep -E "ERROR|:[0-9]+"
else
  echo "✅ 编译通过"
fi
```

---

## 📋 总览时间线

```
Day 1 (8h)                        Day 2 (8h)
├─ [0.0h] 环境初始化              ├─ [0.0h] 阶段 N
├─ [0.5h] 阶段 1                  ├─ [2.0h] 阶段 N+1
├─ ...                            └─ [8.0h] 最终归档
└─ [7.5h] 自测 & 收尾
```

---

## ⚠️ 执行前提 (Claude Code 自动检查)

```bash
# [SHELL] 1. 确认工具链
which [required_tool] || { echo "❌ 未安装"; exit 1; }

# [SHELL] 2. 确认账号
echo "请确认已在 [平台] 注册开发者账号"
# [DIALOG] 等待用户确认 "已确认"
```

---

## Phase 0: 环境初始化

> **[SHELL]** + **[DIALOG]** 环境检查 → 创建项目 → 初始化 Git

### Step 0.1 — 项目创建

```bash
# [DIALOG] 确认项目信息
PROJECT_NAME="MyProject"

# 持久化变量
cat > /tmp/sop_project.env << ENVEOF
PROJECT_NAME="$PROJECT_NAME"
PROJECT_DIR="$(pwd)/$PROJECT_NAME"
ENVEOF
```

### Step 0.2 — 验证 & 构建

```bash
source /tmp/sop_project.env 2>/dev/null && cd ${PROJECT_DIR}
# [SHELL] 首次构建验证
[构建命令]
# [DEBUG] 如有错误, 自动修复
```

### Step 0.3 — Git 初始化

```bash
cd ${PROJECT_DIR}
git init && git checkout -b main
cat > .gitignore << 'EOF'
/build/
.DS_Store
EOF
# [GIT]
git add -A && git commit -m "Initial project structure"
```

> **Phase 0 验收**: 项目文件存在 ✅ | Git 已提交 ✅ | 首次构建通过 ✅

---

## Phase 1: [下一阶段]

> **[SKILL]** 描述

### Step 1.1 — [步骤名]

```bash
# [SHELL] Phase 1 编译验证
source /tmp/sop_project.env 2>/dev/null && cd ${PROJECT_DIR}
[构建命令] 2>&1 | tail -20
# [DEBUG] 常见错误: xxx
# [GIT]
git add -A && git commit -m "Phase 1: xxx"
```

---

## 📚 附录

### A. 命令速查
```bash
# [常用命令]
```

### B. 常见错误速查
| 错误 | 原因 | 修复 |
|------|------|------|
| | | |

### C. 文件依赖关系
```
Phase 0 → Phase 1 → Phase 2 → ...
```

---

> **SOP 版本**: 1.0.0 | **最后更新**: yyyy-mm-dd
> **关联文档**: [相关规范文档路径]
> **技术栈**: [技术栈描述]
