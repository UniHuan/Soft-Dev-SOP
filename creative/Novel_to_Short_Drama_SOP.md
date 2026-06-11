# 📖 Claude Code 短篇小说创作 & 小云雀2.0 短剧制作全流程 SOP

> **适用对象**: Claude Code (AI Agent) 全自动执行 + 用户协作
> **目标**: 使用 Claude Code 完成 10 万字短篇小说创作，通过小云雀 2.0 制作短剧视频
> **技术栈**: Claude Code (AI 写作引擎) + 小云雀 2.0 (AI 视频生成平台) + Markdown 结构化写作
> **最终产出**: 完整的 10 万字小说 + 10-30 集短剧视频
> **创作周期**: 5-7 天 (写作 4-5 天 + 视频制作 1-2 天)
> **最低要求**: Claude Code 可用 + 小云雀 2.0 账号 + 基础图片/音乐素材 (可选)

---

## 🧠 Claude Code 技能调用矩阵

> **每个步骤标注了 Claude Code 需要调用的核心技能。Claude Code 在读到该步骤时自动切换对应模式。**

| 技能标识 | 技能名称 | 说明 | Claude Code 工具 |
|---------|---------|------|-----------------|
| `[SHELL]` | Shell 执行 | 执行 bash 命令、文件管理、字数统计 | `Bash` |
| `[WRITE]` | 文件写入 | 创建/修改 .md 小说章节文件 | `Edit` |
| `[READ]` | 文件读取 | 读取已有章节、大纲、人设 | `Read` |
| `[DIALOG]` | 用户交互 | 与用户讨论剧情方向、确认关键决策 | 对话输出 |
| `[GENERATE]` | 内容生成 | 生成小说正文、大纲、对白、场景描写 | `Edit` |
| `[REVIEW]` | 内容审查 | 审查剧情连贯性、人物一致性、文笔质量 | `Read` → 分析 |
| `[RESEARCH]` | 知识检索 | 查阅写作技法、类型文学规范、短剧剧本格式 | `Read` 参考文档 |
| `[VALIDATE]` | 验证检查 | 对照清单逐项验证章节质量 | `Bash` + `Read` + 分析 |
| `[GIT]` | 版本控制 | git add/commit/push，保留每个版本 | `Bash` |

### 技能调用原则

```
1. 每个创作 Phase 开始前 → [RESEARCH] 回顾大纲和人设，保持一致性
2. 每次生成章节后 → [REVIEW] 自检剧情连贯性、人物性格一致、文笔流畅度
3. 每完成一个章节 → [VALIDATE] 对照验收条件逐项确认
4. 每个 Phase 结束后 → [GIT] 提交章节，保留版本历史
5. 涉及剧情方向/关键转折 → [DIALOG] 必须获得用户确认
6. 每 5000 字 → [SHELL] 统计字数进度，确保总量达标
```

---

## 📋 总览时间线

```
Day 1 (8h) — 规划与大纲                     Day 2 (8h) — 第一幕创作
├─ [0.0h] 项目初始化                        ├─ [0.0h] 第 1-3 章 (开放)
├─ [0.5h] 题材/类型确认                      ├─ [2.5h] 第 4-6 章 (铺垫)
├─ [1.5h] 三幕结构设计                       ├─ [5.0h] 第 7-9 章 (激励事件)
├─ [2.5h] 人物小传                           └─ [8.0h] 字数验收 & Git 提交
├─ [4.0h] 分章大纲 (30-40章)
├─ [6.0h] 世界观设定
└─ [8.0h] 总纲验收

Day 3 (8h) — 第二幕创作                     Day 4 (8h) — 第三幕创作
├─ [0.0h] 第 10-13 章 (对抗升级)             ├─ [0.0h] 第 21-24 章 (高潮前奏)
├─ [2.5h] 第 14-17 章 (中间转折)             ├─ [2.5h] 第 25-28 章 (高潮)
├─ [5.0h] 第 18-20 章 (至暗时刻)             ├─ [5.0h] 第 29-32 章 (结局)
└─ [8.0h] 字数验收 & 连贯性检查               └─ [8.0h] 第 33-36 章 (尾声)

Day 5 (4h) — 编辑润色                         Day 6 (4h) — 短剧改编
├─ [0.0h] 全局通读 & 错别字检查               ├─ [0.0h] 分集大纲 (10-30 集)
├─ [1.0h] 人物弧光一致性审核                 ├─ [1.0h] 场景分镜脚本
├─ [2.0h] 节奏调整 & 删除冗余                 ├─ [2.0h] 对白精炼
└─ [4.0h] 终稿定稿                            └─ [4.0h] 小云雀导入文件准备

Day 7 (4h) — 视频制作
├─ [0.0h] 小云雀 2.0 项目创建
├─ [1.0h] 逐集导入脚本 & AI 生成
├─ [2.5h] 视频审核 & 替换调整
└─ [4.0h] 全集导出 & 归档
```

---

## ⚠️ 执行前提 (Claude Code 自动检查)

```bash
# [SHELL] 1. 确认工作目录
PROJECT_ROOT="/Users/xurui/Projects/SOP/creative"
mkdir -p "$PROJECT_ROOT/novels"
mkdir -p "$PROJECT_ROOT/output"
echo "✅ 工作目录就绪: $PROJECT_ROOT"

# [SHELL] 2. 确认 Git 可用
git --version 2>/dev/null || { echo "❌ 未安装 Git"; exit 1; }

# [SHELL] 3. 确认磁盘空间 >= 1GB (用于小说多版本存储)
df -h "$PROJECT_ROOT" | tail -1
```

---

## Phase 0: 项目初始化

> **[SHELL]** + **[DIALOG]** + **[WRITE]** 创建项目结构 → 确认基本信息 → 初始化小说工程

### Step 0.1 — 项目目录创建

```bash
# [SHELL] 持久化项目变量
PROJECT_NAME="novel_$(date +%Y%m%d)"
PROJECT_DIR="$PROJECT_ROOT/novels/$PROJECT_NAME"

cat > /tmp/sop_novel.env << ENVEOF
PROJECT_NAME="$PROJECT_NAME"
PROJECT_DIR="$PROJECT_DIR"
NOVEL_DIR="$PROJECT_DIR/novel"
SCRIPTS_DIR="$PROJECT_DIR/scripts"
OUTPUT_DIR="$PROJECT_ROOT/output/$PROJECT_NAME"
TOTAL_TARGET=100000
DAILY_TARGET=25000
CREATED_AT="$(date '+%Y-%m-%d %H:%M:%S')"
ENVEOF

source /tmp/sop_novel.env

# 创建目录结构
mkdir -p "$NOVEL_DIR/chapters"       # 章节目录
mkdir -p "$NOVEL_DIR/outlines"       # 大纲目录
mkdir -p "$NOVEL_DIR/characters"     # 人设目录
mkdir -p "$NOVEL_DIR/worldbuilding"  # 世界观设定
mkdir -p "$SCRIPTS_DIR/episodes"     # 短剧分集剧本
mkdir -p "$OUTPUT_DIR"               # 最终输出
mkdir -p "$PROJECT_DIR/logs"         # 创作日志

echo "✅ 项目目录创建完成: $PROJECT_DIR"
```

### Step 0.2 — 元信息配置

```bash
# [WRITE] 创建 meta.yaml
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/meta.yaml" << 'META'
小说标题: (待定)
作者: 
创作日期: (自动填充)
目标字数: 100,000 字
类型: (待 Phase 1 确认)
章数: 30-40 章
目标读者: 
一句话梗概: (待 Phase 1 确认)
META
```

### Step 0.3 — 进度追踪器

