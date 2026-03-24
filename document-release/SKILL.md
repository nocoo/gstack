---
name: document-release
version: 1.0.0
description: |
  发布后文档更新。读取所有项目文档，与 diff 交叉引用，更新 README/ARCHITECTURE/CONTRIBUTING/CLAUDE.md 
  以匹配已发布的内容，润色 CHANGELOG 语言，清理 TODOS，可选地升级 VERSION。
  当被要求"update the docs"、"sync documentation"或"post-ship docs"时使用。
  PR 合并或代码发布后主动建议。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion

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
  --skill "document-release" --duration "$_TEL_DUR" --outcome "OUTCOME" \
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

# 文档发布：发布后文档更新

你正在运行 `/document-release` 工作流。这在 `/ship` **之后**运行（代码已提交，PR 已存在或即将存在），
但在 PR 合并**之前**。你的工作：确保项目中的每个文档文件都是准确的、最新的，并用友好、面向用户的语言编写。

你主要是自动化的。直接进行明显的实事更新。只针对有风险或主观的决定停止并询问。

**只在这些情况下停止：**
- 有风险/有问题的文档更改（叙事性、哲学性、安全性、删除、大型重写）
- VERSION 升级决定（如果尚未升级）
- 要添加的新 TODOS 项目
- 叙事性的跨文档矛盾（非事实性）

**永不停止于：**
- 明显来自 diff 的事实更正
- 向表格/列表添加项目
- 更新路径、计数、版本号
- 修复过时的交叉引用
- CHANGELOG 语言润色（小措辞调整）
- 标记 TODOS 完成
- 跨文档事实不一致（如版本号不匹配）

**永不做：**
- 覆盖、替换或重新生成 CHANGELOG 条目——只润色措辞，保留所有内容
- 不询问就升级 VERSION——始终使用 AskUserQuestion 进行版本更改
- 在 CHANGELOG.md 上使用 `Write` 工具——始终使用带精确 `old_string` 匹配的 `Edit`

---

## Step 1: 预检与 Diff 分析

1. 检查当前分支。如果在基准分支上，**中止**："You're on the base branch. Run from a feature branch."

2. 收集关于变更的上下文：
```bash
git diff <base>...HEAD --stat
git log <base>..HEAD --oneline
git diff <base>...HEAD --name-only
```

3. 发现仓库中的所有文档文件。

4. 将变更分类为与文档相关的类别：
   - **新功能** — 新文件、新命令、新 skill、新能力
   - **行为变更** — 修改的服务、更新的 API、配置更改
   - **移除的功能** — 删除的文件、移除的命令
   - **基础设施** — 构建系统、测试基础设施、CI

5. 输出简要摘要。

---

## Step 2: 每文件文档审计

阅读每个文档文件并与 diff 交叉引用。

**README.md:**
- 它是否描述了 diff 中可见的所有功能和能力？
- 安装/设置说明是否与变更一致？
- 示例、演示和用法描述是否仍然有效？
- 故障排除步骤是否仍然准确？

**ARCHITECTURE.md:**
- ASCII 图表和组件描述是否与当前代码匹配？
- 设计决策和"为什么"解释是否仍然准确？
- 保守——只更新 diff 明显矛盾的内容。

**CONTRIBUTING.md — 新贡献者冒烟测试：**
- 像全新贡献者一样走一遍设置说明。
- 列出的命令是否准确？每一步会成功吗？
- 测试层描述是否与当前测试基础设施匹配？

**CLAUDE.md / 项目说明：**
- 项目结构部分是否与实际文件树匹配？
- 列出的命令和脚本是否准确？
- 构建/测试说明是否与 package.json（或等效）中的匹配？

**任何其他 .md 文件：**
- 阅读文件，确定其目的和受众。
- 与 diff 交叉引用以检查是否与文件所说的任何内容矛盾。

对于每个文件，将所需更新分类为：
- **自动更新** — diff 明确支持的事实更正
- **询问用户** — 叙事性更改、章节删除、安全模型更改、大型重写

---

## Step 3: 应用自动更新

使用 Edit 工具直接进行所有清晰的事实更新。

对于每个修改的文件，输出一行摘要描述**具体更改了什么**。

**永不自动更新：**
- README 介绍或项目定位
- ARCHITECTURE 哲学或设计原理
- 安全模型描述
- 不要从任何文档中删除整个章节

---

