# 🔥 Repo-Roast —— 有证据的代码仓库锐评

<p align="center">
  <i>先完成五维技术审查，再选择怎么把事实说出来。<br>不是 lint，也不是情绪输出：正式问题必须落到真实文件和行号。</i>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-black?style=flat-square&logo=anthropic&logoColor=white" alt="Claude Code">
  <img src="https://img.shields.io/badge/Codex-black?style=flat-square&logo=openai&logoColor=white" alt="Codex">
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="MIT License">
  <img src="https://img.shields.io/badge/Version-0.4.0-blue?style=flat-square" alt="v0.4.0">
</p>

Repo-Roast 是一个证据驱动的仓库审查 Skill。它让 Scout 确定唯一审查范围，让五个 Deep Dive Agent 只输出中性技术事实，最后由 Editor 在不改动事实、严重度和修复建议的前提下，加入可控的锐评表达。

## v0.4.0 有什么不同

- 固定审查 `architecture`、`security`、`performance`、`readability`、`engineering` 五个维度。
- 每个正式 issue 都必须有仓库内真实 `file` 和有效 `line`；验证失败即从正式问题中删除。
- `Flavor` 决定审查立场、权重和优先级，`Tone` 只决定表达锐度，两者不再混用。
- `professional`、`sharp`、`savage` 三档 Tone 共享同一组技术 finding。
- 90 条结构化修辞语料按维度、问题模式、严重度和 Tone 匹配；匹配不到就保留纯技术描述。
- 明确区分 requested、completed、failed、skipped 四种维度状态，未审查不再伪装成低分。
- 所有阶段数据通过 JSON Schema 契约衔接，并提供六组回归 fixture。

## 工作流

```text
Scout：建立项目画像与唯一 review_scope（≤10 次工具调用）
  ↓
Deep Dive：五维并行审查，只产出中性、可验证的 JSON finding
  ↓
Editor：验证证据与状态 → 计算评分 → 按 Flavor 排序 → 按 Tone 表达
  ↓
REPO_ROAST_REPORT.md
```

修辞层永远不能修改 finding 的 `severity`、`file`、`line`、`title`、`description`、`impact`、`suggestion` 或 `pattern`。锐评可以更锋利，事实不能跟着变形。

## 五条铁律

| # | 铁律 | 含义 |
|---|------|------|
| 1 | 有据可依 | 正式批评必须带真实文件和行号 |
| 2 | 有褒有贬 | 亮点如实承认，问题同样直说 |
| 3 | 拒绝模糊 | 只写已确认事实，不拿猜测凑问题 |
| 4 | 对事不对人 | 只评价代码、设计和工程结果，禁止人身攻击 |
| 5 | 给出路 | 每个 issue 都附带可执行的修改方向 |

## 使用

```text
# 默认 Tone 为 sharp
review https://github.com/xxx/yyy

# 三档表达锐度
review https://github.com/xxx/yyy --tone professional
review https://github.com/xxx/yyy --tone sharp
review https://github.com/xxx/yyy --tone savage

# 审查立场、维度、深度和范围
review /absolute/path/to/repo --flavor google
review /absolute/path/to/repo --dimensions security,performance
review /absolute/path/to/repo --depth deep --focus src/core/
```

支持的 Flavor：`default`、`google`、`startup`、`oss-maintainer`。支持的 Tone：`professional`、`sharp`、`savage`。

### 同一个 finding，不同 Tone

以下三句话基于同一条已验证 finding：`src/auth.ts:42` 把访问令牌写入源码，严重度始终为 `critical`，修复建议始终是移除令牌并轮换凭据。

| Tone | 表达示例 |
|------|----------|
| `professional` | `src/auth.ts:42` 存在硬编码访问令牌，源码泄露会直接暴露凭据；请立即移除并轮换。 |
| `sharp` | 访问令牌被写进源码，版本库在这里兼职保险箱，而且还是透明的；立即移除并轮换。 |
| `savage` | 把访问令牌焊进源码，不叫配置管理，叫给泄露事故预填工单；立即移除并轮换。 |

`savage` 允许更强的技术讽刺，但仍然只批评可观察的实现，不攻击作者，不使用歧视、羞辱、威胁或煽动性措辞。

## Flavor 与 Tone

| 概念 | 控制什么 | 不控制什么 |
|------|----------|------------|
| Flavor | 五维权重、审查立场、问题优先级 | 不凭空制造 finding，不取消证据要求 |
| Tone | 句式、修辞密度、表达锐度 | 不改变事实、评分、严重度和建议 |