```bash
# [SHELL] 创建进度追踪脚本
source /tmp/sop_novel.env
cat > "$PROJECT_DIR/logs/progress.sh" << 'SCRIPT'
#!/bin/bash
source /tmp/sop_novel.env 2>/dev/null
if [ -d "$NOVEL_DIR/chapters" ]; then
  CHINESE_CHARS=$(cat "$NOVEL_DIR/chapters"/*.md 2>/dev/null | grep -oP '[\x{4e00}-\x{9fff}\x{3400}-\x{4dbf}\x{f900}-\x{faff}]' | wc -l | tr -d ' ')
else
  CHINESE_CHARS=0
fi
CHAPTER_COUNT=$(ls "$NOVEL_DIR/chapters"/*.md 2>/dev/null | wc -l | tr -d ' ')
echo "📊 创作进度"
echo "  已完成章节: $CHAPTER_COUNT"
echo "  已写中文字数: $CHINESE_CHARS / $TOTAL_TARGET"
if [ "$CHINESE_CHARS" -gt 0 ] && [ "$TOTAL_TARGET" -gt 0 ]; then
  echo "  完成度: $(( CHINESE_CHARS * 100 / TOTAL_TARGET ))%"
  echo "  预估剩余: $(( TOTAL_TARGET - CHINESE_CHARS )) 字"
fi
SCRIPT
chmod +x "$PROJECT_DIR/logs/progress.sh"
```

### Step 0.4 — Git 初始化

```bash
# [GIT]
source /tmp/sop_novel.env
cd "$PROJECT_DIR"
git init
git checkout -b main
cat > .gitignore << 'EOF'
.DS_Store
*.tmp
output/
logs/progress.log
EOF

git add -A
git commit -m "chore: initialize novel project structure

- novels/ : 小说章节
- scripts/ : 短剧分集剧本
- logs/ : 创作日志与进度追踪"
```

> **Phase 0 验收**: 项目目录存在 ✅ | meta.yaml 创建 ✅ | Git 已初始化 ✅ | 进度追踪可用 ✅

---

## Phase 1: 小说定位 & 题材确认

> **[DIALOG]** + **[RESEARCH]** + **[WRITE]** 与用户深度对话 → 确定小说核心定位 → 输出类型定义文档

### Step 1.1 — 题材类型确认

> **[DIALOG]** Claude Code 与用户深入讨论，确定以下要素：

**讨论清单:**

```
1. 题材定位:
   □ 都市情感 (甜宠/虐恋/婚恋/追妻火葬场)
   □ 古装仙侠 (修仙/玄幻/武侠/宫斗)
   □ 悬疑推理 (刑侦/心理/密室/反转)
   □ 科幻未来 (星际/末世/赛博朋克/AI)
   □ 现实题材 (职场/家庭/成长/社会)
   □ 奇幻冒险 (异世界/魔法/异能/怪兽)
   □ 短剧热门: 霸总甜宠 / 赘婿逆袭 / 穿越重生 / 战神归来 / 萌宝助攻

2. 目标受众:
   □ 女性向 (言情/情感/成长)
   □ 男性向 (热血/争霸/升级)
   □ 全年龄 (家庭/喜剧/治愈)

3. 情感基调:
   □ 甜宠轻松 (下饭剧)
   □ 虐心催泪 (情感共鸣)
   □ 爽文快节奏 (强冲突)
   □ 悬疑烧脑 (高智商)
   □ 温暖治愈 (慢节奏)

4. 短剧适配度评估:
   - 冲突密度: 每集至少 1 个冲突点
   - 对白占比: ≥60% (短剧核心是对话)
   - 场景集中度: 80% 场景 ≤5 个主要场景
   - 单集时长: 1-3 分钟 / 集
```

### Step 1.2 — 一句话梗概

> **[GENERATE]** + **[DIALOG]** Claude Code 基于讨论生成 3 个一句话梗概，用户选择或修改

```markdown
# [GENERATE] 梗概示例模板

## 选项 A: 高概念反转型
当 [主角] 发现 [惊天秘密]，他/她必须在 [时限] 内 [核心动作]，
否则 [严重后果]。但真正的敌人，竟然是 [意外反转]。

## 选项 B: 情感共鸣型
[主角身份] 的 [主角名] 遇到 [触发事件]，
在 [挑战过程] 中逐渐 [成长/改变]，
最终 [情感落点]。

## 选项 C: 悬念驱动型
[离奇事件] 发生后，[主角] 被卷入 [巨大阴谋]，
每揭开一层真相，就离 [终极秘密] 更近一步，
直到 [结局反转]。
```

### Step 1.3 — 核心卖点提炼

> **[GENERATE]** 输出小说的 5 个核心卖点

```bash
# [WRITE] 创建核心卖点文件
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/outlines/00_core_pitch.md" << 'PITCH'
# 核心卖点 (用于后续宣传和短剧标题)

## 卖点提炼

1. 卖点一: [独特设定/身份反差/金手指]
2. 卖点二: [强情感冲突/关系张力]
3. 卖点三: [持续悬念/谜题]
4. 卖点四: [价值观传递/情感共鸣]
5. 卖点五: [节奏爽点/反转密度]

## 对标作品 (参考但不抄袭)
- 同类小说: 《XXX》
- 同类短剧: 《XXX》
- 差异化: [我们独特的视角/设定/反转]
PITCH
```

### Step 1.4 — 类型定义文档

```bash
# [GIT]
source /tmp/sop_novel.env
cd "$PROJECT_DIR"
git add -A
git commit -m "feat(phase-1): 小说定位与题材确认"
```

> **Phase 1 验收**: 题材确认 ✅ | 梗概敲定 ✅ | 卖点清晰 ✅ | 用户签字确认 ✅

---

## Phase 2: 三幕结构 & 分章大纲

> **[RESEARCH]** + **[GENERATE]** + **[WRITE]** 基于类型定义构建完整故事结构 → 分章大纲

### Step 2.1 — 三幕结构设计

> **[RESEARCH]** Claude Code 回顾核心卖点和类型 → **[GENERATE]** 三幕结构

```bash
# [WRITE] 创建三幕结构文档
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/outlines/01_three_act_structure.md" << 'STRUCTURE'
# 三幕结构 — 10万字小说框架

## 第一幕: 开局 (约 25,000 字 / 8-10 章)

| 章节 | 功能 | 关键事件 | 冲突等级 |
|------|------|---------|---------|
| 第1章 | 日常世界 | 展示主角常态生活 | ★ |
| 第2章 | 伏笔埋设 | 暗示即将到来的改变 | ★★ |
| 第3章 | 激励事件 | 打破常态的重大事件 | ★★★★★ |
| 第4章 | 拒绝召唤 | 主角犹豫/抗拒改变 | ★★★ |
| 第5章 | 被迫出发 | 外部压力迫使行动 | ★★★★ |
| 第6章 | 初入新世界 | 遇到新环境/新人物 | ★★★ |
| 第7章 | 第一次考验 | 小试牛刀/建立自信 | ★★★★ |
| 第8章 | 第一幕高潮 | 做出不可逆的选择 | ★★★★★ |

## 第二幕: 对抗 (约 50,000 字 / 14-18 章)

| 章节 | 功能 | 关键事件 | 冲突等级 |
|------|------|---------|---------|
| 第9章 | 上升行动 | 正面迎战第一个障碍 | ★★★★ |
| 第10章 | 盟友与敌人 | 人际关系网络展开 | ★★★ |
| 第11章 | 小胜 | 取得阶段性胜利 | ★★★ |
| 第12章 | 暗流涌动 | 敌人的反击酝酿 | ★★★★ |
| 第13章 | 第一次挫败 | 计划受挫/损失 | ★★★★★ |
| 第14章 | 至暗时刻前奏 | 局势恶化 | ★★★★ |
| 第15章 | 中间转折 | 发现关键线索/真相 | ★★★★★ |
| 第16章 | 重新振作 | 调整策略/获得新力量 | ★★★ |
| 第17章 | 反击开始 | 主动出击 | ★★★★ |
| 第18章 | 高潮铺垫 | 逐步逼近核心冲突 | ★★★★ |
| 第19章 | 假高潮 | 看似胜利实则陷阱 | ★★★★★ |
| 第20章 | 至暗时刻 | 最低谷/最绝望 | ★★★★★ |
| 第21章 | 顿悟 | 发现核心真相/解决办法 | ★★★★ |
| 第22章 | 最终准备 | 整合资源/集结力量 | ★★★ |

## 第三幕: 结局 (约 25,000 字 / 8-10 章)

| 章节 | 功能 | 关键事件 | 冲突等级 |
|------|------|---------|---------|
| 第23章 | 最终对决开始 | 正面终极对抗 | ★★★★★ |
| 第24章 | 拉锯战 | 来回拉锯/多次反转 | ★★★★★ |
| 第25章 | 高潮 | 决定性的一击 | ★★★★★ |
| 第26章 | 尘埃落定 | 揭示后果 | ★★★★ |
| 第27章 | 新平衡 | 世界恢复/重建 | ★★★ |
| 第28章 | 回报 | 主角获得成长/奖励 | ★★ |
| 第29章 | 告别 | 重要关系的交代 | ★★★ |
| 第30章 | 尾声 | 升华主题/余韵 | ★ |
STRUCTURE
```

