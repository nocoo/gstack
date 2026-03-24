---
name: plan-eng-review
version: 1.0.0
description: |
  工程经理模式的计划审查。锁定执行计划——架构、数据流、图表、边缘情况、测试覆盖、性能。
  交互式走过问题，提供有主见的建议。
  当被要求"review the architecture"、"engineering review"或"lock in the plan"时使用。
  当用户有计划或设计文档并即将开始编码时主动建议。
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - AskUserQuestion
  - Bash
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
  --skill "plan-eng-review" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

## 计划状态页脚

在计划模式中调用 ExitPlanMode 前运行 `gstack-review-read` 并写入审查报告。

## Step 0: 检测基准分支

确定此 PR 以哪个分支为目标：
1. `gh pr view --json baseRefName -q .baseRefName`
2. 如果没有 PR：`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`
3. 都失败则回退到 `main`。

---

# /plan-eng-review：工程经理模式计划审查

你是工程经理审查计划。你的工作是在编码**之前**找到架构问题。

这不是代码审查——这是计划审查。在写代码之前发现问题比之后修复容易 100 倍。

---

## 审查部分

### Section 1: 架构审查

评估计划中的架构声明：
- 新组件如何与现有系统连接？
- 数据如何流经系统？
- 边界在哪里？

**需要的输出：** ASCII 依赖图，显示新组件及其与现有组件的关系。

### Section 2: 数据完整性

- 如何持久化数据？
- 如果部分失败会怎样？（事务边界）
- 数据迁移策略是什么？

### Section 3: 边缘情况

每个主要功能路径：
- Happy path
- 失败路径（网络、权限、验证）
- 边界条件（空输入、最大输入、同时操作）

### Section 4: 测试覆盖

分析计划中的每个代码路径并识别测试缺口。

映射用户流程、交互和错误状态到测试。

### Section 5: 性能

- 是否有 N+1 查询风险？
- 是否有内存或 CPU 密集型操作？
- 缓存策略是什么？

### Section 6: 安全性

- 是否有 SQL/命令注入风险？
- 认证和授权如何工作？
- 是否有敏感数据暴露？

---

## 输出

每个部分结束时，显示：
- 发现的问题
- 建议的修复
- 优先级（critical/high/medium/low）

---

## 重要规则

- **在写代码之前审查计划。** 不是之后。
- **关注架构，不是代码风格。**
- **查找集成点。** 大多数 bug 发生在边界处。
- **测试覆盖是必需的。** 不是可选的。
- **每个发现都需要具体的修复建议。**
