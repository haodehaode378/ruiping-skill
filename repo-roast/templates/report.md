# 🔥 锐评报告：{{repo_name}}

> {{one_line_roast}}

## 📊 体检报告

| 维度 | 评分 | 一句话 |
|------|------|--------|
| 🏗️ 架构 | {{arch_score}} | {{arch_summary}} |
| 📂 工程化 | {{eng_score}} | {{eng_summary}} |
| 🔒 安全 | {{sec_score}} | {{sec_summary}} |
| ⚡ 性能 | {{perf_score}} | {{perf_summary}} |
| 📖 可读性 | {{read_score}} | {{read_summary}} |

**综合评分：{{overall_score}}** （满分 S）

---

## 💎 亮点

{{#each highlights}}
### {{title}}
- 📁 `{{file}}:{{line}}`
- {{description}}
{{/each}}

---

## 🔴 硬伤（必须改）

{{#each critical_issues}}
### {{title}} — {{severity}}
- 📁 `{{file}}:{{line}}`
- **问题**：{{description}}
- **风险**：{{risk}}
- **建议**：{{suggestion}}
{{/each}}

---

## 🟡 值得关注

{{medium_issues}}

---

## 🟢 建议优化

{{low_issues}}

---

## 🏥 Top 3 处方

1. **{{rx1_title}}** — {{rx1_reason}} — {{rx1_solution}}
2. **{{rx2_title}}** — {{rx2_reason}} — {{rx2_solution}}
3. **{{rx3_title}}** — {{rx3_reason}} — {{rx3_solution}}

---

## 📐 同类参考

{{similar_projects_reference}}

---

_由 Repo-Roast Skill 生成 | Flavor: {{flavor}} | 审查时间: {{date}}_