### Step 2.2 — 每章详细大纲 (300-500字/章)

> **[GENERATE]** Claude Code 为每章生成详细大纲

```bash
# [WRITE] 创建分章大纲模板
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/outlines/02_chapter_outlines.md" << 'OUTLINE'
# 分章详细大纲

## 第 N 章大纲模板

### 章节: "[章节标题]"
- **字数目标**: 2,500-3,500 字
- **时间线**: [故事内时间]
- **场景**: [地点]
- **出场人物**: [角色A、角色B...]

### 剧情概要
[150-200字概括本章内容]

### 核心冲突
- **外部冲突**: [人物 vs 人物/环境/社会...]
- **内部冲突**: [主角内心的矛盾/挣扎...]

### 关键场景 (3-5个)
1. [场景一描述] — 目的: [推进剧情/展示人物/埋设伏笔...]
2. [场景二描述] — 目的: [...]
3. [场景三描述] — 目的: [...]

### 情感曲线
开始: [情绪] → 发展: [情绪变化] → 结束: [情绪落点]

### 短剧适配标记
- 本集对白密集度: 高/中/低
- 视觉冲击点: [关键画面描述]
- 钩子/悬念: [章末留下的悬念]
OUTLINE
```

### Step 2.3 — 全篇伏笔 & 反转表

```bash
# [WRITE] 创建伏笔追踪矩阵
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/outlines/03_foreshadowing_matrix.md" << 'MATRIX'
# 伏笔 & 反转追踪矩阵

## 伏笔追踪表

| 编号 | 伏笔内容 | 埋设章节 | 揭示章节 | 类型 | 重要度 |
|------|---------|---------|---------|------|--------|
| F01 | [伏笔描述] | 第N章 | 第M章 | 人物/情节/设定 | ★★★★★ |
| F02 | ... | ... | ... | ... | ... |

## 反转节点地图

| 章节 | 反转类型 | 反转内容 | 前期铺垫 | 冲击指数 |
|------|---------|---------|---------|---------|
| 第N章 | 身份反转/真相揭露 | [反转描述] | F01, F03 | ★★★★★ |
MATRIX
```

### Step 2.4 — 大纲交审

```bash
# [SHELL] 大纲字数统计
source /tmp/sop_novel.env
echo "=== 大纲统计 ==="
find "$NOVEL_DIR/outlines" -name "*.md" -exec wc -m {} \;
echo "总大纲字数: $(find "$NOVEL_DIR/outlines" -name "*.md" -exec cat {} \; | wc -m) 字符"

# [DIALOG] 将大纲摘要呈现给用户，等待确认后继续
echo "📋 请审核以上大纲。确认后我们将进入人物设计阶段。"

# [GIT]
cd "$PROJECT_DIR"
git add -A
git commit -m "feat(phase-2): 三幕结构与分章大纲完成

- 三幕结构设计: 30章框架
- 每章详细大纲模板
- 伏笔矩阵 + 反转节点地图"
```

> **Phase 2 验收**: 三幕结构完整 ✅ | 30章大纲就绪 ✅ | 伏笔矩阵建立 ✅ | 用户签字确认 ✅

---

## Phase 3: 人物设计 & 世界观设定

> **[GENERATE]** + **[WRITE]** + **[REVIEW]** 创建完整人物小传 → 人物关系网 → 世界观设定文档

### Step 3.1 — 主要人物小传

> **[GENERATE]** Claude Code 基于大纲生成人物小传 (每个主要角色 800-1500 字)

```bash
# [WRITE] 创建主要人物小传文件
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/characters/01_main_characters.md" << 'CHARS'
# 主要人物小传

## 人物小传模板

### 基础信息
- **姓名**: (含寓意/来源)
- **年龄**: 
- **职业/身份**: 
- **外貌特征**: (3-5个标志性特征，便于短剧视觉呈现)
  - 例: "左眉有一道细疤"、"永远戴着一条银色十字项链"
- **标志性动作/口头禅**: 

### 性格维度
- **外在表现**: (别人眼中的他/她)
- **真实内心**: (读者知道的他/她)
- **核心欲望**: (他/她真正想要什么)
- **最大恐惧**: (他/她最害怕什么)
- **致命缺陷**: (阻碍他/她的性格弱点)
- **成长弧光**: 从 [初始状态] → 经历 [关键事件] → 变为 [最终状态]

### 背景故事
[500字以上，包括童年经历、关键转折点、未解决的心理创伤]

### 人际关系
| 关系 | 对象 | 状态演变 | 情感线索 |
|------|------|---------|---------|
| 恋人 | [角色] | 初始→发展→结局 | 从 [X] 到 [Y] |
| 对手 | [角色] | ... | ... |
| 导师 | [角色] | ... | ... |

### 人物对白风格
- **语速**: 快/中/慢
- **句式**: 长句/短句/反问/设问
- **常用词**: (3-5个该角色高频使用的词)
- **禁忌词**: (该角色永远不会说的话)
- **方言/口癖**: (如有)

### 短剧适配
- **选角建议**: [气质类型/年龄区间]
- **标志性场景**: [该角色最具视觉记忆点的场景]
CHARS
```

### Step 3.2 — 次要人物卡片

```bash
# [WRITE] 创建次要人物卡片
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/characters/02_supporting_characters.md" << 'SUPPORT'
# 次要人物卡片

## 人物卡片模板 (次要角色 — 300-500字)

- **姓名/身份**: 
- **功能**: (对剧情的贡献，不超过两句话)
- **与主角关系**: 
- **标志特征**: (1个即可)
- **出场章节**: 第N章 — 第M章
- **退场方式**: 
SUPPORT
```

### Step 3.3 — 人物关系图

```bash
# [WRITE] 创建人物关系矩阵
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/characters/03_relationship_map.md" << 'RELATION'
# 人物关系矩阵

## 关系矩阵

| | 主角A | 主角B | 反派 | 导师 | 闺蜜/兄弟 |
|--|-------|-------|------|------|-----------|
| 主角A | — | [关系] | [关系] | ... | ... |
| 主角B | | — | ... | ... | ... |
| 反派 | | | — | ... | ... |

## 关系演变时间线

- 第1章: [关系状态]
- 第8章: [关系变化] — 触发: [第一幕高潮事件]
- 第15章: [关系转折] — 触发: [中间转折事件]
- 第22章: [关系深水区] — 触发: [至暗时刻事件]
- 第30章: [关系结局]
RELATION
```

### Step 3.4 — 世界观设定

```bash
# [GENERATE] 根据小说类型生成对应世界观文档
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/worldbuilding/01_world_setting.md" << 'WORLD'
# 世界观设定

## 时空背景
- **时代**: [现代/古代/架空...]
- **地域**: [城市/国家/星球...]
- **时间跨度**: [故事覆盖的时间长度]

## 社会结构 (如适用)
- **阶层体系**: 
- **权力结构**: 
- **文化习俗**: 

## 规则系统 (如适用: 修仙/异能/科幻)
- **能力体系**: [力量来源/等级/限制]
- **禁忌/代价**: 
- **关键道具/设定**: 

## 场景清单 (短剧拍摄核心)

| 场景 | 出现章节 | 视觉特征 | 拍摄难度 |
|------|---------|---------|---------|
| [场景名] | 第N, M, P章 | [标志性视觉元素] | 低/中/高 |
WORLD
```

### Step 3.5 — 人物一致性检查清单

