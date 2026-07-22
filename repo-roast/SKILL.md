---
name: repo-roast
description: Opinionated, semantic code repository review with parallel sub-agent analysis across architecture, security, performance, readability, and engineering dimensions. This skill should be used when the user asks to "review a repo", "roast my code", "audit this project", "how good is this codebase", "rate this repository", "代码审查", "锐评", "review this repo", or wants a structured quality assessment of any Git repository with an opinionated, no-BS tone.
---

# Repo-Roast

一个有观点、有审美、有脾气的代码仓库锐评 Skill。不是 lint，不是静态分析——这是一个写了 20 年代码、review 过 10000+ PR 的 Tech Lead 在说真话。

## 五条铁律

执行本 skill 的每一步都必须遵守以下铁律。违反任何一条视为审查失败。

1. **有据可依**：每个批评必须带文件名 + 行号。没有证据的批评是诽谤。
2. **有褒有贬**：只喷不夸是情绪发泄。亮点必须认，烂处必须说。
3. **拒绝模糊**：禁止"可能存在""或许可以"。要么确认有问题，要么闭嘴。
4. **对事不对人**：评代码，不评作者。
5. **给出路**：指出问题必须附带修改建议，哪怕是方向性的。

## 证据规范

- `file` 必须是仓库内的相对路径，不能使用绝对路径、URL 或不存在的路径。
- `line` 必须是正整数，并且落在该文件实际行数范围内。文件级问题使用最能代表问题的配置行；找不到行号就不要输出该 issue。
- `description` 必须说明可观察到的代码事实，不要只写原则判断。
- `suggestion` 必须给出可执行的修改方向，不要只写"优化""重构"。

## 输入解析

从用户输入中提取以下参数：

| 参数 | 必填 | 说明 | 默认值 |
|------|------|------|--------|
| repo | 是 | GitHub URL 或本地绝对路径 | — |
| --flavor | 否 | 审查视角：google / startup / oss-maintainer / default | default |
| --tone | 否 | 表达锐度：professional / sharp / savage | sharp |
| --dimensions | 否 | 逗号分隔的审查维度 | 全部 5 个 |
| --depth | 否 | 审查深度：quick / standard / deep | 按仓库规模自动决定 |
| --focus | 否 | 聚焦特定目录或文件 | 全部 |

### 仓库准备

- **远程 URL**：`git clone` 到 `/tmp/repo-roast/{repo-name}`（浅克隆 `--depth 50` 以保留足够的历史用于热点分析）
- **本地路径**：直接使用绝对路径
- 克隆失败则报告错误并停止

## Phase 1: Scout（侦察兵）

**目标**：快速建立项目画像，决定后续审查策略。不读实现代码。

**步骤**：
1. 读取 `prompts/scout.md` 获取完整侦察指令，并把用户的 `--depth`、`--focus` 原样传入 Scout
2. 严格按照其中的步骤执行：项目身份 → 目录结构 → 构建配置 → CI 配置 → Git 历史 → 代码统计
3. 工具调用上限 10 次
4. 输出结构化 JSON（字段定义见 `prompts/scout.md`）

**根据画像决定**：
- **Flavor**：用户指定 → 用用户指定的；未指定 → 根据 scout 的 `flavor_hint` 自动推断
- **Tone**：用户指定 → 使用指定值；未指定 → 使用 `sharp`。Tone 只影响 Editor 表达，不进入 Scout 或 Deep Dive
- **深度和文件范围**：Scout 必须按 `prompts/scout.md` 的确定性规则输出 `review_scope`：

  | 代码行数 | 深度 | 审查范围 |
  |----------|------|----------|
  | < 5,000 | deep | 全部源文件 |
  | 5,000 – 50,000 | standard | 入口、公共 API、配置、热点、核心业务目录，按规则最多 40 个文件 |
  | > 50,000 | quick | 热点文件 top 20 + 入口文件 + 配置文件 |

  `--focus` 优先于自动范围，只允许把 focus 内的源文件或配置加入 `included_files`；focus 外配置和元数据只能用于理解上下文。最终清单必须去重、按路径排序且全部存在。五个 Deep Dive 共享同一个 `review_scope`，不得各自扩张范围。

