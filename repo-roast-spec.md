# 🔥 Repo-Roast Skill — 详细设计文档

> **历史文档**：本文记录 v0.3 系列方案。v0.4.0 的数据契约、审查状态和 Flavor/Tone 分离设计以 `repo-roast/V0.4-DESIGN.md`、各 JSON Schema 与当前 `SKILL.md` 为准。

> 一个有观点、有审美、有脾气的代码仓库锐评 Skill。
> 支持 Claude Code Sub-Agent 并行审查，输出结构化锐评报告。

---

## 一、项目概述

### 1.1 定位

Repo-Roast 不是 lint 工具，不是静态分析。它是一个**写了 20 年代码、review 过 10000+ PR 的 Tech Lead**，看完你的仓库后说真话。

### 1.2 目标用户

- 想快速了解一个开源项目质量的开发者
- 团队 Lead 审查成员或新人的代码
- 开源维护者想获得第三方视角的项目体检
- 技术选型前评估候选项目

### 1.3 与现有工具的差异

| 工具 | 做什么 | 缺什么 |
|------|--------|--------|
| SonarQube | 静态规则扫描 | 没有语义理解，不会说人话 |
| CodeScene | 热点分析 | 不读代码内容 |
| GitHub Copilot | 补全代码 | 不看全局 |
| ESLint/Prettier | 格式 + 简单规则 | 只管对不对，不管好不好 |
| **Repo-Roast** | 语义级审查 + 有观点 + 带人格 | — |

---

## 二、核心设计原则

### 2.1 五条铁律

写入 SKILL.md 开头，每次执行时生效：

```
铁律一：有据可依。每个批评必须带文件名 + 行号。没有证据的批评 = 诽谤。
铁律二：有褒有贬。只喷不夸 = 情绪发泄。亮点必须认，烂处必须说。
铁律三：拒绝模糊。禁止"可能存在""或许可以"。要么确认有问题，要么闭嘴。
铁律四：对事不对人。评代码，不评作者。
铁律五：给出路。指出问题必须附带修改建议，哪怕是方向性的。
```

### 2.2 设计哲学

- **Token 经济**：Claude Code 的上下文窗口是公共资源。每个 prompt 都要精简，只给模型不知道的信息。
- **高自由度优先**：审查判断需要上下文，不宜过度规定。给方向和评分标准，不给死板的检查清单。
- **Sub-Agent 天然适配**：审查本身就是分维度的，天然适合并行。

---

## 三、功能详述

### 3.1 输入

| 参数 | 必填 | 说明 | 示例 |
|------|------|------|------|
| repo | ✅ | GitHub URL 或本地路径 | `https://github.com/tanweai/pua` |
| --flavor | ❌ | 审查视角 | `google` / `startup` / `oss-maintainer` / `default` |
| --dimensions | ❌ | 指定审查维度（逗号分隔） | `security,performance` |
| --depth | ❌ | 审查深度 | `quick` / `standard` / `deep` |
| --focus | ❌ | 聚焦特定目录或文件 | `src/core/` |

### 3.2 三阶段流程

```
用户输入仓库地址
       │
       ▼
┌──────────────────┐
│  Phase 1: Scout   │  ← 单 Agent，≤10 次工具调用建立项目画像
└────────┬─────────┘
         │ 输出：项目画像 JSON
         ▼
┌──────────────────────────────────────────┐
│  Phase 2: Deep Dive                       │
│                                           │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐     │
│  │ 🏗️ Arch  │ │ 🔒 Sec   │ │ ⚡ Perf  │  ← Sub-Agent 并行
│  └─────────┘ └─────────┘ └─────────┘     │
│  ┌─────────┐ ┌─────────┐                 │
│  │ 📖 Read  │ │ 📂 Eng   │  ← Sub-Agent 并行
│  └─────────┘ └─────────┘                 │
└────────┬─────────────────────────────────┘
         │ 各维度评分 + 问题清单
         ▼
┌──────────────────┐
│  Phase 3: Editor  │  ← 单 Agent，汇总写报告
└────────┬─────────┘
         │
         ▼
   锐评报告（Markdown）
```

### 3.3 Phase 1: Scout（侦察兵）

**目标**：快速建立项目画像，决定后续审查策略。