```bash
# [WRITE] 创建角色行为一致性守则
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/characters/04_consistency_checklist.md" << 'CHECKLIST'
# 角色行为一致性守则

创作每一章时，在写该角色对白/行动前，问自己:
1. 这句话/行为是否符合该角色的「致命缺陷」?
2. 他/她的「核心欲望」是否在驱动这个决定?
3. 这个反应是否符合该角色的「背景故事」?
4. 口头禅/对白风格是否正确?
CHECKLIST
```

### Step 3.6 — Phase 3 归档

```bash
# [GIT]
source /tmp/sop_novel.env
cd "$PROJECT_DIR"
git add -A
git commit -m "feat(phase-3): 人物设计与世界观设定完成

- 主要人物小传模板
- 次要人物卡片模板
- 人物关系图 + 演变时间线
- 世界观设定文档
- 人物一致性检查清单"
```

> **Phase 3 验收**: 主要人物小传完整 ✅ | 关系网清晰 ✅ | 世界观自洽 ✅ | 用户签字确认 ✅

---

## Phase 4: 第一幕创作 — 开局篇 (第 1-8 章)

> **[GENERATE]** + **[REVIEW]** + **[VALIDATE]** 逐章创作第一幕 → 自动审查 → 字数把控

### 每章创作标准流程

```
1. [RESEARCH] 重读该章大纲 + 相关人物小传 + 伏笔矩阵
2. [GENERATE] 生成 2,500-3,500 字正文
3. [REVIEW] 自动审查:
   - 人物行为是否符合人设?
   - 伏笔是否正确埋设?
   - 对白占比是否 ≥ 60%?
   - 章节尾部是否留有钩子?
4. [VALIDATE] 对照验收清单逐项确认
5. [SHELL] 更新进度追踪
6. [GIT] 提交该章
```

### Step 4.1 — 第 1 章: 开场钩子

> **[GENERATE]** 第 1 章是全书最重要的章节，决定读者/观众是否继续

```
第 1 章创作要点:
□ 前 300 字必须出现「钩子」— 一个让读者必须继续读的问题/悬念/冲突
□ 展示主角的「日常世界」和「核心欲望」的萌芽
□ 暗示即将到来的改变（伏笔 F01）
□ 建立至少 1 个读者可以共鸣的情感锚点
□ 对白占比 ≥ 60%
□ 章节末尾留悬念
```

```bash
# [SHELL] 创建第1章文件
source /tmp/sop_novel.env
cat > "$NOVEL_DIR/chapters/ch01_opening.md" << 'CH01'
# 第一章: [章节标题]

[Claude Code 在此生成 2,500-3,500 字正文]

---
*字数: [N] 字 | 创作时间: ... | 伏笔: F01*
CH01

# [REVIEW] 审查第1章
echo "=== 第1章审查 ==="

# [GIT]
cd "$PROJECT_DIR"
git add -A
git commit -m "feat(phase-4): 第1章 — 开场钩子"
```

### Step 4.2 — 第 2-8 章: 铺垫、激励事件、第一幕高潮

> **[GENERATE]** 逐章生成，每章独立审查

```bash
# [SHELL] 逐章创作框架
source /tmp/sop_novel.env

for ch in 02 03 04 05 06 07 08; do
  echo "📝 正在创作第${ch}章..."
  
  # [RESEARCH] 读取大纲中该章内容
  # [GENERATE] Claude Code 生成章节
  # [REVIEW] Claude Code 审查
  
  # [SHELL] 字数统计
  if [ -f "$NOVEL_DIR/chapters/ch${ch}_*.md" ]; then
    WORDS=$(cat "$NOVEL_DIR/chapters/ch${ch}_"*.md | grep -oP '[\x{4e00}-\x{9fff}]' | wc -l | tr -d ' ')
    echo "  第${ch}章: $WORDS 字"
  fi
  
  # [GIT] 逐章提交
  cd "$PROJECT_DIR"
  git add -A
  git commit -m "feat(phase-4): 第${ch}章"
done

# 第一幕总字数统计
bash "$PROJECT_DIR/logs/progress.sh"
```

**每章验收清单:**
- [ ] 字数 2,500-3,500 ✓
- [ ] 章节末尾有钩子 ✓
- [ ] 伏笔按矩阵埋设 ✓
- [ ] 人物行为符合人设 ✓
- [ ] 对白占比 ≥ 60% ✓

> **Phase 4 验收**: 第 1-8 章完成 ✅ | 总字数 20,000-28,000 字 ✅ | 激励事件明确 ✅ | 第一幕高潮有力 ✅

---

## Phase 5: 第二幕创作 — 对抗篇 (第 9-22 章)

> **[GENERATE]** + **[REVIEW]** + **[VALIDATE]** 逐章创作第二幕，这是小说的主体

### Step 5.1 — 第 9-14 章: 上升行动 & 第一次挫败

```bash
# [SHELL] 逐章创作
source /tmp/sop_novel.env

for ch in 09 10 11 12 13 14; do
  echo "📝 正在创作第${ch}章..."
  # [RESEARCH] → [GENERATE] → [REVIEW] → [VALIDATE] → [GIT]
  
  cd "$PROJECT_DIR"
  git add -A
  git commit -m "feat(phase-5): 第${ch}章"
done

echo "=== 阶段性审查: 第 9-14 章 ==="
echo "审查要点:"
echo "  1. 剧情推进速度是否合理"
echo "  2. 是否有连续 2 章以上无冲突升级"
echo "  3. 反派/对立面的塑造是否充足"
echo "  4. 第一次挫败是否足够有力"
```

### Step 5.2 — 第 15-18 章: 中间转折 & 反击

```bash
# [REVIEW] 中间转折审查要点:
cat << 'REVIEW15'
🔍 中间转折审查 (第15章前后)
  1. 转折是否有前期铺垫? → 检查伏笔矩阵
  2. 转折是否改变故事走向? → 回顾三幕结构
  3. 转折是否加深人物关系? → 检查关系演变时间线
  4. 转折是否有情感冲击力? → 评估情感曲线
REVIEW15
```

### Step 5.3 — 第 19-22 章: 至暗时刻 & 顿悟

```bash
# [REVIEW] 至暗时刻审查要点:
cat << 'REVIEW20'
🔍 至暗时刻审查 (第20章前后)
  1. 这真的是最低谷吗? → 列出主角失去的一切
  2. 读者是否会产生强烈共情? → 评估情感投入度
  3. 反转是否有足够的铺垫? → 检查反转节点地图
  4. 第22章的动力是否合理? → 检查顿悟的逻辑
  5. 至暗时刻之后必然要有曙光 → 第21-22章的转折
REVIEW20
```

> **Phase 5 验收**: 第 9-22 章完成 ✅ | 总字数 55,000-65,000 字 ✅ | 伏笔 ≥ 5 个已埋设 ✅ | 反转让读者意外但合理 ✅

---

## Phase 6: 第三幕创作 — 结局篇 (第 23-30 章)

> **[GENERATE]** + **[REVIEW]** + **[VALIDATE]** 创作高潮与结局

### Step 6.1 — 第 23-26 章: 最终对决 & 高潮

```bash
# [REVIEW] 高潮审查要点:
cat << 'REVIEW23'
🔍 最终高潮审查
  1. 是否所有主要伏笔都在此汇合?
  2. 高潮场景是否有足够的视觉冲击力? (短剧关键)
  3. 主角是否主动做出关键决定? (不能是巧合)
  4. 情绪曲线是否达到了全篇最高点?
  5. 第25章高潮之后，第26章需要让读者喘口气
REVIEW23
```

### Step 6.2 — 第 27-30 章: 结局与尾声

```bash
# [REVIEW] 结局审查要点:
cat << 'REVIEW27'
🔍 结局审查
  1. 所有人物弧光是否闭合?
  2. 主题是否有明确的表达?
  3. 结尾是否有余韵? (不是戛然而止)
  4. 是否给读者情感满足?
  5. 是否暗示了未来的可能性? (番外/第二季的钩子)
REVIEW27
```

### Step 6.3 — 全篇字数终审