## Step 4: 询问有风险/有问题的更改

对于 Step 2 中识别的每个有风险或有问题的更新，使用 AskUserQuestion。

---

## Step 5: CHANGELOG 语言润色

**关键——永不覆盖 CHANGELOG 条目。**

此步骤润色语言。它不重写、替换或重新生成 CHANGELOG 内容。

**规则：**
1. 先阅读整个 CHANGELOG.md。了解已有什么。
2. 只修改现有条目中的措辞。永不删除、重新排序或替换条目。
3. 永不从零重新生成 CHANGELOG 条目。
4. 如果条目看起来错误或不完整，使用 AskUserQuestion——不要静默修复。
5. 使用带精确 `old_string` 匹配的 Edit 工具——永不使用 Write 覆盖 CHANGELOG.md。

**如果 CHANGELOG 在此分支中未被修改：** 跳过此步骤。

**如果 CHANGELOG 在此分支中被修改**，审查条目的语言：
- **销售测试：** 用户阅读每个 bullet 会认为"哦不错，我想试试那个"吗？如果不是，重写措辞（不是内容）。
- 以用户现在可以**做什么**开头——不是实现细节。
- "你现在可以..."不是"重构了..."
- 标记并重写任何读起来像提交消息的条目。
- 内部/贡献者更改属于单独的"### 贡献者"子章节。

---

## Step 6: 跨文档一致性与可发现性检查

在单独审计每个文件后，进行跨文档一致性检查：

1. README 的功能/能力列表是否与 CLAUDE.md 描述的匹配？
2. ARCHITECTURE 的组件列表是否与 CONTRIBUTING 的项目结构描述匹配？
3. CHANGELOG 的最新版本是否与 VERSION 文件匹配？
4. **可发现性：** 每个文档文件是否可以从 README.md 或 CLAUDE.md 到达？
5. 标记文档之间的任何矛盾。

---

## Step 7: TODOS.md 清理

这是补充 `/ship` Step 5.5 的第二遍。

如果 TODOS.md 不存在，跳过此步骤。

1. **尚未标记的已完成项目：** 与 diff 交叉引用开放的 TODO 项目。
2. **需要描述更新的项目：** 如果 TODO 引用的文件或组件被显著更改。
3. **新的延期工作：** 检查 diff 中的 TODO、FIXME、HACK 和 XXX 注释。

---

## Step 8: VERSION 升级问题

**关键——永不静默升级 VERSION。**

1. **如果 VERSION 不存在：** 静默跳过。

2. 检查 VERSION 是否已在此分支上被修改。

3. **如果 VERSION 未被升级：** 使用 AskUserQuestion：
   - RECOMMENDATION: 选择 C（跳过），因为仅文档更改很少需要版本升级
   - A) 升级 PATCH (X.Y.Z+1)
   - B) 升级 MINOR (X.Y+1.0)
   - C) 跳过——不需要版本升级

4. **如果 VERSION 已被升级：** 检查升级是否仍覆盖此分支上的全部变更范围。

---

## Step 9: 提交与输出

**先空检查：** 运行 `git status`。如果没有文档文件被任何先前步骤修改，输出"All documentation is up to date."并不提交地退出。

**提交：**

1. 按名称暂存修改的文档文件（永不 `git add -A` 或 `git add .`）。
2. 创建单个提交。
3. 推送到当前分支。

**PR 正文更新（幂等，竞态安全）：**

1. 将现有 PR 正文读入 PID 唯一临时文件。
2. 如果临时文件已包含 `## Documentation` 章节，用更新内容替换该章节。
3. 文档章节应包含**文档 diff 预览**。
4. 写回更新的正文。
5. 清理临时文件。

**结构化文档健康摘要（最终输出）：**

输出可扫描的摘要显示每个文档文件的状态。

---

## 重要规则

- **编辑前先阅读。** 在修改之前始终阅读文件的完整内容。
- **永不覆盖 CHANGELOG。** 只润色措辞。永不删除、替换或重新生成条目。
- **永不静默升级 VERSION。** 始终询问。
- **明确更改了什么。** 每次编辑都有一行摘要。
- **通用启发式，不是项目特定的。** 审计检查适用于任何仓库。
- **可发现性很重要。** 每个文档文件都应该可以从 README 或 CLAUDE.md 到达。
- **语言：友好、面向用户、不晦涩。** 像向没看过代码的聪明人解释一样写。
