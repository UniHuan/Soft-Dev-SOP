# 🎭 Agency Agents — 精华提炼与实战指南

> 深度学习自 [msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents)
> **120,000+ Stars | 232 个专家 Agent | 16 个部门 | MIT 开源**
> 提炼日期: 2026-06-30

---

## 📖 一、核心理念：为什么这个项目火了

### 1.1 问题洞察

传统 AI prompt 的三大痛点：

| 痛点 | 表现 | Agency Agents 的解法 |
|------|------|---------------------|
| **泛化无能** | "Act as a developer" 太宽泛，产出平庸 | 每个 Agent 深度专业化，有具体领域知识 |
| **无个性** | 模板化输出，读起来像 AI 写的 | 每个 Agent 有独特的性格、语气、沟通风格 |
| **无流程** | 一次性 prompt，无法复现高质量结果 | 每个 Agent 有固定的工作流程 + 交付物模板 |

### 1.2 设计哲学五原则

```
1. 🎭 强人格 (Strong Personality)
   不是通用模板，而是有真实性格和声音的角色
   反例: "你是一个前端开发者"
   正例: "你是 Frontend Developer，细节控、性能痴、像素完美主义者"

2. 📋 清晰交付物 (Clear Deliverables)
   具体产出，不是模糊指导
   反例: "写出好的代码"
   正例: "交付: 组件库 + 设计系统 + 单元测试 + 性能报告"

3. ✅ 可度量成功标准 (Measurable Success Metrics)
   数字化的质量门槛
   反例: "确保代码质量好"
   正例: "Lighthouse ≥ 90, LCP < 2.5s, 单元测试覆盖率 ≥ 85%"

4. 🔄 经过验证的工作流 (Proven Workflows)
   分步骤、可复现的执行流程
   每个 Agent 都有 Step 1 → Step 2 → Step 3 → Step 4 的固定工序

5. 💡 持续学习记忆 (Learning Memory)
   每次执行后积累模式识别能力
   "记住成功的 UI 模式、性能优化技巧、无障碍最佳实践"
```

### 1.3 与传统方案的对比

| 维度 | 传统 Prompt | Prompt 库 | 此项目 |
|------|-----------|----------|--------|
| 专业深度 | 浅 | 中 | 深（领域专家级） |
| 可复用性 | 低 | 中 | 高（标准化工作流） |
| 个性 | 无 | 无 | 强（角色驱动） |
| 交付标准 | 无 | 无 | 有（量化指标） |
| 多人协作 | 不支持 | 不支持 | 支持（Agent 间协作） |

---

## 🏢 二、16 大部门架构全景

```
The Agency
├── 🎓 Academic (学术)        5 agents  — 人类学/地理学/历史学/叙事学/心理学
├── 🎨 Design (设计)          9 agents  — UI/UX/品牌/视觉/可访问性/趣味注入
├── 💻 Engineering (工程)    34 agents  — 前端/后端/AI/DevOps/安全/合约/嵌入式...
├── 💰 Finance (财务)         5 agents  — 记账/分析师/FP&A/投资/税务
├── 🎮 Game Dev (游戏开发)   19 agents  — Unity/Unreal/Godot/Blender/Roblox
├── 🌍 GIS (地理信息)        10 agents  — 地图/空间数据/遥感/数字孪生/BIM
├── 📢 Marketing (营销)      36 agents  — 内容/SEO/社交媒体/短视频/跨境电商/知乎/小红书...
├── 💸 Paid Media (付费媒体)  7 agents  — PPC/程序化/搜索分析/创意策略/追踪
├── 📊 Product (产品)         5 agents  — 产品经理/Sprint/趋势/反馈/行为助推
├── 📋 Project Mgmt (项目管理) 7 agents — 制片人/项目牧羊/会议记录/实验追踪
├── 💼 Sales (销售)           9 agents  — 外拓/发现/交易/账户/提案/Salesforce
├── 🔒 Security (安全)       10 agents  — 渗透测试/合规/云安全/事件响应/威胁情报
├── 🥽 Spatial (空间计算)     6 agents  — XR/WebXR/VisionOS/Cockpit/macOS Metal
├── ⚡ Specialized (专业)    55 agents  — 法务/HR/医疗/政府/供应链/培训/客服...
├── 🛟 Support (支持)         N agents  — 客服/分析/响应
└── 🧪 Testing (测试)         8 agents  — 现实检查/证据收集/性能/无障碍/工具评估
```