```bash
# [SHELL] 全篇字数终审
source /tmp/sop_novel.env
bash "$PROJECT_DIR/logs/progress.sh"

TOTAL=$(cat "$NOVEL_DIR/chapters"/*.md 2>/dev/null | grep -oP '[\x{4e00}-\x{9fff}]' | wc -l | tr -d ' ')
if [ "$TOTAL" -lt 90000 ]; then
  echo "⚠️ 字数不足 9万字，当前 $TOTAL 字，需补写 $((100000 - TOTAL)) 字"
  echo "   建议: 为重点章节的场景描写和内心独白各增加300-500字"
elif [ "$TOTAL" -gt 110000 ]; then
  echo "⚠️ 字数超出 11万字，当前 $TOTAL 字，建议 Phase 7 精简"
else
  echo "✅ 字数在目标范围内: $TOTAL / 100000"
fi
```

> **Phase 6 验收**: 第 23-30 章完成 ✅ | 全篇 95,000-105,000 字 ✅ | 所有伏笔已回收 ✅ | 人物弧光完整 ✅

---

## Phase 7: 全局编辑 & 润色

> **[READ]** + **[REVIEW]** + **[WRITE]** 全局通读 → 一致性审查 → 精修润色

### Step 7.1 — 全局通读

```bash
# [SHELL] 合并全部章节为单一文件以便连贯阅读
source /tmp/sop_novel.env
cat "$NOVEL_DIR/chapters"/*.md > "$NOVEL_DIR/full_novel_draft.md"
echo "✅ 全篇草稿已合并: $(wc -m < "$NOVEL_DIR/full_novel_draft.md" | tr -d ' ') 字符"
```

### Step 7.2 — 一致性审查清单

> **[REVIEW]** Claude Code 逐项审查

```
# [REVIEW] 一致性审查

## 人物一致性
□ 每个角色的口头禅/语言风格是否统一?
□ 角色关系演变是否按照关系时间线进行?
□ 次要角色是否有 "工具人" 感? (出现只为推进剧情)
□ 人物动机是否前后一致?
□ 同一角色不同章节的说话方式是否一致?

## 情节一致性
□ 时间线是否无矛盾?
□ 伏笔矩阵: 所有伏笔是否已回收?
□ 反转节点: 前期铺垫是否充足?
□ 是否有逻辑漏洞? (角色本可以做X却选择做Y)
□ 关键道具/设定是否前后统一?

## 节奏审查
□ 每章是否有至少 1 个冲突点?
□ 是否有连续 3 章以上无情感波动?
□ 25,000字节点: 激励事件是否已发生?
□ 50,000字节点: 中间转折是否已发生?
□ 75,000字节点: 至暗时刻是否已发生?
□ 90,000字节点: 高潮是否已到达?

## 短剧适配检查
□ 每章对白占比是否 ≥ 60%?
□ 是否有过多内心独白? (短剧无法呈现，需转为对白或动作)
□ 场景切换是否过于频繁? (>5个/章需简化)
□ 是否有强视觉记忆点? (每3章至少1个)
□ 环境描写是否足够具象? (便于AI生成画面)
```

### Step 7.3 — 文笔润色

```
润色要点:
1. 删除冗余形容词和副词 — 每句话问 "这个词删掉意思变了吗?"
2. 确保「展示，而非告知」(Show, don't tell)
3. 对白节奏优化: 短句交锋 → 长句抒情 → 短句爆发
4. 场景转换添加过渡 — 不能跳跃
5. 每章开头重新抓住注意力 — 不能用平淡叙述开头
6. 对白中去除 "他说"、"她说" — 用动作和表情代替
7. 删除重复信息的段落
```

```bash
# [SHELL] 润色后字数统计
source /tmp/sop_novel.env
bash "$PROJECT_DIR/logs/progress.sh"
```

### Step 7.4 — 终稿定稿

```bash
# [SHELL] 生成终稿
source /tmp/sop_novel.env
cat "$NOVEL_DIR/chapters"/*.md > "$OUTPUT_DIR/novel_final.md"

# 生成带目录的终稿
echo "# 《小说标题》" > "$OUTPUT_DIR/novel_final_with_toc.md"
echo "" >> "$OUTPUT_DIR/novel_final_with_toc.md"
echo "## 目录" >> "$OUTPUT_DIR/novel_final_with_toc.md"
echo "" >> "$OUTPUT_DIR/novel_final_with_toc.md"

for f in "$NOVEL_DIR/chapters"/*.md; do
  TITLE=$(head -1 "$f" | sed 's/^# //')
  echo "- $TITLE" >> "$OUTPUT_DIR/novel_final_with_toc.md"
done

echo "" >> "$OUTPUT_DIR/novel_final_with_toc.md"
echo "---" >> "$OUTPUT_DIR/novel_final_with_toc.md"
echo "" >> "$OUTPUT_DIR/novel_final_with_toc.md"
cat "$OUTPUT_DIR/novel_final.md" >> "$OUTPUT_DIR/novel_final_with_toc.md"

echo "✅ 终稿已生成: $OUTPUT_DIR/novel_final_with_toc.md"

# [GIT]
cd "$PROJECT_DIR"
git add -A
git commit -m "feat(phase-7): 全局编辑润色 & 终稿定稿

- 一致性审查通过
- 文笔润色完成
- 终稿含目录版已输出"
```

> **Phase 7 验收**: 一致性审查通过 ✅ | 伏笔全部回收 ✅ | 节奏合理 ✅ | 终稿 95,000-105,000 字 ✅ | 用户签字确认 ✅

---

## Phase 8: 短剧改编 — 分集剧本

> **[GENERATE]** + **[WRITE]** 将小说转换为短剧分集剧本 → 适配小云雀 2.0 格式

### Step 8.1 — 分集策略

```bash
# [WRITE] 创建分集策略文档
source /tmp/sop_novel.env
cat > "$SCRIPTS_DIR/00_episode_plan.md" << 'PLAN'
# 分集策略

根据小说节奏，将 30 章小说改编为 N 集短剧 (建议 15-30 集)

## 分集原则
- 每集时长: 1-3 分钟 (小云雀 2.0 最佳适配)
- 每集字数: 800-1,500 字 (对白 + 场景描述)
- 每集黄金结构: 钩子(5秒) → 冲突展开(60秒) → 小高潮(20秒) → 悬念(10秒)
- 情感曲线: 每集有独立的情感起伏

## 3 集一组的节奏单元
借鉴抖音/快手短剧的 "3集付费解锁" 模式:
- 第1集: 抛钩子 (悬念)
- 第2集: 推冲突 (升级)
- 第3集: 落卡点 (必须付费才能看下一集)

## 分集映射表

| 剧集 | 对应章节 | 核心冲突 | 情感标签 | 卡点设计 |
|------|---------|---------|---------|---------|
| EP01 | 第1章 | [冲突描述] | [标签] | [悬念] |
| EP02 | 第1-2章 | [冲突描述] | [标签] | [悬念] |
| ... | ... | ... | ... | ... |
PLAN
```

### Step 8.2 — 单集剧本模板

> **[WRITE]** 为每集创建标准化剧本

```bash
# [WRITE] 剧本模板
source /tmp/sop_novel.env
cat > "$SCRIPTS_DIR/episode_template.md" << 'TEMPLATE'
# 第 N 集剧本模板

---
episode: N
title: "[剧集标题]"
duration: "1-3分钟"
source_chapters: "第X-Y章"
emotion_tag: "[甜/虐/爽/悬疑/治愈]"
hook_score: ★★★★★
---

## 开场钩子 (前5秒)
*[1-2句最具冲击力的对白或画面描述]*

## 场景1: [场景名] 
**地点**: [场所描述]
**人物**: [出场角色]
**氛围**: [情绪基调]

[角色A]: "(对白)"
[角色B]: "(对白)"
*(动作/表情描写)*

## 场景2: [场景名]
...

## 卡点悬念 (最后5秒)
*[让观众必须点下一集的悬念/反转/情绪暴击]*

## 视觉提示 (给小云雀的生成指令)
- **画风**: [写实/动漫/古风/...]
- **色调**: [暖/冷/高饱和/...]
- **关键帧建议**: [1-2个关键画面的文字描述]
- **BGM建议**: [紧张/温馨/悲伤/...]
- **角色外观提示**: [确保角色一致性的关键描述]
TEMPLATE
```

