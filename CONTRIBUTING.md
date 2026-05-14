# Contributing to Repo-Roast

感谢你对 Repo-Roast 的关注！

## 开发环境

本项目是 Claude Code Skill，无代码依赖。只需要：

1. Claude Code CLI
2. Git

## 项目结构

```
repo-roast/
├── SKILL.md           # 入口文件：铁律 + 三阶段编排
├── prompts/           # 各阶段 Agent 指令
│   ├── scout.md       # Phase 1 项目侦察
│   ├── architect.md   # 架构审查
│   ├── security.md    # 安全审查
│   ├── performance.md # 性能审查
│   ├── readability.md # 可读性审查
│   ├── engineering.md # 工程化审查
│   └── editor.md      # Phase 3 汇总
├── rubrics/           # S/A/B/C/D 评分标准
├── flavors/           # 审查视角（语气/权重/关注点）
└── templates/         # 报告 Markdown 模板
```

## 如何贡献

### 添加新 Flavor

1. 在 `repo-roast/flavors/` 下创建 `your-flavor.md`
2. 参考现有 flavor 的结构：权重、语气、特别关注、报告调整、禁止措辞
3. 权重必须引用 5 个维度：architecture, security, performance, readability, engineering
4. 在 `SKILL.md` 的输入解析表中添加新 flavor
5. 在 `README.md` 的 Flavor 表中添加说明

### 修改评分标准

1. 编辑 `repo-roast/rubrics/` 下对应文件
2. 保持 S/A/B/C/D 五级结构
3. 每级需有具体可量化的标准

### 修改 Prompt

1. 编辑 `repo-roast/prompts/` 下对应文件
2. 保持 Role / Review Scope / Scoring / Output / Constraints 结构
3. 输出 JSON 格式不得随意变更（已有下游依赖）

### 修改报告模板

1. 编辑 `repo-roast/templates/report.md`
2. 使用 `{{placeholder}}` 风格的变量
3. 确保 Editor prompt 中引用的变量名一致

## Commit 规范

使用中文 conventional commits：

```
feat: 添加新功能描述
fix: 修复具体问题
docs: 文档更新
chore: 杂项维护
refactor: 重构（不改变功能）
```

## 测试

手动测试：在 Claude Code 中使用 `/repo-roast` 审查一个真实仓库，确认：

- [ ] Scout 输出有效 JSON
- [ ] 5 个 Sub-Agent 正常启动并返回结果
- [ ] 报告生成完整且格式正确
- [ ] Flavor 切换后语气体感不同
- [ ] `--dimensions` 参数能正确过滤维度