### 关键统计

- **232 个 Agent**，覆盖软件开发生命周期 + 企业运营全链条
- **10,000+ 行**性格描述、流程定义、代码示例
- **14 种工具集成**：Claude Code / Copilot / Cursor / Gemini CLI / Aider / Windsurf / Codex 等

---

## 🧬 三、Agent 设计解剖学（最重要的精华）

### 3.1 标准 Agent 结构模板

每个 Agent 文件遵循统一的 8 段式结构：

```markdown
---
name: Agent 显示名称
description: 一句话专业描述
color: 品牌色 (cyan/blue/green/purple/red/orange/teal/pink/yellow)
emoji: 代表 emoji
vibe: 一句体现性格的标语
tools: 推荐工具列表（可选，如 WebFetch, WebSearch, Read, Write, Edit）
---

# Agent 名称 — 角色副标题

## 🧠 你的身份与记忆 (Identity & Memory)
- Role: 角色定位
- Personality: 性格特征（3-4 个形容词）
- Memory: 记住了什么（领域模式、成功经验、失败教训）
- Experience: 经历过什么（正反案例）

## 🎯 你的核心使命 (Core Mission)
- 主要职责 1：具体化描述
- 主要职责 2：具体化描述
- 主要职责 3：具体化描述
- 默认要求：非功能性默认标准

## 🚨 必须遵守的铁律 (Critical Rules)
- 硬性规则 1：不可协商的原则
- 硬性规则 2：行业标准/法规要求
- 硬性规则 3：方法论约束

## 📋 技术交付物 (Technical Deliverables)
- 具体的代码示例（真实可运行的代码）
- 架构图/流程图模板
- 报告模板
- 检查清单

## 🔄 工作流 (Workflow Process)
Step 1: [阶段名] — [具体做什么]
Step 2: [阶段名] — [具体做什么]
Step 3: [阶段名] — [具体做什么]
Step 4: [阶段名] — [具体做什么]

## 📋 交付模板 (Deliverable Template)
```markdown
# [项目名] 交付物
## 模块1
**决策**: [具体选择及理由]
## 模块2
**指标**: [量化结果]
...
```

## 💭 沟通风格 (Communication Style)
- 口头禅/语气示例
- 如何在评论中表达建议
- 如何在冲突中坚持专业立场

## 🎯 成功度量 (Success Metrics)
- 量化指标 1: 具体数值目标
- 量化指标 2: 具体数值目标
- 量化指标 3: 具体数值目标
```

### 3.2 Agent 设计的"性格魔法"

观察真实 Agent 的性格设定，发现一个规律：**最好的 Agent 性格总是带一点"极端"**。

| Agent | 性格设定 | 为什么有效 |
|-------|---------|-----------|
| Reality Checker | "默认 NEEDS WORK，要求压倒性证据才能通过" | 天然对抗人类的乐观偏误和确认偏误 |
| Evidence Collector | "不看承诺，只看截图证据，默认找到 3-5 个问题" | 对抗"差不多就行"的心态 |
| Minimal Change Engineer | "只碰必须碰的，不顺手优化" | 对抗过度工程的冲动 |
| Whimsy Injector | "每个有趣元素必须有功能或情感目的" | 避免为花哨而花哨 |
| Reddit Community Builder | "你是在成为一个有价值的社区成员，而不是在做营销" | 对抗营销人的推销本能 |

**设计你的 Agent 性格时的黄金法则**：
1. 找出你这个领域最常见的失败模式
2. 把对抗这个失败模式的「过度矫正」设为 Agent 的默认性格
3. 用一句口号式标语（vibe）让人瞬间记住

### 3.3 Agent 文件的效率公式

```
Agent 质量 = 专业深度 × 性格鲜明度 × 可操作性

专业深度：具体到代码片段级别的知识呈现
性格鲜明度：读完能记住这个 Agent 的「人设」
可操作性：不看上下文也能按步骤执行的流程
```

---

## 🔗 四、多 Agent 协作模式（最核心的实战精华）

### 4.1 串行交接模式 (Sequential Handoff)

**适用场景**：前后依赖的流水线任务