### Step 8.3 — 批量生成分集剧本

```bash
# [SHELL] 逐集生成剧本
source /tmp/sop_novel.env
EPISODE_COUNT=20  # 根据分集策略调整

for ep in $(seq -w 1 $EPISODE_COUNT); do
  echo "📺 正在生成第 ${ep} 集剧本..."
  
  # [RESEARCH] 读取对应章节
  # [GENERATE] Claude Code 生成剧本
  # [REVIEW] 审查:
  #   - 钩子是否有冲击力?
  #   - 卡点是否让人想看下一集?
  #   - 对白是否足够精炼?
  
  # [GIT] 逐集提交
  cd "$PROJECT_DIR"
  git add -A
  git commit -m "feat(phase-8): 第${ep}集短剧剧本"
done

echo "✅ 全部 $EPISODE_COUNT 集剧本生成完毕"
```

### Step 8.4 — 小云雀 2.0 导入文件准备

```bash
# [SHELL] 生成小云雀导入文件
source /tmp/sop_novel.env
mkdir -p "$OUTPUT_DIR/xiaoyunque"

# 格式1: 逐集独立文件 (推荐，便于单集修改和重新生成)
for ep in "$SCRIPTS_DIR/episodes"/ep*.md; do
  EP_NUM=$(basename "$ep" .md)
  # 提取纯文本内容，保留结构但去除 Markdown 标记
  cat "$ep" > "$OUTPUT_DIR/xiaoyunque/${EP_NUM}.txt"
done

# 格式2: 全剧合集文本 (含分集标识)
echo "# 短剧全集剧本" > "$OUTPUT_DIR/xiaoyunque_full_script.txt"
echo "" >> "$OUTPUT_DIR/xiaoyunque_full_script.txt"
for ep in "$SCRIPTS_DIR/episodes"/ep*.md; do
  EP_NUM=$(basename "$ep" .md)
  echo "========== $EP_NUM ==========" >> "$OUTPUT_DIR/xiaoyunque_full_script.txt"
  cat "$ep" >> "$OUTPUT_DIR/xiaoyunque_full_script.txt"
  echo "" >> "$OUTPUT_DIR/xiaoyunque_full_script.txt"
done

echo "✅ 小云雀导入文件已准备:"
echo "   逐集文件目录: $OUTPUT_DIR/xiaoyunque/"
echo "   全集文本: $OUTPUT_DIR/xiaoyunque_full_script.txt"
```

```bash
# [GIT] 
cd "$PROJECT_DIR"
git add -A
git commit -m "feat(phase-8): 短剧分集剧本 & 小云雀2.0导入文件完成"
```

> **Phase 8 验收**: 分集策略合理 ✅ | 每集有钩子和卡点 ✅ | 对白精炼适合短剧 ✅ | 小云雀导入文件就绪 ✅

---

## Phase 9: 小云雀 2.0 视频制作

> **[DIALOG]** + **[SHELL]** 用户操作 + Claude Code 指导 → 导入脚本 → AI 生成视频 → 审核迭代

### Step 9.1 — 小云雀 2.0 项目创建

> **[DIALOG]** Claude Code 指导用户在浏览器中操作

```
📋 操作指引 — 小云雀 2.0 项目创建:

1. 访问小云雀 2.0 平台 (https://xiaoyunque.ai 或对应地址)
2. 登录账号
3. 点击「创建新项目」
4. 项目名称: [小说标题 - 短剧]
5. 配置参数:
   - 视频风格: [根据题材选择: 现代都市/古装/动漫/写实...]
   - 画幅比例: 9:16 (竖屏，适配抖音/快手/视频号)
   - 分辨率: 1080×1920
   - 语言: 中文
6. 点击「创建」

# [DIALOG] 等待用户确认 "项目已创建"
```

### Step 9.2 — 逐集导入脚本并 AI 生成

> **[DIALOG]** 指导用户导入每集剧本到小云雀 2.0

```
📋 逐集导入流程:

对于每一集 (EP01 — EP[N]):

1. 在小云雀中点击「导入脚本」
2. 粘贴文件内容: $OUTPUT_DIR/xiaoyunque/ep01.txt
3. 配置该集参数:
   - 集号: EP01
   - 时长: 自动/手动设定 1-3 分钟
   - 角色声音: [选择AI配音: 男声/女声/少年...]
   - 背景音乐: [从素材库选择或上传]
4. 点击「AI 生成视频」
5. 等待生成 (通常 2-5 分钟/集)

# [DIALOG] 每完成 5 集, 询问用户是否继续
# "EP01-05 已导入并生成，是否继续 EP06-10？"
```

### Step 9.3 — 视频逐集审核

> **[REVIEW]** Claude Code 提供审核维度，用户执行视觉审核

```
# 小云雀 2.0 视频审核清单 — 每集必查

## 画面质量 (最重要)
□ 画风是否符合预期?
□ 人物形象是否一致? (同一角色在不同集应保持相同外观)
□ 场景是否与剧本描述匹配?
□ 画面切换是否流畅自然?
□ 是否有明显的 AI 生成瑕疵? (手指变形/面部扭曲/文字乱码/闪烁)
□ 人物表情是否与对白情绪匹配?

## 声音质量
□ AI 配音是否自然? (语气/情绪/停顿/语速)
□ 角色声音是否与性格匹配?
□ BGM 是否与情绪契合?
□ 音量是否适中统一? (不能忽大忽小)
□ 音效是否与画面同步?

## 叙事检查
□ 钩子是否足够吸引? (前5秒能否抓住人)
□ 核心冲突是否清晰传达?
□ 卡点悬念是否有效? (让人想点下一集)
□ 本集是否完整 (有起承转合)?
□ 观众是否能理解剧情? (不过度依赖小说阅读经验)

## 平台规范 (如需发布)
□ 是否有违规内容? (暴力/色情/敏感/政治)
□ 时长是否在 1-3 分钟内?
□ 字幕是否清晰可读?
□ 是否有版权问题? (BGM/字体/素材)
```

### Step 9.4 — 问题修复 & 重新生成

> **[DIALOG]** 对不合格的剧集进行处理

```
📋 重新生成流程:

如果某集审核不通过:
1. 记录问题: [具体问题描述，如 "第3场景人物面部变形"]
2. 修改剧本:
   - 如果是画面问题 → 调整「视觉提示」部分，增加更具体的画面描述
   - 如果是对白问题 → 修改对白文本
   - 如果是节奏问题 → 调整场景顺序或冲突密度
3. 覆盖导入: 在小云雀中替换该集剧本
4. 重新生成: 点击「重新生成」
5. 再次审核: 重复 Step 9.3

# 注意: 小云雀 AI 生成存在随机性，同一剧本可能产生不同结果
# 建议: 如果 3 次重新生成都不满意，考虑:
#   a. 调整剧本中的「视觉提示」部分 — 更具体、更画面化
#   b. 更换 AI 风格/模型版本
#   c. 将该集拆分为多个子场景分别生成后拼接
#   d. 手动使用剪辑工具对画面进行微调
```

### Step 9.5 — 全集导出 & 管理

```bash
# [SHELL] 在本地建立视频管理目录
source /tmp/sop_novel.env
mkdir -p "$OUTPUT_DIR/videos/raw"        # 原始 AI 生成视频
mkdir -p "$OUTPUT_DIR/videos/final"      # 最终成品
mkdir -p "$OUTPUT_DIR/videos/thumbnails" # 封面图

# 创建视频清单
cat > "$OUTPUT_DIR/videos/manifest.md" << 'MANIFEST'
# 短剧视频清单

| 集数 | 标题 | 时长 | 状态 | 审核结果 |
|------|------|------|------|---------|
MANIFEST

EPISODE_COUNT=20
for ep in $(seq -w 1 $EPISODE_COUNT); do
  echo "| EP$ep | [标题] | [时长] | ⏳待导出 | ⏳待审核 |" >> "$OUTPUT_DIR/videos/manifest.md"
done

echo "✅ 视频管理目录已创建"
```

