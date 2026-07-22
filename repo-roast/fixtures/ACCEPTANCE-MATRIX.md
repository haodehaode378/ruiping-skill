# v0.4.0 回归验收矩阵

## 重放规则

每个 `findings/*.json` fixture 都提供 Scout 输入、维度结果、Flavor、Tone、期望行为、禁止行为和不可变字段。重放时：

1. 使用 `schemas/scout.schema.json` 验证 `scout_input`。
2. 使用 `schemas/review.schema.json` 验证每个 `dimension_reviews` 值。
3. 将 fixture case 的状态、Flavor、Tone 与 Scout、Reviews 组装为 Editor 输入，并使用 `schemas/report-input.schema.json` 验证。
4. 使用 `repository_files` 模拟证据边界：键是存在的相对路径，值是最大有效行号。
5. 运行证据过滤、聚合和修辞匹配。
6. 比较 `expected_behavior`、`forbidden_behavior` 和 `immutable_fields`。

`immutable_fields` 固定为：`severity`、`file`、`line`、`title`、`description`、`impact`、`suggestion`、`pattern`。Tone 变化只能改变生成的 `roast`。

## 场景覆盖

| # | 场景 | Fixture / Case | 必须证明 |
|---:|---|---|---|
| 1 | 有证据的 N+1 | `aggregation.json / verified-n-plus-one` | 保留 finding 并匹配 N+1 语料 |
| 2 | 无证据的疑似漏洞 | `evidence-validation.json / unverified-vulnerability` | observation 不进入问题、评分或锐评 |
| 3 | 零测试 | `absence-and-architecture.json / zero-tests` | 只进入 stats、summary 或处方，不伪造行号 |
| 4 | 没有 CI | `absence-and-architecture.json / missing-ci` | 只进入 stats、summary 或处方，不伪造配置文件 |
| 5 | 小项目过度抽象 | `absence-and-architecture.json / small-over-engineering` | 使用已验证位置和 over-engineering 模式 |
| 6 | 跨维度重复 finding | `aggregation.json / cross-dimension-duplicate` | 同一位置和事实只报告一次 |
| 7 | Professional | `tone-invariance.json / professional` | 可以无幽默，事实字段不变 |
| 8 | Sharp | `tone-invariance.json / sharp` | 使用默认锐度，事实字段不变 |
| 9 | Savage | `tone-invariance.json / savage` | 只增强表达，不提高严重度 |
| 10 | 只请求两个维度 | `states-and-fallback.json / two-requested-dimensions` | 显示 2/2，其他维度为 skipped |
| 11 | 请求维度超时 | `states-and-fallback.json / requested-timeout` | 显示 1/2，超时维度为 failed |
| 12 | 没有匹配语料 | `states-and-fallback.json / unmatched-pattern` | 保留技术内容，`roast` 为空 |
| 13 | 重复 pattern | `aggregation.json / duplicate-pattern` | 不重复模板 id，不删除不同位置的问题 |
| 14 | Critical 安全问题 | `critical-security.json` | 优先条件、影响和修复，允许无锐评 |
| 15 | 文件不存在 | `evidence-validation.json / missing-file` | 删除正式 finding，不降级保留 |
| 16 | 行号越界 | `evidence-validation.json / line-out-of-range` | 删除正式 finding，不修改行号救场 |

## Tone 不变量

对 `tone-invariance.json` 的三次执行逐字段比较：

| 字段 | Professional | Sharp | Savage |
|---|---|---|---|
| severity | 相同 | 相同 | 相同 |
| file | 相同 | 相同 | 相同 |
| line | 相同 | 相同 | 相同 |
| title | 相同 | 相同 | 相同 |
| description | 相同 | 相同 | 相同 |
| impact | 相同 | 相同 | 相同 |
| suggestion | 相同 | 相同 | 相同 |
| pattern | 相同 | 相同 | 相同 |
| roast | 可选变化 | 可选变化 | 可选变化 |

## 报告验收

`expected-reports/two-dimension-sharp.md` 是完整预期报告，必须满足：

- 显示 `2/2` 而非 `2/5`。
- 三个未请求维度明确显示“未请求”。
- 锐评句后仍保留问题、影响和建议。
- 删除引用块中的锐评句后，报告仍完整说明 N+1 finding。
- 不包含未解析模板占位符。
- 不把缺失型事实包装成带伪造行号的问题。