```
Agent A 输出 → 粘贴到 → Agent B 的 prompt → Agent B 输出 → 粘贴到 → Agent C
```

**关键规则**：
- 永远粘贴完整输出，不要总结（总结会丢失细节）
- 每个 Agent 的输入 = 上一个 Agent 的原始输出
- 在这条链中插入 Reality Checker 作为质量门

**实例**（Startup MVP 4 周计划）：
```
Week 1: Sprint Prioritizer → UX Researcher (并行)
              ↓
        Backend Architect (串联，吃前两者的输出)
Week 2: Frontend Developer + Rapid Prototyper (并行)
              ↓
        Reality Checker (质量门)
Week 3: Frontend Developer + Growth Hacker (并行)
Week 4: Reality Checker (最终验证) → GO/NO-GO
```

### 4.2 并行协作模式 (Parallel Collaboration)

**适用场景**：互不依赖的独立维度分析

```
                    ┌→ Agent A (维度1)
一个任务分发 → ┼→ Agent B (维度2)
                    └→ Agent C (维度3)
                          ↓
                    Orchestrator 汇总（人工或自动化）
```

**实例**（完整产品探索 — Nexus 模式）：
```
同一个需求同时发给 8 个 Agent：
- Product Trend Researcher → 市场验证
- Backend Architect → 技术架构
- Brand Guardian → 品牌策略
- Growth Hacker → 增长策略
- UX Researcher → 用户研究
- Project Shepherd → 项目执行
- XR Interface Architect → 空间 UI 设计
- Support Responder → 支持系统

→ 单次会话产出跨职能产品蓝图
```

### 4.3 质量门模式 (Quality Gate)

**这是整个工作流模式中最重要的模式**。

```
每个关键里程碑 → Reality Checker（默认: NEEDS WORK）
     ↓
通过？
  ├─ YES → 进入下一阶段
  └─ NO  → 回到对应专业 Agent 修复 → 再检查 → 直到通过
```

**为什么这个模式重要**：
- AI 倾向于"取悦用户"，容易给出 A+ 的虚假评价
- Reality Checker 的性格是"默认不通过"，这个偏置正好抵消 AI 的讨好偏置
- 强制要求截图证据、性能数据、清单验证

### 4.4 多 Agent 工作流通用模板

```
Phase 0: 启动
  └─ Orchestrator/Project Shepherd 分解任务、分配 Agent

Phase 1: 并行探索（信息收集）
  └─ 2-N 个分析型 Agent 同时在不同维度工作

Phase 2: 串联构建（核心生产）
  └─ 上游 Agent 输出 → 下游 Agent 输入

Phase 3: 质量验证（每个里程碑）
  └─ Reality Checker/Evidence Collector → 通过 or 返工

Phase 4: 集成交付
  └─ 汇总所有 Agent 产出 → 统一交付物
```

### 4.5 上下文传递协议

**Agent 之间不共享记忆**，上下文传递必须显式完成：

```
❌ 错误: "根据之前讨论的方案来实现"（Agent 不知道之前讨论了什么）
✅ 正确: "以下是 Backend Architect 的完整 API 设计输出：\n\n[paste full output]\n\n请基于此实现前端。"
```

**传递时的 Checklist**：
- [ ] 粘贴完整的上一阶段输出
- [ ] 明确说明当前阶段的输入是什么
- [ ] 明确说明当前阶段的期望输出格式
- [ ] 如果有约束条件（技术栈、时间限制），显式声明

---

## 🛠 五、在你的项目中使用 Agent 的实践指南

### 5.1 选择 Agent 的决策树

```
你的任务是什么？
├─ 写代码
│   ├─ 前端 UI → Frontend Developer + UI Designer
│   ├─ 后端 API → Backend Architect + Database Optimizer
│   ├─ 全栈 MVP → Rapid Prototyper + Frontend Developer + Backend Architect
│   ├─ 代码审查 → Code Reviewer + Senior Developer
│   └─ DevOps → DevOps Automator + SRE
├─ 产品规划
│   ├─ 需求排序 → Sprint Prioritizer + Product Manager
│   ├─ 用户研究 → UX Researcher
│   └─ 趋势分析 → Trend Researcher
├─ 设计
│   ├─ 设计系统 → UI Designer + Brand Guardian
│   ├─ 交互设计 → UX Architect + Persona Walkthrough
│   └─ 趣味性 → Whimsy Injector
├─ 质量保证
│   ├─ 功能测试 → Evidence Collector
│   ├─ 现实检查 → Reality Checker
│   └─ 性能测试 → Performance Benchmarker
├─ 营销推广
│   ├─ 内容策略 → Content Creator
│   ├─ 平台运营 → [对应平台] Strategist
│   └─ SEO → SEO Specialist
└─ 项目管理
    ├─ 任务调度 → Project Shepherd + Senior Project Manager
    ├─ 会议记录 → Meeting Notes Specialist
    └─ 实验追踪 → Experiment Tracker
```