```bash
# [GIT]
cd "$PROJECT_DIR"
git add -A
git commit -m "feat(phase-9): 小云雀2.0视频制作流程完成

- 项目创建 & 脚本导入指引
- 逐集审核清单 (画面/声音/叙事/规范)
- 问题修复 & 重新生成流程
- 视频清单 & 管理目录"
```

> **Phase 9 验收**: 所有剧集 AI 生成完成 ✅ | 画面风格统一 ✅ | 角色形象一致 ✅ | 每集审核通过 ✅ | 视频文件已导出 ✅

---

## Phase 10: 最终发布 & 归档

> **[SHELL]** + **[GIT]** + **[WRITE]** 最终整理 → 完整归档 → 发布准备

### Step 10.1 — 最终文件整理

```bash
# [SHELL] 生成最终交付包
source /tmp/sop_novel.env

# 创建交付目录
DELIVERY_DIR="$OUTPUT_DIR/delivery_$(date +%Y%m%d)"
mkdir -p "$DELIVERY_DIR/小说"
mkdir -p "$DELIVERY_DIR/短剧剧本"
mkdir -p "$DELIVERY_DIR/视频"
mkdir -p "$DELIVERY_DIR/元数据"

# 复制小说终稿
cp "$OUTPUT_DIR/novel_final_with_toc.md" "$DELIVERY_DIR/小说/" 2>/dev/null

# 复制分集剧本
cp -r "$SCRIPTS_DIR/episodes" "$DELIVERY_DIR/短剧剧本/" 2>/dev/null

# 复制视频文件
cp -r "$OUTPUT_DIR/videos/final" "$DELIVERY_DIR/视频/" 2>/dev/null

# 复制元数据
cp -r "$NOVEL_DIR/characters" "$DELIVERY_DIR/元数据/" 2>/dev/null
cp -r "$NOVEL_DIR/outlines" "$DELIVERY_DIR/元数据/" 2>/dev/null
cp -r "$NOVEL_DIR/worldbuilding" "$DELIVERY_DIR/元数据/" 2>/dev/null

# 生成交付包 README
cat > "$DELIVERY_DIR/README.md" << 'README'
# 《[小说标题]》完整创作包

## 基本信息
- **创作日期**: [日期]
- **小说字数**: [N] 字
- **章节数**: [N] 章
- **短剧集数**: [N] 集
- **创作工具**: Claude Code (AI 写作) + 小云雀 2.0 (AI 视频)

## 文件说明

| 目录 | 内容 |
|------|------|
| 小说/ | 完整小说终稿 (含目录) |
| 短剧剧本/ | 逐集剧本 (小云雀适配格式) |
| 视频/ | 短剧成品视频 |
| 元数据/ | 人物设定、世界观、大纲、伏笔矩阵等 |
README

echo "✅ 最终交付包已准备: $DELIVERY_DIR"
```

### Step 10.2 — 发布平台适配指南

```bash
# [WRITE] 创建发布指南
source /tmp/sop_novel.env
cat > "$OUTPUT_DIR/publishing_guide.md" << 'PUBLISH'
# 短剧发布平台指南

## 抖音
- **发布方式**: 合集功能 (按顺序发布，自动串联)
- **单集时长**: ≤3分钟
- **账号要求**: 无粉丝门槛
- **变现方式**: 星图广告 / 短剧付费 / 直播带货
- **封面尺寸**: 1080×1920 (3:4 竖图)
- **标题字数**: ≤55字符
- **最佳发布时间**: 12:00 / 18:00 / 21:00
- **标签建议**: #短剧 #热门短剧 #[题材标签]

## 快手
- **发布方式**: 短剧频道 / 合集
- **单集时长**: ≤3分钟
- **账号要求**: 需申请短剧权限
- **变现方式**: 快手短剧激励 / 直播带货 / 磁力聚星
- **封面尺寸**: 1080×1920
- **特色**: 老铁文化，接地气的内容更有优势

## 视频号
- **发布方式**: 合集 / 单集
- **单集时长**: ≤30分钟
- **账号要求**: 需认证
- **变现方式**: 创作分成 / 互选广告
- **优势**: 微信生态联动 (朋友圈/公众号/社群)

## B站
- **发布方式**: 合集 / 分P
- **单集时长**: 不限
- **账号要求**: 无门槛
- **变现方式**: 创作激励 / 充电 / 广告分成
- **用户特点**: 更注重剧情深度和制作质量

## 红果短剧
- **发布方式**: 专辑发布
- **单集时长**: 1-3分钟
- **审核**: 较严格，需剧情完整、无敏感内容
- **变现方式**: 付费解锁 / 广告分成
- **优势**: 专门的短剧平台，用户付费意愿高

## 跨平台发布建议
1. 先在 1-2 个平台试水，根据数据反馈优化后面剧集
2. 不同平台使用不同的封面和标题 (适配各平台调性)
3. 同一账号不要在不同平台发布完全相同的内容 (可能被判定搬运)
4. 保留 3-5 集延迟在其他平台发布 (平台独家期)
PUBLISH
```

### Step 10.3 — 全流程数据总结

```bash
# [SHELL] 生成创作报告
source /tmp/sop_novel.env

cat > "$OUTPUT_DIR/creation_report.md" << 'REPORT'
# 创作数据报告

## 时间统计
- 创作开始: (Phase 0 时间)
- 创作完成: (Phase 10 时间)
- 总投资时间: [手动填写]

## 字数统计
- 小说总字数: [N] 字
- 大纲总字数: [N] 字
- 人设总字数: [N] 字
- 剧本总字数: [N] 字

## 章节统计
- 总章节数: [N]
- 平均每章字数: [N]
- 最长章: 第N章 ([N]字)
- 最短章: 第N章 ([N]字)

## 剧集统计
- 总集数: [N]
- 单集平均时长: [N] 分钟
- 审核通过率: [N]%

## Git 版本统计
- 总提交数: [N]
- 分支: main
- 标签: v1.0.0-final

## 创作心得
[用户手动填写: 本次创作的经验和教训]
REPORT

echo "✅ 创作报告已生成: $OUTPUT_DIR/creation_report.md"
```

### Step 10.4 — 最终归档

```bash
# [GIT] 最终提交 & 打标签
source /tmp/sop_novel.env
cd "$PROJECT_DIR"
git add -A
git commit -m "feat(phase-10): 最终发布准备 & 全流程归档

🎉 短篇小说创作 & 短剧制作全流程完成!

- 小说终稿
- 短剧剧本
- 视频成品
- 交付包装备
- 发布平台指南

Created with Claude Code + 小云雀 2.0"

# 打版本标签
git tag -a "v1.0.0-final" -m "小说终稿 & 短剧成品"

echo ""
echo "🎉 ========================"
echo "   全流程创作完成!"
echo "   ========================"
echo ""
echo "📦 最终交付包: $DELIVERY_DIR"
echo "📖 小说终稿: $OUTPUT_DIR/novel_final_with_toc.md"
echo "🎬 视频目录: $OUTPUT_DIR/videos/final/"
echo "📊 创作报告: $OUTPUT_DIR/creation_report.md"
echo "📋 发布指南: $OUTPUT_DIR/publishing_guide.md"
echo ""
echo "🚀 下一步行动建议:"
echo "   1. 审核全部视频成品"
echo "   2. 制作每集封面图 (建议统一模板)"
echo "   3. 剪裁1-2个预告片 (选取最精彩片段)"
echo "   4. 选择首发平台 (抖音/快手/视频号/B站/红果短剧)"
echo "   5. 制定发布计划 (建议日更 1-2 集)"
echo "   6. 准备推广素材 (预告片/切片/海报/文案)"
echo "   7. 建立粉丝社群 (微信群/QQ群)"
```

> **Phase 10 验收**: 交付包完整 ✅ | 创作报告生成 ✅ | Git 标签已打 ✅ | 发布指南就绪 ✅ | 全流程归档 ✅

---

## 📚 附录

### A. 命令速查