**执行步骤**：
1. 读 README / CONTRIBUTING / CHANGELOG — 项目是什么、给谁用
2. `ls -la` + `tree -L 2` — 目录结构
3. 读构建配置（package.json / Cargo.toml / go.mod / pom.xml）
   - 依赖数量、版本策略、有无 lockfile
4. 读 CI 配置（.github/workflows / Jenkinsfile / .gitlab-ci.yml）
5. `git log --oneline -30` — 最近活跃度
6. `git log --stat --since="3 months ago" | head -100` — 高频改动文件
7. 统计：语言、文件数、代码行数、测试文件占比

**输出格式**（JSON，给 Phase 2 用）：
```json
{
  "project": {
    "name": "pua",
    "description": "AI Coding Agent PUA Skill",
    "language": "Shell/Markdown",
    "framework": "Claude Code Plugin",
    "type": "cli-tool"
  },
  "scale": {
    "files": 42,
    "loc": 3200,
    "test_ratio": 0.05,
    "has_tests": false
  },
  "structure": {
    "verdict": "meh",
    "depth": 3,
    "notes": ["skills/ 和 hooks/ 分离合理，但 hooks/ 缺文档"]
  },
  "dependencies": {
    "count": 0,
    "risk_level": "low",
    "notes": ["纯 Shell + Markdown，无外部依赖"]
  },
  "ci": {
    "exists": true,
    "tools": ["GitHub Actions"],
    "coverage": ["lint", "test"]
  },
  "hotspots": ["hooks/pua-hook.sh", "skills/high-agency/SKILL.md"],
  "flavor_hint": "oss-maintainer",
  "first_impression": "精巧的单用途工具，结构清晰但缺测试"
}
```

**Phase 1 铁律**：
- 不读具体代码逻辑，只建立画像
- 控制在 10 次工具调用以内
- 输出必须是 JSON，为 Phase 2 提供结构化输入

### 3.4 Phase 2: Deep Dive（深入审查）

**核心设计：Sub-Agent 并行**

这是本 Skill 最关键的架构决策。Phase 2 由 5 个独立的 Sub-Agent 并行执行，每个负责一个审查维度。

#### Sub-Agent 调度方式

在 Claude Code 中，通过 `sessions_spawn` 启动 5 个 Sub-Agent：

```
sessions_spawn(
  task: "架构审查 prompt + Phase 1 JSON + 仓库代码",
  label: "roast-arch"
)
// 同时启动其余 4 个...
```

每个 Sub-Agent：
- 独立的上下文窗口（不会互相污染）
- 只接收自己需要的维度 prompt + Phase 1 画像
- 输出结构化的审查结果 JSON
- 可以自由读取仓库中的任何文件

#### 5 个审查维度

##### 🏗️ 架构审查（Architect Agent）

**审查内容**：
1. 模块依赖关系（import/require 图）
2. 分层清晰度（controller → service → dao 有没有越层）
3. 循环依赖检测
4. 扩展性评估（加一个功能要改几个文件？）
5. 与同类优秀项目的结构对比

**输出**：
```json
{
  "dimension": "architecture",
  "score": "B",
  "summary": "模块划分基本合理，但 core/ 和 utils/ 之间存在双向依赖",
  "issues": [
    {
      "severity": "high",
      "file": "src/core/parser.js",
      "line": 42,
      "description": "直接 import 了 utils/format.js，但 utils/format.js 又反向依赖 core/parser.js，形成循环依赖",
      "suggestion": "将 format 中依赖 parser 的逻辑提取到 shared/ 层"
    }
  ],
  "highlights": [
    {
      "file": "src/plugins/",
      "description": "插件系统设计干净，注册机制清晰，扩展性好"
    }
  ]
}
```

##### 🔒 安全审查（Security Agent）

**审查内容**：
1. 硬编码密钥/密码/token（grep + 语义分析）
2. SQL/命令拼接（字符串拼接进查询或 shell 命令）
3. 输入校验（用户输入直接进敏感操作？）
4. 依赖漏洞（已知 CVE）
5. 权限校验逻辑
6. 敏感信息泄露（日志、错误消息、前端暴露）

**输出格式**：同上，`dimension: "security"`

##### ⚡ 性能审查（Performance Agent）

**审查内容**：
1. N+1 查询模式
2. 算法复杂度（不必要的 O(n²) 或更差）
3. 资源泄漏（未关闭连接/文件句柄）
4. 前端：bundle size、渲染阻塞、图片优化
5. 数据库：缺索引的查询
6. 缓存策略缺失