## Phase 2: Deep Dive（深入审查）

**目标**：5 个独立 Sub-Agent 并行审查，各负责一个维度。

**步骤**：

1. 根据 `--dimensions` 参数（或全部 5 个）确定要启动的维度：

   - `roast-arch` → 架构审查 → `prompts/architect.md` + `rubrics/architecture.md`
   - `roast-sec` → 安全审查 → `prompts/security.md` + `rubrics/security.md`
   - `roast-perf` → 性能审查 → `prompts/performance.md` + `rubrics/performance.md`
   - `roast-read` → 可读性审查 → `prompts/readability.md` + `rubrics/readability.md`
   - `roast-eng` → 工程化审查 → `prompts/engineering.md` + `rubrics/engineering.md`

2. 为每个维度构造自包含的 task，包含：
   - 对应 `prompts/{dimension}.md` 的完整内容
   - 对应 `rubrics/{dimension}.md` 的完整内容
   - Phase 1 输出的项目画像 JSON
   - 仓库的绝对路径
   - Phase 1 输出的完整 `review_scope`（五个维度使用完全相同的 `included_files`）
   - 严格的 JSON 输出格式要求

3. 构造 task prompt 的模板：

   ```
   你是 {dimension} 审查专家。

   ## 项目信息
   {scout JSON 内容}

   ## 仓库路径
   {repo absolute path}

   ## 唯一审查范围
   {scout.review_scope JSON 内容}

   只审查 review_scope.included_files 中的实现或配置。范围外的配置和元数据只能用于上下文理解，不能静默扩展 finding 范围。

   ## 审查指令
   {完整读取 prompts/{dimension}.md 内容}

   ## 评分标准
   {完整读取 rubrics/{dimension}.md 内容}

   ## 输出要求
   只输出 JSON，不要有任何其他文字。JSON 必须符合 `schemas/review.schema.json`。
   每个 issue 必须包含 file 和 line 字段，且 file 必须存在、line 必须落在文件行数范围内。
   只陈述中性、可验证的技术事实。不要读取 rhetoric 目录，不要生成锐评、笑话、侮辱或戏剧化表达。
   pattern 必须描述已经确认的问题模式；不得为了匹配语料而猜测。
   ```

4. 并行启动所有 Sub-Agent（使用 Agent 工具，label 为对应名称）

5. 等待所有 Sub-Agent 完成。120 秒软超时——超时的维度标记为 `timeout`，不阻塞报告生成。

**结果收集**：
- 每个 Sub-Agent 的输出应该是严格的 JSON
- 解析 JSON，并按 `schemas/review.schema.json` 验证必要字段、枚举值和字段结构；每个 issue 的 file/line 还必须在仓库中实际可定位
- 如果 JSON 解析失败，尝试从输出文本中提取 JSON 块。仍失败则标记为 `parse_error`
- 维护四个互斥状态：`requested_dimensions` 是用户请求（省略则为全部五维），`completed_dimensions` 是通过 Schema 的请求维度，`failed_dimensions` 是超时/解析/验证失败的请求维度，`skipped_dimensions` 是未请求维度
- 必须满足：requested = completed ∪ failed，requested ∩ skipped = ∅，requested ∪ skipped = 固定五维；每个 failed 都有对应 `review_errors`
- 将画像、结果、错误、四种维度状态、Flavor 和 Tone 聚合为符合 `schemas/report-input.schema.json` 的 Editor 输入

**输出格式验证示例**（架构维度）：
```json
{
  "dimension": "architecture",
  "score": "B",
  "summary": "基本分层但存在循环依赖",
  "issues": [
    {
      "severity": "high",
      "file": "src/utils.ts",
      "line": 42,
      "title": "工具模块形成反向依赖",
      "description": "utility 模块反向依赖 service 层",
      "impact": "底层工具与业务层形成耦合，修改 service 时会扩大影响范围",
      "suggestion": "提取共享类型到独立模块",
      "pattern": "boundary-violation"
    }
  ],
  "highlights": [
    {
      "file": "src/index.ts",
      "line": 8,
      "title": "入口职责单一",
      "description": "入口文件职责清晰，仅做引导不做业务"
    }
  ],
  "stats": {
    "module_count": 12,
    "circular_dep_pairs": 1,
    "max_dependents": "src/utils.ts",
    "god_object_count": 0
  }
}
```

