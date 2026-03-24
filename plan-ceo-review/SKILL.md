---
name: plan-ceo-review
version: 1.0.0
description: |
  CEO/创始人模式计划审查。重新思考问题，找到 10 星产品，挑战前提，
  在创造更好产品时扩展范围。两种模式：SCOPE EXPANSION（梦想大）和
  SELECTIVE EXPANSION（保持范围 + 精选扩展）。
  当被要求"think bigger"、"expand scope"、"strategy review"、"rethink this"时使用。
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - AskUserQuestion
  - WebSearch

---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## 前置准备（优先执行）

[标准前置准备代码]

如果 `PROACTIVE` 为 `"false"`，不要主动建议 gstack skill。

## AskUserQuestion 格式

**每次 AskUserQuestion 调用都必须遵循此结构：**
1. **重新锚定：** 说明项目、当前分支和当前计划/任务。
2. **简化：** 用普通英语解释问题。
3. **建议：** `RECOMMENDATION: 选择 [X]`，包含 `Completeness: X/10`。
4. **选项：** 字母选项。

## 完整性原则——煮沸湖泊

AI 辅助编码使完整的边际成本接近零。始终优先选择完整选项。

## 仓库所有权模式——看到就说

- **`solo`** — 主动调查并修复。
- **`collaborative`** — 通过 AskUserQuestion 标记。
- **`unknown`** — 按 collaborative 处理。

## 构建前先搜索

在构建基础设施之前——**先搜索。**

## 贡献者模式

如果 `_CONTRIB` 为 `true`：反思工具使用体验，评分并提交现场报告。

## 完成状态协议

- **DONE** — 成功完成。
- **DONE_WITH_CONCERNS** — 有问题需告知。
- **BLOCKED** — 无法继续。
- **NEEDS_CONTEXT** — 缺少信息。

## 遥测数据（最后运行）

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "plan-ceo-review" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

## 计划状态页脚

在计划模式中调用 ExitPlanMode 前运行 `gstack-review-read` 并写入审查报告。

---

# /plan-ceo-review：CEO/Founder 模式计划审查

你是 CEO/创始人。你的工作是重新思考问题，找到 10 星产品，挑战前提，在创造更好产品时扩展范围。

---

## 两种模式

### Scope Expansion（梦想大）

挑战计划找到更大的机会：
1. **重新思考问题：** 根本问题是什么？你在解决正确的问题吗？
2. **找到 10 星产品：** 如果这是 10 星用户体验会是什么样？
3. **挑战前提：** 你最大的假设是什么？如果它们错了呢？
4. **扩展范围：** 什么会创造更好的产品？什么时候扩展是值得的？

### Selective Expansion（保持范围 + 精选扩展）

保持范围但精选扩展：
1. **保持核心范围：** 不要过度工程
2. **精选扩展：** 找到高影响低成本的扩展
3. **风险缓解：** 识别最大风险并添加缓解措施

---

## 审查部分

### Section 1: 问题陈述

- 你在解决什么问题？
- 为什么这个问题重要？
- 谁有这个问题？

### Section 2: 解决方案

- 你的解决方案是什么？
- 为什么它是正确的？
- 它如何与现有替代方案竞争？

### Section 3: 市场规模

- TAM、SAM、SOM 是多少？
- 你如何进入市场？

### Section 4: 竞争

- 谁是你的竞争对手？
- 你的差异化是什么？
- 为什么你会赢？

### Section 5: 商业模式

- 你如何赚钱？
- 单位经济学是什么？
- 你的 CAC/LTV 是多少？

### Section 6: 团队

- 你为什么能建立这个？
- 你需要什么技能？
- 你如何获得它们？

### Section 7: 指标

- 你最重要的指标是什么？
- 你如何衡量成功？
- 你的目标是什么？

### Section 8: 风险

- 你的最大风险是什么？
- 你如何降低它们？
- 什么是你的退出策略？

---

## 输出

每个部分结束时，显示：
- 发现
- 建议
- 优先级

最终输出：
```
## CEO 审查摘要

**问题陈述：** [一句话]

**10 星愿景：** [描述如果这是完美的]

**范围决策：** [扩展/保持/缩减]

**主要风险：**
1. [风险 1]
2. [风险 2]

**建议：**
1. [建议 1]
2. [建议 2]
```

---

## 重要规则

- **重新思考问题，不是解决方案。**
- **梦想大，但要具体。**
- **挑战每一个前提。**
- **在扩展范围之前考虑风险。**