**输出格式**：同上，`dimension: "performance"`

##### 📖 可读性审查（Readability Agent）

**审查内容**：
1. 命名质量（data/temp/result/info 等泛滥命名计数）
2. 函数长度（超过 50 行的函数清单）
3. 注释密度和质量
4. 风格一致性（命名、文件组织）
5. 抽象层次一致性

**输出格式**：同上，`dimension: "readability"`

##### 📂 工程化审查（Engineering Agent）

**审查内容**：
1. 测试覆盖率（目测 + 工具检测）
2. 错误处理质量（空 catch、吞异常）
3. 文档完整性
4. 配置管理（环境变量、配置文件）
5. 发布流程（版本管理、CHANGELOG、tag）
6. Git 工作流（commit message 质量、branch 策略）

**输出格式**：同上，`dimension: "engineering"`

#### Sub-Agent Prompt 模板（通用结构）

每个维度的 prompt 遵循同一模板：

```markdown
你是 {维度} 审查专家。

## 任务
审查以下仓库的 {维度}，输出 JSON 报告。

## 项目画像
{Phase 1 JSON}

## 审查要求
- 每个问题必须带 file + line
- 严重程度：critical / high / medium / low / info
- 评分：S / A / B / C / D
- 同时列出亮点

## 输出格式
严格输出以下 JSON，不要有其他内容：
{
  "dimension": "{维度}",
  "score": "S|A|B|C|D",
  "summary": "一句话总结",
  "issues": [{ "severity", "file", "line", "description", "suggestion" }],
  "highlights": [{ "file", "description" }],
  "stats": { ... }
}
```

#### Sub-Agent 策略细节

**超时控制**：每个 Sub-Agent 设置 120 秒超时。超时的维度标记为 `timeout`，不阻塞报告生成。

**失败处理**：Sub-Agent 失败时，Editor 阶段用 "该维度审查未完成" 占位，不跳过也不编造。

**仓库访问**：Sub-Agent 通过 `cwd` 继承父工作空间。Phase 1 阶段将仓库 clone 到固定路径，Sub-Agent 直接读取。

**Token 控制**：
- 大仓库（>50k 行）：每个 Sub-Agent 只审查高频改动文件 + 入口文件 + 配置文件，不逐文件扫描
- 中等仓库（5k-50k 行）：审查所有核心模块
- 小仓库（<5k 行）：审查全部代码

### 3.5 Phase 3: Editor（主笔）

**目标**：综合 Phase 1 + Phase 2 所有输出，写成一份有观点的锐评报告。

**输入**：
- Phase 1 项目画像 JSON
- 5 个 Sub-Agent 的审查结果 JSON
- 选定的 Flavor 模板

**执行步骤**：
1. 汇总所有维度的 score → 计算综合评分
2. 从所有 issues 中筛选 Top 10（按 severity 排序）
3. 汇总所有 highlights → 写成"亮点"章节
4. 按模板生成完整报告
5. 根据 Flavor 调整语气和侧重点

**评分规则**：
（具体加权算法见 `prompts/editor.md`，此处为简化参考）

| 维度评分组合 | 综合评分 |
|-------------|---------|
| 全部 S | S |
| 多数 A+，无 C 以下 | A |
| 多数 B+，无 D | B |
| 有 D 或两个 C | C |
| 多数 C 以下 | D |

> 注：实际实现采用加权数值算法（S=5, A=4, B=3, C=2, D=1），按 Flavor 权重加权平均后映射回字母等级。以 editor.md 为准。

**Flavor 影响**：
- `google`：强调可读性、测试覆盖率、文档，语气严谨
- `startup`：强调速度和功能完整性，容忍部分工程化缺陷，语气务实
- `oss-maintainer`：强调 CONTRIBUTING 指南、issue 模板、社区友好度，语气友善
- `default`：老炮直率风格，有褒有贬

---

## 四、架构设计

### 4.1 目录结构

