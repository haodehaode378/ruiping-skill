# CLAUDE.md

## Project Overview

Repo-Roast 是一个兼容 Claude Code 与 Codex 的 Agent Skill，用于对 Git 仓库进行证据驱动的五维度语义审查（架构、安全、性能、可读性、工程化）。

## Tech Stack

- **Type**: Agent Skill（纯 Markdown/JSON，无代码依赖）
- **Runtime**: Claude Code / Codex Agent 框架
- **Output**: Markdown 报告

## Architecture

三阶段流水线：

```
Scout (单 Agent, ≤10 调用) → Deep Dive (5 并行 Sub-Agent) → Editor (单 Agent)
```

## Directory Layout

```
repo-roast/
├── SKILL.md           # 入口：铁律 + 三阶段流程编排
├── prompts/           # 各阶段 Agent 指令
├── rubrics/           # S/A/B/C/D 评分标准
├── schemas/           # 阶段间数据契约
├── flavors/           # 审查立场、权重、关注点
├── rhetoric/          # Tone、表达边界和结构化语料
├── fixtures/          # 回归验收材料
└── templates/         # 报告模板
```

## Configuration

- `plugin.json` — 根级插件元数据（名称、版本、标签）
- `.claude-plugin/plugin.json` — Claude Code 插件系统元数据
- `.claude-plugin/marketplace.json` — 市场发布元数据
- `repo-roast/agents/openai.yaml` — Codex 展示与默认提示元数据

## Commit Convention

使用中文 conventional commits：

```
<type>: 中文描述

feat: 添加新功能
fix: 修复问题
docs: 文档更新
chore: 杂项维护
refactor: 重构
```

## Key Constraints

- 五个审查维度固定：architecture, security, performance, readability, engineering
- 每个 flavor 必须引用上述 5 个维度名称
- 所有子 Agent 输出必须是严格 JSON
- 每个 issue 必须包含 `file` + `line` 字段
- Flavor 只控制立场和权重；Tone 只控制表达，不能改变 finding 或评分
