# 🔥 Repo-Roast —— 锐评你的代码仓库

<p align="center">
  <i>一个有观点、有审美、有脾气的 Claude Code Skill。<br>不是 lint，不是静态分析——是写了 20 年代码的 Tech Lead 在说真话。</i>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-black?style=flat-square&logo=anthropic&logoColor=white" alt="Claude Code">
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="MIT License">
  <img src="https://img.shields.io/badge/Version-0.3.0-blue?style=flat-square" alt="v0.3.0">
  <img src="https://img.shields.io/badge/Made_with-🔥_Attitude-red?style=flat-square" alt="Attitude">
</p>

---

## 为什么需要 Repo-Roast？

现有工具各管一摊，没人看全局：

| 工具 | 做什么 | 缺什么 |
|------|--------|--------|
| SonarQube | 静态规则扫描 | 没有语义理解，不会说人话 |
| CodeScene | 热点分析 | 不读代码内容 |
| ESLint/Prettier | 格式 + 简单规则 | 只管对不对，不管好不好 |
| **Repo-Roast** | 五维度语义级审查 | — |

## 能做什么

输入一个 GitHub 仓库地址或本地路径，三阶段自动完成：

```
用户输入仓库地址
       │
       ▼
┌──────────────────┐
│  Phase 1: Scout   │  ← 单 Agent，≤10 次调用建立项目画像
└────────┬─────────┘
         │ 输出：项目画像 JSON
         ▼
┌──────────────────────────────────────────┐
│  Phase 2: Deep Dive  ← 5 Sub-Agent 并行  │
│                                           │
│  🏗️ 架构  │  🔒 安全  │  ⚡ 性能          │
│  📖 可读性 │  📂 工程化                    │
└────────┬─────────────────────────────────┘
         │ 各维度评分 + 问题清单
         ▼
┌──────────────────┐
│  Phase 3: Editor  │  ← 单 Agent，汇总写报告
└────────┬─────────┘
         │
         ▼
  锐评报告（REPO_ROAST_REPORT.md）
```

审查结束后，在你的仓库根目录生成一份带评分、带证据、带建议的结构化锐评报告。

## 五条铁律

不是建议，是底线。违反任何一条视为审查失败：

| # | 铁律 | 含义 |
|---|------|------|
| 1 | **有据可依** | 每个批评必须带文件名 + 行号。没有证据的批评是诽谤 |
| 2 | **有褒有贬** | 只喷不夸是情绪发泄。亮点必须认，烂处必须说 |
| 3 | **拒绝模糊** | 禁止"可能存在""或许可以"。确认有问题就说，否则闭嘴 |
| 4 | **对事不对人** | 评代码，不评作者 |
| 5 | **给出路** | 指出问题必须附带修改建议，哪怕是方向性的 |

## 使用

```bash
# 基础用法
review https://github.com/xxx/yyy

# 指定审查视角
review https://github.com/xxx/yyy --flavor google              # Google 工程文化：重视可读性和测试
review https://github.com/xxx/yyy --flavor startup              # 硅谷创业公司：务实速度导向
review https://github.com/xxx/yyy --flavor oss-maintainer       # 开源维护者：关注文档和社区

# 指定审查维度
review https://github.com/xxx/yyy --dimensions security,performance

# 聚焦特定目录
review /path/to/local/project --focus src/core/

# 审查深度控制
review https://github.com/xxx/yyy --depth deep    # <5k 行全量审查
review https://github.com/xxx/yyy --depth quick   # >50k 行只查热点
```

## 审查维度

| 维度 | 审查内容 |
|------|----------|
| 🏗️ **架构** | 模块依赖、分层清晰度、循环依赖、扩展性 |
| 🔒 **安全** | 硬编码密钥、注入点、输入校验、依赖漏洞 |
| ⚡ **性能** | N+1 查询、算法复杂度、资源泄漏、缓存策略 |
| 📖 **可读性** | 命名质量、函数长度、注释密度、风格一致性 |
| 📂 **工程化** | 测试覆盖、错误处理、文档、配置管理、Git 工作流 |

## 评分体系

| 等级 | 含义 | 标准 |
|------|------|------|
| **S** | 标杆级 | 该维度几乎完美，可作为同类项目参考实现 |
| **A** | 优秀 | 主流最佳实践遵循度 >90%，偶有小瑕疵 |
| **B** | 良好 | 1-2 个核心问题，有明显改进空间 |
| **C** | 及格 | 3+ 个核心问题，但不影响基本功能 |
| **D** | 不及格 | 存在严重影响可维护性/安全性的问题，需要重构 |

综合评分由各维度评分按矩阵聚合得出（详见 spec）。

## 审查视角（Flavor）

同一个仓库，不同视角看到的问题不同：

| Flavor | 视角 | 侧重 |
|--------|------|------|
| `default` | 🧓 老炮工程师 | 直率、有观点。过度设计？简历驱动开发？依赖膨胀？ |
| `google` | 🏢 Google 工程文化 | 可读性优先、80%+ 覆盖率、style guide 合规 |
| `startup` | 🚀 硅谷创业公司 | MVP 可交付性、技术债可不可控、上手成本 |
| `oss-maintainer` | 🌍 开源维护者 | CONTRIBUTING 指南、社区友好度、onboarding 体验 |

## 安装

```bash
git clone https://github.com/haodehaode378/ruiping-skill.git ~/.claude/skills/repo-roast
```

重启 Claude Code 后，说 "review https://github.com/xxx/yyy" 即触发。

## 大仓库自适应

| 代码行数 | 深度 | 审查范围 |
|----------|------|----------|
| < 5,000 | deep | 全部源文件 |
| 5,000 – 50,000 | standard | 核心模块 |
| > 50,000 | quick | 热点文件 top 20 + 入口 + 配置 |

## 目录结构

```
repo-roast/
├── SKILL.md                    # 入口：铁律 + 三阶段流程编排
├── prompts/
│   ├── scout.md                # Phase 1 项目侦察
│   ├── architect.md            # 架构审查 Agent
│   ├── security.md             # 安全审查 Agent
│   ├── performance.md          # 性能审查 Agent
│   ├── readability.md          # 可读性审查 Agent
│   ├── engineering.md          # 工程化审查 Agent
│   └── editor.md               # Phase 3 汇总主笔
├── rubrics/                    # S/A/B/C/D 评分标准
│   ├── architecture.md
│   ├── security.md
│   ├── performance.md
│   ├── readability.md
│   └── engineering.md
├── flavors/                    # 审查视角
│   ├── default.md
│   ├── google.md
│   ├── startup.md
│   └── oss-maintainer.md
└── templates/
    └── report.md               # 报告 Markdown 模板
```

## License

MIT © 锐评