```bash
# 进度查询
source /tmp/sop_novel.env && bash "$PROJECT_DIR/logs/progress.sh"

# 查看某章内容
source /tmp/sop_novel.env && cat "$NOVEL_DIR/chapters/ch01_opening.md"

# 查看大纲
source /tmp/sop_novel.env && cat "$NOVEL_DIR/outlines/01_three_act_structure.md"

# 查看人物
source /tmp/sop_novel.env && cat "$NOVEL_DIR/characters/01_main_characters.md"

# 统计总字数
source /tmp/sop_novel.env && cat "$NOVEL_DIR/chapters"/*.md | grep -oP '[\x{4e00}-\x{9fff}]' | wc -l

# 查看 Git 历史
source /tmp/sop_novel.env && cd "$PROJECT_DIR" && git log --oneline -20

# 合并全篇
source /tmp/sop_novel.env && cat "$NOVEL_DIR/chapters"/*.md > /tmp/full_novel.md
```

### B. 常见问题速查

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 字数不够 | 大纲过于简略/场景描写不足 | 为关键场景增加 500-800 字细节描写；增加次要人物支线；扩充情感场景 |
| 字数超标 | 对话冗余/描述过度 | Phase 7 删减非核心场景；合并相似情节；精简形容词和副词 |
| 节奏拖沓 | 连续多章无冲突升级 | 在每章插入新的障碍或信息；提高反派活跃度；加入倒计时元素 |
| 人物扁平 | 缺乏内部冲突/成长 | 重写人物小传；为角色添加致命缺陷；增加内心独白展现矛盾 |
| 伏笔忘记回收 | 创作周期长导致遗忘 | 每写完一章对照「伏笔追踪矩阵」勾选；Phase 7 全局扫描 |
| 对话生硬 | 角色同质化/缺乏个性 | 为每个角色建立「对白风格卡」；朗读对话检查自然度 |
| 短剧对白过长 | 单句超过 20 字 | 拆分长句为短句交锋；减少一句话的信息量 |
| 小云雀生成效果差 | AI 提示不够具体 | 增加「视觉提示」细节描述；调整风格参数；多次重新生成去随机性 |
| 人物形象不统一 | AI 每次生成独立无记忆 | 在每集剧本最前面附上角色视觉参考描述；使用小云雀的角色锁定功能 |
| 视频审核不通过 | 内容违规 | 检查敏感词；调整敏感情节；添加合规声明；确保符合平台社区规范 |
| AI 生成乱码/变形 | 复杂画面超出模型能力 | 简化画面元素；减少人物数量；避免复杂手势；分多镜头呈现 |
| 剧集间情绪断层 | 分集时未考虑情绪衔接 | 每集结尾的情绪与下一集开头呼应；在剧本添加「上集回顾」和「下集预告」 |

### C. 创作文件依赖关系

```
Phase 1 (定位与题材)
  └─→ Phase 2 (三幕结构与大纲) ← 依赖 Phase 1
       └─→ Phase 3 (人物与世界观) ← 依赖 Phase 2
            └─→ Phase 4 (第一幕) ← 依赖 Phase 2+3
                 └─→ Phase 5 (第二幕) ← 依赖 Phase 4
                      └─→ Phase 6 (第三幕) ← 依赖 Phase 5
                           └─→ Phase 7 (润色) ← 依赖 Phase 4+5+6
                                └─→ Phase 8 (短剧剧本) ← 依赖 Phase 7
                                     └─→ Phase 9 (视频制作) ← 依赖 Phase 8
                                          └─→ Phase 10 (归档发布)
```

### D. 短剧剧本格式完整示例

```
---
episode: 01
title: "七年后再遇"
---

【开场 — 5秒钩子】
*咖啡厅门口，女主推门而入，与男主四目相对*

女主 (震惊): "七年了...你竟然在这?"

【场景1 — 咖啡厅内 — 日】
*阳光透过落地窗洒在原木桌面上，空气中弥漫着咖啡香*

男主 (放下咖啡杯，眼神复杂): "我一直在等这一天。"

女主 (冷笑): "等我? 当年是你先离开的。"

*女主在男主对面坐下，动作僵硬，手指捏紧包带*

男主: "我有苦衷。"

女主 (猛地站起，椅子发出刺耳的摩擦声): "苦衷? 你知道这七年我是怎么过的吗?"

*周围客人侧目。女主意识到自己失态，缓缓坐下，声音压低*

女主 (咬牙，压低声音): "给我一个解释。"

男主 (从西装内口袋掏出一张泛黄的照片，推过桌面): "先看看这个。"

*女主拿起照片，表情从愤怒 → 震惊 → 难以置信，手指开始颤抖*

女主: "这是...怎么可能..."

【卡点 — 最后5秒】
*照片特写拉近: 上面是女主和...一个和她长得一模一样的女人*

男主 (画外音，低沉): "你从来就不是你以为的那个人。"

---
*本集目标: 建立重逢 + 抛出身份悬念 + 制造强烈好奇*
*下集预告: 照片中的女人是谁? 男主七年去哪了?*
```

### E. 小云雀 2.0 提示词优化指南

```
## 画面提示词公式
[角色描述 + 服装] + [场景描述 + 光线] + [动作 + 表情] + [镜头语言]

示例:
"一个25岁的都市女性，穿着白色衬衫和黑色西装裤，
站在高层办公室的落地窗前，夕阳逆光勾勒出轮廓。
她低头看着手机，表情从平静逐渐变为震惊。
中景镜头，缓慢推近至面部特写。"

## 情绪提示词库
- 震惊: 瞳孔放大、嘴唇微张、后退半步、手中物品滑落
- 甜蜜: 嘴角不自觉上扬、眼神温柔如水、脸颊微红
- 愤怒: 眉头紧锁、握紧拳头至指节发白、呼吸急促
- 悲伤: 眼泪无声滑落、肩膀微微颤抖、低头不语
- 紧张: 手心出汗、不停踱步、频繁看表、舔嘴唇

## 避免的 AI 常见问题
❌ 多人同框 → 分镜头单独拍摄
❌ 复杂手部动作 → 简化或省略，用道具遮挡
❌ 文字出现在画面中 → 用后期字幕代替
❌ 快速运动镜头 → 改用静态画面 + 镜头推拉
❌ 过于抽象的情感描写 → 转化为具体动作和表情

## 提升 AI 生成质量的技巧
✅ 每个场景开头重复角色外观描述
✅ 使用明确的镜头指令 (近景/中景/远景/特写)
✅ 光线描述要具体 (逆光/侧光/柔光/顶光)
✅ 环境氛围要有色彩提示 (暖黄/冷蓝/昏暗/明亮)
```

### F. 发布排期 & 数据追踪模板

```markdown
# 短剧发布排期表

| 日期 | 星期 | 集数 | 发布时间 | 推广动作 | 预计流量 |
|------|------|------|---------|---------|---------|
| D1 | 周一 | EP01-02 | 18:00 | 发布预告片 | — |
| D2 | 周二 | EP03-04 | 18:00 | 评论区互动 | — |
| D3 | 周三 | EP05-06 | 18:00 | 投放抖加/粉条 | — |
| D4 | 周四 | EP07-08 | 18:00 | 切片分发 | — |
| D5 | 周五 | EP09-10 | 18:00 | 发起话题挑战 | — |
| D6 | 周六 | EP11-13 | 12:00 | 直播预热 | — |
| D7 | 周日 | EP14-15 | 12:00 | 数据复盘 | — |

## 数据指标追踪
| 指标 | 目标 | 实际 | 差距 |
|------|------|------|------|
| 总播放量 | 100万+ | | |
| 追剧率 | >40% | | |
| 完播率 | >60% | | |
| 互动率 (点赞+评论+转发) | >5% | | |
| 付费转化率 | >3% | | |
| 粉丝增长 | >5000 | | |
```

---

> **SOP 版本**: 1.0.0 | **最后更新**: 2026-06-11
> **创作者**: Claude Code + 用户协作
> **技术栈**: Claude Code (AI 写作) → 小云雀 2.0 (AI 视频生成)
> **适用平台**: 抖音 / 快手 / 视频号 / B站 / 红果短剧
> **关联文档**: 小说终稿 → 分集剧本 → 短剧视频
