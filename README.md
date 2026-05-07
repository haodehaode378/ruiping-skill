# 🔥 Repo-Roast —— 锐评你的代码仓库

一个有观点、有审美、有脾气的 Claude Code Skill。不是 lint，不是静态分析——这是一个写了 20 年代码、review 过 10000+ PR 的 Tech Lead 在说真话。

## 能做什么

输入一个 GitHub 仓库地址或本地路径，自动完成三阶段审查：

1. **Scout** — 快速建立项目画像（语言、规模、依赖、CI、热点文件）
2. **Deep Dive** — 5 个 Sub-Agent 并行审查：架构、安全、性能、可读性、工程化
3. **Editor** — 汇总生成一份带评分、带证据、带建议的锐评报告

输出示例：`REPO_ROAST_REPORT.md`

## 安装

```bash
# 克隆到 Claude Code skills 目录
git clone https://github.com/haodehaode378/ruiping-skill.git ~/.claude/skills/repo-roast
```

## 使用

```bash
# 基础用法
review https://github.com/xxx/yyy

# 指定审查视角
review https://github.com/xxx/yyy --flavor google      # Google 工程文化视角
review https://github.com/xxx/yyy --flavor startup      # 硅谷创业公司视角
review https://github.com/xxx/yyy --flavor oss-maintainer # 开源维护者视角

# 指定审查维度
review https://github.com/xxx/yyy --dimensions security,performance

# 聚焦特定目录
review /path/to/local/project --focus src/core/
```

## 评分体系

| 等级 | 含义 |
|------|------|
| S | 几乎完美，可作为参考实现 |
| A | 优秀，偶有小瑕疵 |
| B | 良好，有明显改进空间 |
| C | 及格，问题较多 |
| D | 不及格，需要重构 |

## 五条铁律

1. **有据可依** — 每个批评必须带文件名 + 行号
2. **有褒有贬** — 亮点必须认，烂处必须说
3. **拒绝模糊** — 禁止"可能存在""或许可以"
4. **对事不对人** — 评代码，不评作者
5. **给出路** — 指出问题必须附带修改建议

## 目录结构

```
repo-roast/
├── SKILL.md                  # 入口文件
├── prompts/                  # 各阶段 prompt
│   ├── scout.md              # Phase 1 项目侦察
│   ├── architect.md          # 架构审查
│   ├── security.md           # 安全审查
│   ├── performance.md        # 性能审查
│   ├── readability.md        # 可读性审查
│   ├── engineering.md        # 工程化审查
│   └── editor.md             # Phase 3 汇总
├── rubrics/                  # 评分标准
│   ├── architecture.md
│   ├── security.md
│   ├── performance.md
│   ├── readability.md
│   └── engineering.md
├── flavors/                  # 审查视角
│   ├── default.md
│   ├── google.md
│   ├── startup.md
│   └── oss-maintainer.md
└── templates/
    └── report.md             # 报告模板
```

## License

MIT