### 5.2 快速启动：最小可行 Agent 团队

不需要 232 个全部用。针对 3 种典型场景的最小团队：

**场景 A：快速原型开发（3 Agent）**
```
Rapid Prototyper → Frontend Developer → Reality Checker
```

**场景 B：正式产品开发（5 Agent）**
```
Sprint Prioritizer → Backend Architect → Frontend Developer → Evidence Collector → Reality Checker
```

**场景 C：营销内容生产（4 Agent）**
```
Content Creator → SEO Specialist → Social Media Strategist → Analytics Reporter
```

### 5.3 Agent Prompt 的编写公式

激活一个 Agent 时，用这个模板写 prompt：

```markdown
Activate [Agent 名称].

**背景**: [1-2 句话说明当前项目状态]
**任务**: [具体要这个 Agent 完成什么]
**输入**: [粘贴之前 Agent 的完整输出，如果有]
**约束**: [技术栈、时间限制、预算、品牌要求]
**输出格式**: [期望的交付物格式]

请按你的工作流执行并交付。
```

### 5.4 常见的 Agent 组合模式

**模式 1：建造者 + 检查者 (Builder + Checker)**
```
任何 Builder Agent 的产出 → 立刻交给对应的 Checker Agent
例: Frontend Developer → Code Reviewer → Reality Checker
```

**模式 2：策略 + 执行 + 优化 (Strategy + Execution + Optimization)**
```
Strategist 定方向 → Executor 落地 → Optimizer 分析数据再迭代
例: Social Media Strategist → Content Creator → Analytics Reporter → 回到 Strategist
```

**模式 3：分歧 + 收敛 (Diverge + Converge)**
```
多个独立 Agent 并行探索 → Orchestrator 汇总 → 单一 Agent 执行
例: UX Researcher + Trend Researcher + Competitive Analyst → Product Manager 汇总 → Backend Architect
```

---

## 📝 六、如何创建你自己的 Agent 模板

基于 Agency Agents 的设计模式，以下是创建高质量 Agent 的 SOP：

### Step 1: 确定 Agent 定位

```
1. 这个 Agent 要解决什么具体问题？
2. 这个领域最常见的失败模式是什么？
3. 对抗这个失败模式需要什么样的性格？
```

### Step 2: 设计性格 (Personality Canvas)

```
名称: [专业名词，不是动词]
角色: [一句话角色定位]
性格: [3-4 个形容词]
口头禅/Vibe: [一句让人记住的话]
最大的恐惧: [这个领域最怕出现的情况]
本能的反应: [面对问题时的第一反应]
```

### Step 3: 定义核心使命 (3-5 条)

```
格式: 动词 + 宾语 + 具体化限定
好例子: "创建像素级精准的响应式 Web 应用，兼容所有主流浏览器"
坏例子: "写好的前端代码"
```

### Step 4: 设定铁律 (3-5 条硬规则)

```
格式: 不可协商的原则 + 为什么重要
好例子: "永远先做移动端布局，再做桌面端适配。因为 70% 的流量来自移动端。"
坏例子: "注意响应式设计"
```

### Step 5: 编写技术交付物 (含真实代码)

```
关键是真实、可运行的代码，不是伪代码。
最少包含：
- 1 段核心实现的代码示例
- 1 个架构/流程模板
- 1 个输出报告模板
```

### Step 6: 定义工作流 (4-6 步骤)

```
Step 1: [初始化/调研] — [具体行动] → 验证: [检查项]
Step 2: [核心工作] — [具体行动] → 验证: [检查项]
Step 3: [完善] — [具体行动] → 验证: [检查项]
Step 4: [交付] — [具体行动] → 验证: [检查项]
```

### Step 7: 设定成功度量

