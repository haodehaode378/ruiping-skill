# 锐评修辞原则

## 最高原则

**技术事实优先。Roast the code, never the coder.**

锐评只能表达已经通过 `file`、`line` 和上下文验证的 finding。它不能参与发现事实、判断严重度或推断作者意图。

## 八条不变量

1. **先有事实，再有修辞**：没有有效 finding，就没有锐评句。
2. **事实字段只读**：修辞不得改变 `severity`、`file`、`line`、`title`、`description`、`impact`、`suggestion` 或 `pattern`。
3. **不添加新事实**：模板不能声称 finding 未证明的调用关系、数据规模、攻击路径、生产影响或团队动机。
4. **一句足够**：每个问题最多使用一句主要锐评，不用多个比喻包围同一个事实。
5. **Critical 先讲风险**：严重安全、数据完整性和可用性问题以准确说明条件和影响为先，不强行玩梗。
6. **允许没有笑话**：没有可靠匹配时只输出技术描述，不使用通用辱骂填空。
7. **避免修辞疲劳**：同一模板 id 在一份报告中只使用一次；不得连续两次使用相同修辞手法。
8. **删掉仍成立**：删除全部锐评句后，报告必须仍包含完整的事实、影响和修改建议。

## 适用顺序

Editor 只能在完成 JSON 验证、证据验证、去重和严重度排序后匹配修辞：

```text
dimension → pattern → severity → tone → 去除已使用模板 → 安全检查
```

匹配成功还必须满足：

- 模板的 `dimension` 与 finding 完全一致。
- finding 的 `pattern` 包含在模板的 `patterns` 中。
- finding 的 `severity` 包含在模板的 `severities` 中。
- 模板包含当前 Tone 的文本。
- `requires_verified_evidence` 为 `true`，且证据已验证。
- 文本没有命中 `banned.md` 的禁区。

任一条件不满足就回退到纯技术表达。

## 输出组合

正式问题按以下逻辑组织：

```text
可选锐评句 + description + impact + suggestion
```

- 锐评句负责帮助读者记住问题。
- `description` 负责陈述证据所支持的事实。
- `impact` 负责说明后果和成立条件。
- `suggestion` 负责给出行动方向。

锐评句不得替代后三项，也不得把建议写成惩罚、命令服从或人格判断。

## 五维共同边界

以下规则同时适用于 `architecture`、`security`、`performance`、`readability`、`engineering`：

- 只描述对应维度已经确认的事实。
- 不跨维度借题发挥。
- 不把工程偏好伪装成正确性问题。
- 不把仓库规模、团队背景或作者身份当作笑点。
- 不因 Tone 更锋利而提高 severity。
- 不因 Flavor 更宽容而隐藏已经成立的高严重度问题。