| Flavor | 主要视角 |
|--------|----------|
| `default` | 维护成本、边界、长期工程质量 |
| `google` | 可读性、测试、清晰接口和一致性 |
| `startup` | 交付速度、风险敞口和可控技术债 |
| `oss-maintainer` | 文档、贡献体验、兼容性和社区维护 |

## 审查状态与评分

- `completed`：该维度结果通过 Schema 和证据验证，参与评分。
- `failed`：请求过但超时、解析失败或验证失败，报告明确原因。
- `skipped`：用户没有请求，不参与评分，也不显示成低分。
- `requested`：用户本次要求审查的维度全集。

S/A/B/C/D 只评价已完成维度；安全维度还可以记录没有足够证据升级为正式 finding 的观察项，避免“没发现”和“没检查”混为一谈。

## 报告片段

```markdown
# Repo-Roast 锐评报告

> Flavor: startup | Tone: sharp
> Requested: architecture, security
> Completed: architecture, security | Failed: — | Skipped: performance, readability, engineering

## 🔒 security — D

### [CRITICAL] 硬编码访问令牌
- 证据：`src/auth.ts:42`
- 事实：访问令牌以字符串字面量写入源码。
- 影响：获得仓库读取权限的人可以直接取得有效凭据。
- 锐评：版本库在这里兼职保险箱，而且还是透明的。
- 建议：删除源码中的令牌，从密钥管理服务读取，并立即轮换现有凭据。
```

## 安装

本仓库的可安装 Skill 是其中的 `repo-roast/` 目录。无论使用哪种客户端，安装后都应满足：

```text
<skills-root>/repo-roast/SKILL.md
```

不要把整个仓库直接克隆到 `<skills-root>/repo-roast`，否则入口会变成 `<skills-root>/repo-roast/repo-roast/SKILL.md`，客户端可能无法发现。

### Claude Code

Claude Code 官方支持项目级 `.claude/skills/<skill-name>/` 和用户级 `~/.claude/skills/<skill-name>/`。先克隆源码，再只复制内部 Skill 目录：

```bash
git clone --depth 1 https://github.com/haodehaode378/ruiping-skill.git ruiping-skill-source
mkdir -p .claude/skills
cp -R ruiping-skill-source/repo-roast .claude/skills/repo-roast
```

安装结果应为 `.claude/skills/repo-roast/SKILL.md`，之后可使用 `/repo-roast` 或提出匹配其描述的审查请求。若顶层 `.claude/skills` 是在当前会话启动后首次创建，请重启 Claude Code。

### Codex

Codex 官方支持仓库级 `$REPO_ROOT/.agents/skills` 和用户级 `$HOME/.agents/skills`。仓库级安装示例：

```bash
git clone --depth 1 https://github.com/haodehaode378/ruiping-skill.git ruiping-skill-source
mkdir -p .agents/skills
cp -R ruiping-skill-source/repo-roast .agents/skills/repo-roast
```

安装结果应为 `.agents/skills/repo-roast/SKILL.md`，之后可使用 `$repo-roast` 或提出匹配其描述的审查请求。Codex 通常会自动发现 Skill 变更；若没有出现，请重启客户端。

PowerShell 用户可将最后两行替换为：

```powershell
New-Item -ItemType Directory -Force .agents\skills | Out-Null
Copy-Item -Recurse .\ruiping-skill-source\repo-roast .\.agents\skills\repo-roast
```

当前版本已完成 Schema、语料和 fixture 的仓库内自动验收；发布流程没有冒充完成真实 Claude Code/Codex 会话的端到端运行验证。

## 目录结构

```text
repo-roast/
├── SKILL.md                 # 入口与三阶段编排
├── V0.4-DESIGN.md           # v0.4.0 设计与数据流
├── agents/openai.yaml       # Codex 展示与默认提示元数据
├── prompts/                 # Scout、五维 Deep Dive、Editor 指令
├── rubrics/                 # 五维 S/A/B/C/D 评分标准
├── schemas/                 # Scout、Review、Report Input、Rhetoric JSON Schema
├── flavors/                 # 审查立场、权重和优先级
├── rhetoric/
│   ├── principles.md        # 事实与修辞边界
│   ├── banned.md            # 禁止表达
│   ├── devices.md           # 可用修辞设备
│   ├── transitions.json     # 连接语料
│   ├── tones/               # professional / sharp / savage
│   └── patterns/            # 五维 90 条结构化锐评语料
├── fixtures/                # 六组回归输入、期望报告与验收矩阵
└── templates/report.md      # 报告模板
```

## 参考文档

- [Codex：Build skills](https://learn.chatgpt.com/docs/build-skills)
- [Claude Code：Extend Claude with skills](https://code.claude.com/docs/en/skills)

## License

MIT © 锐评