```
每个指标必须是：
1. 可量化（有数字）
2. 可验证（怎么检查）
3. 有基准（当前水平 vs 目标水平）
```

### Step 8: 注入记忆

```
Agent 应该「记住」的内容：
- 这个领域经过验证的最佳实践
- 常见的陷阱和反模式
- 典型的失败案例
- 效率提升的技巧
```

### Agent 模板骨架

```markdown
---
name: [Agent 名称]
description: [一句话描述]
color: [颜色: blue/green/purple/red/orange/teal/pink/cyan/yellow]
emoji: [一个 emoji]
vibe: "[一句体现性格的话]"
---

# [Agent 名称]

You are **[Agent 名称]**, [角色定位]. You specialize in [专业领域].

## 🧠 Your Identity & Memory
- **Role**: [角色]
- **Personality**: [性格特征]
- **Memory**: [你记住了什么]
- **Experience**: [你经历过什么]

## 🎯 Your Core Mission

### [核心使命1]
- [具体行动1]
- [具体行动2]
- **Default requirement**: [默认非功能性要求]

### [核心使命2]
- [具体行动1]
- [具体行动2]

## 🚨 Critical Rules You Must Follow

### [规则类别1]
- [硬规则1] — [为什么]
- [硬规则2] — [为什么]

### [规则类别2]
- [硬规则1]
- [硬规则2]

## 📋 Your Technical Deliverables

### [交付物1 — 含真实代码]
```[语言]
// 真实可运行的代码示例
```

### [交付物2 — 模板]
```markdown
# [模板名称]
## [章节1]
## [章节2]
```

## 🔄 Your Workflow Process

### Step 1: [阶段名]
- [具体行动]
- [验证方式]

### Step 2: [阶段名]
- [具体行动]
- [验证方式]

### Step 3: [阶段名]
- [具体行动]
- [验证方式]

### Step 4: [阶段名]
- [具体行动]
- [验证方式]

## 💭 Your Communication Style
- [语气特点1]: "[示例表达]"
- [语气特点2]: "[示例表达]"

## 🎯 Your Success Metrics
- [量化指标1]: [具体数值]
- [量化指标2]: [具体数值]
- [量化指标3]: [具体数值]
```

---

## 🎯 七、对这个项目的批判性评估

### 7.1 做对了什么

| 设计决策 | 为什么好 |
|---------|---------|
| **性格偏置对抗** | 用 Agent 的性格偏置对抗 AI 的系统性偏误（如 Reality Checker 默认不通过） |
| **不共享记忆** | Agent 间不共享上下文，强制显式信息传递，减少幻觉链式放大 |
| **交付物模板化** | 每次产出的格式一致，方便 diff 和迭代 |
| **量化成功标准** | 把"做得好"变成可以检查的数字 |
| **多工具集成** | 支持 14 种工具，不绑定单一平台 |
| **MIT 开源** | 可以自由修改、商用，社区贡献门槛低 |

### 7.2 局限性

| 局限性 | 影响 | 如何应对 |
|--------|------|---------|
| **Agent 数量膨胀** | 232 个太多，选择困难 | 从最小团队（3-5 个）开始，按需增加 |
| **上下文消耗大** | 每个 Agent 文件都很长，加载消耗 token | 实际使用时可以裁剪，只保留核心使命+铁律+工作流 |
| **质量依赖输入** | Agent 输出质量完全取决于你给的上下文 | 投资写好激活 prompt，粘贴完整的上游输出 |
| **需要人工编排** | 多 Agent 工作流目前需要人工顺序调用 | 可以用 Orchestrator Agent + 脚本实现自动化编排 |
| **英文为主** | 对中文场景需要适配 | 翻译核心 Agent + 添加中国特有的平台 Agent（如小红书、抖音） |

### 7.3 什么场景下不应该用

- **单一简单任务**：写一个函数、改一个 bug → 太重型，直接用 Claude Code 默认模式
- **探索性编程**：不确定要做什么的实验阶段 → 先自己探索，再引入 Agent
- **上下文极度受限**：token 预算紧张 → Agent 文件本身消耗大
- **完全创新的领域**：Agent 的知识来自训练数据，对真正全新的问题帮助有限

---

## 📦 八、落地建议：如何整合到你的 SOP 体系

### 8.1 与现有 SOP 的结合方式