## Phase 3: Editor（主笔）

**目标**：综合所有审查结果，生成一份有观点的锐评报告。

**步骤**：

1. 读取 `prompts/editor.md` 获取完整的汇总指令和评分矩阵
2. 读取选定的 `flavors/{flavor}.md`（默认 `flavors/default.md`）获取审查立场、权重和优先级
3. 读取 `rhetoric/tones/{tone}.md`（默认 `rhetoric/tones/sharp.md`）获取表达锐度
4. 读取 `rhetoric/principles.md`、`rhetoric/banned.md`、对应五维语料和 `rhetoric/transitions.json`
5. 读取 `templates/report.md` 获取报告结构
6. 执行汇总：
   - 计算综合评分（评分矩阵见 `prompts/editor.md`）
   - 从所有维度聚合 issue，按 severity 排序，选 Top 10
   - 聚合所有 highlights，去重，分组
   - 在全部问题完成证据验证后，按 dimension → pattern → severity → tone 匹配可选锐评
   - 生成基于已验证结论的一句话点评
   - 写出 Top 3 处方
7. 按模板生成完整 Markdown 报告
8. 保存到 `{repo-path}/REPO_ROAST_REPORT.md`
9. 在对话中输出完整报告

**幻觉防线**：输出前逐条验证全部正式 issue 的文件、行号和引用事实。不存在、超出范围或无法定位证据的 issue 必须从正式问题列表删除，不得通过降级严重度继续保留。修辞只能写入生成的 `roast` 字段，不得改变 severity、file、line、title、description、impact、suggestion 或 pattern。

**错误处理**：
- 某维度超时 → 报告中标注"(未完成)"，不影响其他维度
- JSON 解析失败 → 尝试从文本中提取 JSON，失败则标记 `parse_error`
- 某维度无 issues → 正常，可能是好信号，如实反映
- Tone 文件缺失 → 回退到 sharp；sharp 也缺失则输出纯技术描述
- pattern 无匹配语料 → 保留技术 finding，不生成锐评
- 全部维度失败 → 输出错误报告，说明原因，建议重试

## 评分速查

| 等级 | 含义 |
|------|------|
| **S** | 该维度几乎完美，可作为参考实现 |
| **A** | 优秀，偶有小瑕疵 |
| **B** | 良好，有明显改进空间 |
| **C** | 及格，问题较多 |
| **D** | 不及格，需要重构 |

综合评分矩阵详见 `prompts/editor.md`。

## 参考文件地图

### Prompts
- `prompts/scout.md` — Phase 1 项目侦察指令
- `prompts/architect.md` — 架构审查 Agent 指令
- `prompts/security.md` — 安全审查 Agent 指令
- `prompts/performance.md` — 性能审查 Agent 指令
- `prompts/readability.md` — 可读性审查 Agent 指令
- `prompts/engineering.md` — 工程化审查 Agent 指令
- `prompts/editor.md` — Phase 3 汇总主笔指令

### 评分标准
- `rubrics/architecture.md` — 架构 S/A/B/C/D 标准
- `rubrics/security.md` — 安全评分标准
- `rubrics/performance.md` — 性能评分标准
- `rubrics/readability.md` — 可读性评分标准
- `rubrics/engineering.md` — 工程化评分标准

### 视角模板
- `flavors/default.md` — 老炮工程师（默认）
- `flavors/google.md` — Google 工程文化
- `flavors/startup.md` — 硅谷创业公司
- `flavors/oss-maintainer.md` — 开源维护者

### 锐评修辞
- `rhetoric/principles.md` — 事实与修辞边界
- `rhetoric/banned.md` — 禁止表达
- `rhetoric/tones/*.md` — professional / sharp / savage 表达等级
- `rhetoric/patterns/*.json` — 五维结构化锐评语料
- `rhetoric/transitions.json` — 事实、影响和建议的连接语

### 模板
- `templates/report.md` — 报告 Markdown 模板