```
repo-roast/
├── SKILL.md                    # 入口文件，触发条件 + 主流程编排
├── prompts/
│   ├── scout.md                # Phase 1 prompt
│   ├── architect.md            # Phase 2: 架构审查 prompt
│   ├── security.md             # Phase 2: 安全审查 prompt
│   ├── performance.md          # Phase 2: 性能审查 prompt
│   ├── readability.md          # Phase 2: 可读性审查 prompt
│   ├── engineering.md          # Phase 2: 工程化审查 prompt
│   └── editor.md               # Phase 3: 汇总 prompt
├── flavors/
│   ├── google.md               # Google 工程文化视角
│   ├── startup.md              # 硅谷创业公司视角
│   ├── oss-maintainer.md       # 开源维护者视角
│   └── default.md              # 老炮工程师视角
├── rubrics/
│   ├── architecture.md         # 架构评分标准（S/A/B/C/D 对应什么）
│   ├── security.md             # 安全评分标准
│   ├── performance.md          # 性能评分标准
│   ├── readability.md          # 可读性评分标准
│   └── engineering.md          # 工程化评分标准
├── templates/
│   └── report.md               # 报告 Markdown 模板
└── scripts/
    ├── clone.sh                # 克隆仓库 + 统计信息收集
    └── hotspots.sh             # git log 分析高频改动文件
```

### 4.2 SKILL.md 主流程伪代码

```markdown
# Repo-Roast

## 铁律
{五条铁律}

## 流程

### Step 1: 准备
1. 解析用户输入（repo URL / 本地路径 / 可选参数）
2. 如果是 URL：clone 到 /tmp/repo-roast/{repo-name}
3. 如果是本地路径：直接使用
4. 运行 scripts/clone.sh 收集基础统计

### Step 2: Phase 1 — Scout
1. 读取 prompts/scout.md
2. 执行侦察步骤，输出项目画像 JSON
3. 根据画像决定：
   - flavor（用户指定 or 自动推断）
   - 审查深度（quick/standard/deep）
   - 需要审查的文件范围

### Step 3: Phase 2 — Deep Dive（Sub-Agent 并行）
使用 sessions_spawn 启动 5 个 Sub-Agent：

  sessions_spawn(
    task: "读取 prompts/architect.md，结合以下画像审查仓库：\n{画像JSON}\n仓库路径：{path}",
    label: "roast-arch"
  )
  // 同时启动 security, performance, readability, engineering

等待所有 Sub-Agent 完成（或超时 120s）。

### Step 4: Phase 3 — Editor
1. 读取 prompts/editor.md
2. 收集所有 Sub-Agent 输出
3. 读取选定的 flavor/{flavor}.md
4. 生成锐评报告
5. 保存到 {repo-path}/REPO_ROAST_REPORT.md
6. 同时输出到聊天
```

### 4.3 Sub-Agent 通信协议

```
Parent Agent (SKILL.md)
    │
    │ sessions_spawn(task=...)
    │
    ├── Sub-Agent: roast-arch
    │   └── 输出: prompts/architect.md 定义的 JSON
    ├── Sub-Agent: roast-sec
    │   └── 输出: prompts/security.md 定义的 JSON
    ├── Sub-Agent: roast-perf
    │   └── 输出: prompts/performance.md 定义的 JSON
    ├── Sub-Agent: roast-read
    │   └── 输出: prompts/readability.md 定义的 JSON
    └── Sub-Agent: roast-eng
        └── 输出: prompts/engineering.md 定义的 JSON

所有输出回到 Parent Agent → 传给 Editor prompt → 生成报告
```

**关键约束**：
- Sub-Agent 之间不通信（避免复杂度）
- Sub-Agent 只输出 JSON（结构化，便于 Editor 解析）
- Sub-Agent 不做评分汇总（那是 Editor 的事）
- Sub-Agent 可以读取仓库中的任何文件（通过 cwd 继承）

---

## 五、Flavor 系统

### 5.1 设计目的

同一个仓库，不同视角看到的问题不同。Flavor 控制审查的**价值取向**和**语气风格**。

### 5.2 Flavor 定义

每个 Flavor 文件定义：
1. **权重调整**：哪些维度更重要
2. **语气指南**：措辞风格
3. **特别关注**：该视角特有的检查项
4. **报告模板调整**：章节顺序、标题风格

### 5.3 Flavor 示例