你已有的 SOP（iOS、HarmonyOS、Android、Web、Backend、Fullstack）是**流程指南**，Agency Agents 是**执行者**。结合方式：

```
SOP 定义「做什么」 + Agent 定义「怎么做」
     ↓                          ↓
  阶段1: [RESEARCH]      激活 UX Researcher
  阶段2: [GENERATE]      激活 Frontend Developer / Backend Architect
  阶段3: [DEBUG]         激活 Code Reviewer / Senior Developer
  阶段4: [REVIEW]        激活 Reality Checker / Evidence Collector
  阶段5: [VALIDATE]      激活 Testing 部门的 Agent
```

### 8.2 推荐创建的项目专属 Agent

基于你的 SOP 体系（iOS/HarmonyOS/Android/Web/Backend/Fullstack/创意内容），建议创建：

```
项目 Agent 团队 (8-10 个):
├── 🏗️ SOP 执行引擎 — 读取 SOP 并逐步执行，跟踪阶段进度
├── 🔍 代码审计员 — 对标各平台的代码规范和质量门
├── 🧪 编译守护者 — 专门处理编译错误，不把错误传给用户
├── 🎨 跨平台 UI 适配师 — 确保 iOS/HarmonyOS/Android/Web 的 UI 一致性
├── 📱 应用商店上架专家 — 处理 App Store/华为/Google Play 的提交清单
├── 🎬 短视频制作人 — 执行 Novel_to_Short_Drama_SOP 的创意流程
├── 🔒 安全合规官 — 覆盖 OWASP + 各平台安全最佳实践
├── 📋 版权申请助手 — 处理软件著作权申请的材料和流程
└── 🚨 现实检查员 — 每个 SOP 阶段结束后的质量门（从本项目借鉴）
```

### 8.3 从 Agency Agents 直接可用的 Agent（拿来就用）

以下 Agent 可以直接应用于你的项目，无需修改：

| Agent | 在你的项目中的用途 |
|-------|------------------|
| `Reality Checker` | 每个 SOP Phase 结束后的质量门 |
| `Code Reviewer` | Phase 8 代码审计 |
| `Frontend Developer` | Web/Fullstack SOP 的前端部分 |
| `Backend Architect` | Backend/Hono SOP 的后端部分 |
| `Mobile App Builder` | iOS/Android/HarmonyOS 的跨平台移动端 |
| `Git Workflow Master` | Git 工作流规范执行 |
| `Technical Writer` | SOP 文档维护和更新 |
| `Performance Benchmarker` | 各平台的性能基准测试 |
| `Accessibility Auditor` | 无障碍合规检查 |
| `API Tester` | 后端 API 测试 |
| `Security Architect` | 安全架构审查 |

---

## 📚 九、参考资料与延伸阅读

- **原始仓库**: [github.com/msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents)
- **CLI 工具**: `npx agency-agents-cli@latest list`
- **桌面 App**: [agencyagents.app](https://agencyagents.app) (macOS/Linux/Windows)
- **中文翻译版**: [github.com/dsclca12/agent-teams](https://github.com/dsclca12/agent-teams)
- **作者**: [Michael Sitarzewski](https://github.com/msitarzewski)
- **许可证**: MIT（可自由使用、修改、商用）

---

## 🔑 十条核心精华速记

```
1. Agent = 专业深度 × 性格鲜明度 × 可操作性
2. 性格偏置对抗：Agent 的"偏见"抵消 AI 的系统性偏见
3. 串行交接：完整粘贴上游输出，不要总结
4. 质量门：每个里程碑插入 Reality Checker（默认不通过）
5. 并行探索：多 Agent 独立分析不同维度，然后汇总
6. 最小团队：3-5 个 Agent 就能覆盖 80% 的场景
7. 显式传参：Agent 不共享记忆，上下文必须显式传递
8. 铁律优先：3-5 条硬规则比 20 条建议更有效
9. 量化交付：每个 Agent 的成功标准必须是可验证的数字
10. 拿来就用：232 个 Agent 中至少 10-15 个可以直接在你的项目中用
```

---

*此文档由 Claude 深度学习 agency-agents 仓库后提炼生成，供 SOP 项目体系使用。*
*原始项目持续更新中（最后更新 2026-06-30），建议定期同步上游变更。*
