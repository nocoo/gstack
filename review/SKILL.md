---
name: review
version: 1.0.0
description: |
  落地前 PR 审查。分析相对于基准分支的 diff，检查 SQL 安全性、LLM 信任边界违规、
  条件副作用等问题。
  当被要求"review this PR"、"code review"、"pre-landing review"时使用。
  当用户即将合并或落地代码变更时主动建议。
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
  - Grep
  - Glob
  - Agent
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
  --skill "review" --duration "$_TEL_DUR" --outcome "OUTCOME" \
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

# Pre-Landing PR 审查

运行 `/review` 工作流。分析当前分支相对于基准分支的 diff，检查测试未捕获的结构性问题。

---

## Step 1: 检查分支

1. 运行 `git branch --show-current` 获取当前分支。
2. 如果在基准分支上，输出"Nothing to review"并停止。
3. 运行 `git fetch origin <base> --quiet && git diff origin/<base> --stat` 检查是否有 diff。

---

## Step 1.5: 范围漂移检测

在审查代码质量之前检查：**他们构建的是请求的内容吗——不多不少？**

1. 读取 `TODOS.md`（如果存在）、PR 描述和提交消息。
2. 识别**声明的意图**——这个分支应该完成什么？
3. 运行 `git diff origin/<base> --stat` 并将更改的文件与声明的意图比较。
4. 评估范围：
   - **范围蔓延检测：** 更改的文件与声明意图无关
   - **需求缺失检测：** TODOS.md/PR 描述中的需求未在 diff 中解决

---

## Step 2: 读取检查清单

读取 `.claude/skills/review/checklist.md`。如果无法读取，停止并报告错误。

---

## Step 2.5: 检查 Greptile 审查评论

读取 `.claude/skills/review/greptile-triage.md` 并遵循获取、过滤、分类步骤。

---

## Step 3: 获取 diff

```bash
git fetch origin <base> --quiet
```

运行 `git diff origin/<base>` 获取完整 diff。

---

## Step 4: 两遍审查

1. **第一遍（CRITICAL）：** SQL 和数据安全、竞态条件和并发、LLM 输出信任边界、枚举和值完整性
2. **第二遍（信息性）：** 条件副作用、魔法数字和字符串耦合、死代码和一致性、LLM Prompt 问题、测试缺口、视图/前端、性能和Bundle影响

**枚举和值完整性需要读取 diff 之外的代码。**

---

## Step 4.5: 设计审查（条件性）

如果 diff 触及前端文件：
1. 检查 `DESIGN.md` 是否存在
2. 读取 `.claude/skills/review/design-checklist.md`
3. 读取每个更改的前端文件（完整文件，不只是 diff 块）
4. 应用设计检查清单

---

## Step 4.75: 测试覆盖图

100% 覆盖是目标。评估 diff 中更改的每个代码路径并识别测试缺口。

### 测试框架检测

检测项目测试框架。

### 追踪每个代码路径

读取每个更改的文件。追踪数据流：
1. 输入从哪里来？
2. 经过什么转换？
3. 最终去哪里？
4. 每一步可能出什么错？

### 映射用户流程、交互和错误状态

### 检查每个分支的现有测试

质量评分标准：
- ★★★ 测试行为包含边缘情况和错误路径
- ★★ 测试正确行为，仅愉快路径
- ★ 冒烟测试/存在性检查

### E2E 测试决策矩阵

- **推荐 E2E：** 跨 3+ 组件/服务的常见用户流程
- **推荐 EVAL：** 需要质量评估的关键 LLM 调用
- **保持单元测试：** 纯函数

### 回归规则

当覆盖审计识别到**回归**时，立即编写回归测试。

---

## Step 5: Fix-First 审查

每个发现都有操作——不只是 critical 的。

### Step 5a: 分类每个发现

每个发现分类为 AUTO-FIX 或 ASK。

### Step 5b: 自动修复所有 AUTO-FIX 项目

直接应用每个修复。

### Step 5c: 批量询问 ASK 项目

如果存在剩余的 ASK 项目，在一个 AskUserQuestion 中呈现。

### Step 5d: 应用用户批准的修复

---

## Step 5.5: TODOS 交叉引用

读取 `TODOS.md`。交叉引用 PR 与开放 TODOs。

---

## Step 5.6: 文档陈旧检查

交叉引用 diff 与文档文件。

---

## Step 5.7: 对抗性审查（自动缩放）

对抗性审查深度根据 diff 大小自动缩放。

- **小（< 50 行）：** 跳过对抗性审查
- **中（50-199 行）：** 运行 Codex 对抗性挑战（或 Claude 对抗性子代理）
- **大（200+ 行）：** 运行所有剩余 passes

---

## Step 5.8: 持久化 Eng Review 结果

```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"review","timestamp":"TIMESTAMP","status":"STATUS","issues_found":N,"critical":N,"informational":N,"commit":"COMMIT"}'
```

---

## 重要规则

- **在评论之前阅读完整 diff。** 不要标记 diff 中已解决的问题。
- **Fix-first，不是只读。** AUTO-FIX 项目直接应用。ASK 项目仅在用户批准后应用。
- **简洁。** 一行问题，一行修复。
- **只标记真实问题。** 跳过任何正常的。
- **使用 Greptile 回复模板。** 每个回复都包含证据。