#### `default.md` — 老炮工程师
```markdown
# Default Flavor

## 权重
所有维度等权。

## 语气
直率、有观点。好的说好，烂的说烂。
像一个 Tech Lead 在周五下午给你做 code review。

## 特别关注
- 代码是否"过度设计"或"设计不足"
- 有没有"简历驱动开发"的痕迹（用了一堆新框架但没用到核心特性）
- 依赖是否合理（用了 10 个库做 2 个库能做的事）

## 禁止
- "考虑到项目规模..." — 别找借口
- "这是一个有趣的选择" — 不有趣就说不有趣
```

#### `startup.md` — 硅谷创业公司
```markdown
# Startup Flavor

## 权重
功能完整性 > 工程化 > 架构 > 安全 > 性能

## 语气
务实、速度导向。
"能跑就行"不是罪，但"能跑但没人看得懂"是。

## 特别关注
- MVP 是否可交付
- 技术债是否可控（可以欠，但要知道欠了多少）
- 上线需要补什么（监控、日志、错误追踪）

## 宽容项
- 测试覆盖率低（早期可以接受）
- CI 不完善（但核心流程要有）
```

---

## 六、评分标准（Rubrics）

### 6.1 评分等级

| 等级 | 含义 | 标准 |
|------|------|------|
| **S** | 该维度几乎完美 | 作为参考实现都可以 |
| **A** | 优秀，偶有小瑕疵 | 主流最佳实践遵循度 >90% |
| **B** | 良好，有明显改进空间 | 核心问题 1-2 个 |
| **C** | 及格，问题较多 | 核心问题 3+ 个，但不影响基本功能 |
| **D** | 不及格，需要重构 | 存在严重影响可维护性/安全性的问题 |

### 6.2 各维度评分标准示例

#### 架构评分

| 等级 | 标准 |
|------|------|
| S | 分层清晰、模块职责单一、无循环依赖、扩展性强、有清晰的架构文档 |
| A | 分层合理、偶有职责混杂、无循环依赖、扩展性好 |
| B | 基本分层、部分模块职责不单一、无严重耦合问题 |
| C | 分层模糊、存在循环依赖、部分模块过大 |
| D | 无明显分层、大量耦合、"大泥球"架构 |

---

## 七、与 Claude Code Sub-Agent 的集成

### 7.1 为什么用 Sub-Agent

| 方案 | 优点 | 缺点 |
|------|------|------|
| 单 Agent 串行 | 简单、上下文共享 | Token 消耗大、慢、上下文窗口可能溢出 |
| 多 Agent 并行（Sub-Agent） | 快、独立上下文、不互相污染 | 需要通信协议、实现复杂 |

**结论**：审查任务天然分维度 → 天然适合 Sub-Agent 并行。

### 7.2 Sub-Agent 调用细节

```python
# 伪代码：Claude Code 中的 Sub-Agent 调用

# Phase 2: 启动 5 个并行 Sub-Agent
arch_task = f"""
你是架构审查专家。审查以下仓库的架构设计。
项目画像：{scout_json}
仓库路径：{repo_path}
读取 prompts/architect.md 获取详细指引。
输出严格 JSON，无其他内容。
"""

# 同时 spawn 5 个
spawn(label="roast-arch",   task=arch_task)
spawn(label="roast-sec",    task=sec_task)
spawn(label="roast-perf",   task=perf_task)
spawn(label="roast-read",   task=read_task)
spawn(label="roast-eng",    task=eng_task)

# 等待完成
# Sub-Agent 完成后会自动通知父 Agent
# 设置 120s 超时，超时的维度标记为未完成
```

### 7.3 Sub-Agent Prompt 设计原则

1. **自包含**：Sub-Agent 收到 prompt 后应该能独立完成任务，不需要回问父 Agent
2. **结构化输出**：严格要求 JSON 输出，便于父 Agent 解析
3. **最小上下文**：只给 Sub-Agent 需要的信息（画像 + 维度 prompt），不给其他维度的结果
4. **明确边界**：告诉 Sub-Agent "只审查你的维度，不要越界"

### 7.4 大仓库策略

仓库超过 50k 行时，逐文件审查不现实。策略：

```
if loc > 50000:
    审查范围 = 高频改动文件(top 20) + 入口文件 + 配置文件 + README
    审查深度 = quick
elif loc > 5000:
    审查范围 = 核心模块全部文件
    审查深度 = standard
else:
    审查范围 = 全部代码
    审查深度 = deep
```

高频改动文件通过 `git log --stat` 分析得出，Phase 1 已经收集。

---

## 八、报告输出

