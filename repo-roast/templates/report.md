# 🔥 锐评报告：{{repo_name}}

> {{one_line_roast}}

---

## 📊 体检报告

| 维度 | 评分 | 一句话 |
|---|---|---|
| 🏗️ 架构 | {{arch_score}} | {{arch_summary}} |
| 🔒 安全 | {{sec_score}} | {{sec_summary}} |
| ⚡ 性能 | {{perf_score}} | {{perf_summary}} |
| 📖 可读性 | {{read_score}} | {{read_summary}} |
| 📂 工程化 | {{eng_score}} | {{eng_summary}} |

**综合评分：{{overall_score}}** （满分 S）　|　**审查视角：{{flavor}}**　|　**表达锐度：{{tone}}**

- **请求维度**：{{requested_dimensions}}
- **完成维度**：{{completed_dimensions}}
- **失败维度**：{{failed_dimensions}}
- **跳过维度**：{{skipped_dimensions}}

---

## 💎 亮点

{{#each highlights}}
### {{title}}
- 📁 `{{file}}:{{line}}`
- {{description}}
{{/each}}

{{#if no_highlights}}
> 没有足够证据支持特别表扬的实现。
{{/if}}

---

## 🔴 硬伤（必须改）

{{#each critical_issues}}
### {{index}}. {{title}}
{{#if roast}}
> {{roast}}
{{/if}}
- **严重级别**：{{severity}}
- **来源维度**：{{dimension}}
- **位置**：`{{file}}:{{line}}`
- **问题**：{{description}}
- **影响**：{{impact}}
- **建议**：{{suggestion}}

{{/each}}

---

## 🟡 值得关注

{{#each medium_issues}}
### {{index}}. {{title}}
{{#if roast}}
> {{roast}}
{{/if}}
- **来源维度**：{{dimension}}
- **位置**：`{{file}}:{{line}}`
- **问题**：{{description}}
- **影响**：{{impact}}
- **建议**：{{suggestion}}

{{/each}}

{{#if no_medium_issues}}
> 没有需要报告的中等严重度问题。
{{/if}}

---

## 🟢 建议优化

{{#each low_issues}}
### {{index}}. {{title}}
{{#if roast}}
> {{roast}}
{{/if}}
- **来源维度**：{{dimension}}
- **位置**：`{{file}}:{{line}}`
- **问题**：{{description}}
- **影响**：{{impact}}
- **建议**：{{suggestion}}

{{/each}}

{{#if no_low_issues}}
> 没有需要报告的低优先级问题。
{{/if}}

---

## 🏥 Top 3 处方

| # | 处方 | 原因 | 方向 |
|---|---|---|---|
| 1 | {{rx1_title}} | {{rx1_reason}} | {{rx1_solution}} |
| 2 | {{rx2_title}} | {{rx2_reason}} | {{rx2_solution}} |
| 3 | {{rx3_title}} | {{rx3_reason}} | {{rx3_solution}} |

---

## 📐 同类参考

{{similar_projects_reference}}

---

_由 Repo-Roast Skill 生成 | Flavor: {{flavor}} | Tone: {{tone}} | 维度完成: {{completed_dimension_count}}/{{requested_dimension_count}} | 审查时间: {{date}}_
