---
name: repo-roast
description: Opinionated, semantic code repository review with parallel sub-agent analysis across architecture, security, performance, readability, and engineering dimensions. This skill should be used when the user asks to "review a repo", "roast my code", "audit this project", "how good is this codebase", "rate this repository", "代码审查", "锐评", "review this repo", or wants a structured quality assessment of any Git repository with an opinionated, no-BS tone.
version: 0.1.0
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

## 输入解析

从用户输入中提取以下参数：

| 参数 | 必填 | 说明 | 默认值 |
|------|------|------|--------|
| repo | 是 | GitHub URL 或本地绝对路径 | — |
| --flavor | 否 | 审查视角：google / startup / oss-maintainer / default | default |
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
1. 读取 `prompts/scout.md` 获取完整侦察指令
2. 严格按照其中的步骤执行：项目身份 → 目录结构 → 构建配置 → CI 配置 → Git 历史 → 代码统计
3. 工具调用上限 10 次
4. 输出结构化 JSON（字段定义见 `prompts/scout.md`）

**根据画像决定**：
- **Flavor**：用户指定 → 用用户指定的；未指定 → 根据 scout 的 `flavor_hint` 自动推断
- **深度和文件范围**：

  | 代码行数 | 深度 | 审查范围 |
  |----------|------|----------|
  | < 5,000 | deep | 全部源文件 |
  | 5,000 – 50,000 | standard | 核心模块（排除 node_modules、vendor、.git、dist、build） |
  | > 50,000 | quick | 热点文件 top 20 + 入口文件 + 配置文件 |

  热点文件清单来自 scout 的 `hotspots` 字段。如果为空，回退到：入口文件 + 配置文件 + 按行数排序的前 20 个源文件。

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
   - 审查文件范围（根据深度决定）
   - 严格的 JSON 输出格式要求

3. 并行启动所有 Sub-Agent（使用 Agent 工具，label 为对应名称）

4. 等待所有 Sub-Agent 完成。120 秒软超时——超时的维度标记为 `timeout`，不阻塞报告生成。

**结果收集**：
- 每个 Sub-Agent 的输出应该是严格的 JSON
- 解析 JSON，验证必要字段存在、severity 值合法（critical/high/medium/low/info）、score 为 S/A/B/C/D
- 如果 JSON 解析失败，尝试从输出文本中提取 JSON 块。仍失败则标记为 `parse_error`
- 将各维度结果聚合为 `{ reviews: { architecture: {...}, security: {...}, ... } }`

## Phase 3: Editor（主笔）

**目标**：综合所有审查结果，生成一份有观点的锐评报告。

**步骤**：

1. 读取 `prompts/editor.md` 获取完整的汇总指令和评分矩阵
2. 读取选定的 `flavors/{flavor}.md`（默认 `flavors/default.md`）获取语气和权重
3. 读取 `templates/report.md` 获取报告结构
4. 执行汇总：
   - 计算综合评分（评分矩阵见 `prompts/editor.md`）
   - 从所有维度聚合 issue，按 severity 排序，选 Top 10
   - 聚合所有 highlights，去重，分组
   - 生成一句话毒舌点评
   - 写出 Top 3 处方
5. 按模板生成完整 Markdown 报告
6. 保存到 `{repo-path}/REPO_ROAST_REPORT.md`
7. 在对话中输出完整报告

**幻觉防线**：输出前抽查 2-3 个 issue 引用的文件是否在仓库中存在。不存在则删除该条。

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

### 模板
- `templates/report.md` — 报告 Markdown 模板