### 8.1 输出位置

1. **聊天输出**：完整报告直接输出到对话
2. **文件保存**：同时保存到 `{repo-path}/REPO_ROAST_REPORT.md`
3. **（可选）飞书文档**：通过 feishu_create_doc 创建云文档

### 8.2 报告模板

```markdown
# 🔥 锐评报告：{repo_name}

> {一句话毒舌点评，根据 flavor 调整语气}

## 📊 体检报告

| 维度 | 评分 | 一句话 |
|------|------|--------|
| 🏗️ 架构 | {score} | {summary} |
| 📂 工程化 | {score} | {summary} |
| 🔒 安全 | {score} | {summary} |
| ⚡ 性能 | {score} | {summary} |
| 📖 可读性 | {score} | {summary} |

**综合评分：{overall}** （满分 S）

---

## 💎 亮点

### {亮点 1 标题}
- 📁 `{file}:{line}`
- {说明}

### {亮点 2 标题}
...

---

## 🔴 硬伤（必须改）

### {问题 1 标题} — {severity}
- 📁 `{file}:{line}`
- **问题**：{描述}
- **风险**：{为什么严重}
- **建议**：{怎么改}

---

## 🟡 值得关注

...

---

## 🟢 建议优化

...

---

## 🏥 Top 3 处方

1. **{最该改的}** — {原因} — {建议方案}
2. **{第二}**
3. **{第三}**

---

## 📐 同类参考

{同类型优秀项目是怎么做的，给出 GitHub 链接}

---

_由 Repo-Roast Skill 生成 | Flavor: {flavor} | 审查时间: {date}_
```

---

## 九、开发路线

### Phase 0: 最小可用（P0）
- [ ] SKILL.md（主流程 + 铁律）
- [ ] prompts/scout.md
- [ ] prompts/architect.md（先做 1 个维度验证流程）
- [ ] prompts/editor.md
- [ ] templates/report.md
- **验收**：对一个真实仓库跑通三阶段流程，输出完整报告

### Phase 1: 完整维度（P1）
- [ ] prompts/security.md
- [ ] prompts/performance.md
- [ ] prompts/readability.md
- [ ] prompts/engineering.md
- [ ] rubrics/ 5 个评分标准文件
- **验收**：5 个 Sub-Agent 并行审查，输出完整五维度报告

### Phase 2: Flavor 系统（P2）
- [ ] flavors/default.md
- [ ] flavors/google.md
- [ ] flavors/startup.md
- [ ] flavors/oss-maintainer.md
- [ ] 自动 flavor 推断逻辑
- **验收**：同一仓库用不同 flavor 输出不同风格的报告

### Phase 3: 自动化集成（P3）
- [ ] scripts/clone.sh
- [ ] scripts/hotspots.sh
- [ ] git hook 集成（pre-commit / pre-push）
- [ ] CI 集成（GitHub Actions workflow）
- **验收**：push 代码自动生成锐评报告

### Phase 4: 增强（P4）
- [ ] 历史对比（两次锐评的 diff）
- [ ] 趋势分析（质量是变好还是变差）
- [ ] 自动修复建议（不只是说问题，直接生成 patch）
- [ ] 飞书/Slack 集成（报告推送到频道）

---

## 十、注意事项

### 10.1 Token 消耗控制

- Phase 1 严格限制工具调用次数（≤10 次）
- Phase 2 Sub-Agent 各自控制，互不影响
- 大仓库采用抽样策略，不逐文件扫描
- 报告模板精简，避免冗余描述

### 10.2 幻觉防范

- 铁律一（有据可依）是核心防线
- 每个问题必须有 file + line，否则不收录
- Sub-Agent 输出 JSON 时，父 Agent 做基本校验（file 是否存在、line 是否合理）
- Editor 阶段可以抽查验证

### 10.3 安全考虑

- 不审查私有仓库的敏感内容（密钥等）不外泄
- 审查结果中的密钥/token 自动脱敏
- Sub-Agent 不联网（不发 HTTP 请求），只读本地文件

### 10.4 兼容性

- 支持：JavaScript/TypeScript、Python、Go、Rust、Java、Shell
- 基础支持：任何有文本代码的仓库（fallback 到通用审查）
- 语言特定规则放在各自的 rubric 中

---

_文档版本：v0.1 | 创建时间：2026-05-07_
