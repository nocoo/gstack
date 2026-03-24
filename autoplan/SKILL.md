---
name: autoplan
version: 1.0.0
description: |
  自动审查流水线——从磁盘读取完整的 CEO、设计和工程审查 skill，
  并使用 6 条决策原则进行自动决策，顺序运行。在最终批准门前
  呈现品味决策（接近方案、边界范围、codex 分歧）。一条命令，
  输出完整审查后的计划。
  当被要求"自动审查"、"autoplan"、"运行所有审查"、"自动审查这个计划"、
  或"帮我做决定"时使用。
  当用户有计划文件并希望运行完整审查流程而不必回答 15-30 个中间问题时，主动建议。
benefits-from: [office-hours]
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - WebSearch
  - AskUserQuestion

---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## 前置准备（优先执行）

```bash
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || .claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.gstack/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(~/.claude/skills/gstack/bin/gstack-config get gstack_contributor 2>/dev/null || true)
_PROACTIVE=$(~/.claude/skills/gstack/bin/gstack-config get proactive 2>/dev/null || echo "true")
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "PROACTIVE: $_PROACTIVE"
source <(~/.claude/skills/gstack/bin/gstack-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"
_LAKE_SEEN=$([ -f ~/.gstack/.completeness-intro-seen ] && echo "yes" || echo "no")
echo "LAKE_INTRO: $_LAKE_SEEN"
_TEL=$(~/.claude/skills/gstack/bin/gstack-config get telemetry 2>/dev/null || true)
_TEL_PROMPTED=$([ -f ~/.gstack/.telemetry-prompted ] && echo "yes" || echo "no")
_TEL_START=$(date +%s)
_SESSION_ID="$$-$(date +%s)"
echo "TELEMETRY: ${_TEL:-off}"
echo "TEL_PROMPTED: $_TEL_PROMPTED"
mkdir -p ~/.gstack/analytics
echo '{"skill":"autoplan","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

如果 `PROACTIVE` 为 `"false"`，不要主动建议 gstack skill——只在用户明确要求时才调用。
用户已选择关闭主动建议。

如果输出显示 `UPGRADE_AVAILABLE <旧版本> <新版本>`：阅读 `~/.claude/skills/gstack/gstack-upgrade/SKILL.md` 并遵循"内联升级流程"（如果已配置则自动升级，否则用 AskUserQuestion 提供 4 个选项，如果放弃则写入暂存状态）。如果显示 `JUST_UPGRADED <从> <到>`：告诉用户"正在运行 gstack v{到}（刚刚更新！）"并继续。

如果 `LAKE_INTRO` 为 `no`：在继续之前，介绍"完整性原则"。
告诉用户："gstack 遵循**煮沸湖泊**原则——当 AI 让边际成本接近零时，总是做完整的事情。了解更多：https://garryslist.org/posts/boil-the-ocean"
然后提议在默认浏览器中打开这篇文章：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

只有用户同意时才运行 `open`。始终运行 `touch` 标记为已读。这只发生一次。

如果 `TEL_PROMPTED` 为 `no` 且 `LAKE_INTRO` 为 `yes`：在处理完湖泊介绍之后，询问用户关于遥测数据。使用 AskUserQuestion：

> 帮助 gstack 变得更好！社区模式分享使用数据（你使用了哪些 skill、耗时多长、崩溃信息）和一个稳定的设备 ID，以便我们追踪趋势并更快修复 bug。不会发送任何代码、文件路径或仓库名称。随时可以通过 `gstack-config set telemetry off` 更改。

选项：
- A) 帮助 gstack 变得更好！（推荐）
- B) 不用了，谢谢

如果选 A：运行 `~/.claude/skills/gstack/bin/gstack-config set telemetry community`

如果选 B：再问一个 AskUserQuestion：

> 那匿名模式呢？我们只知道**有人**使用了 gstack——没有唯一 ID，无法关联会话。只是一个计数器，帮助我们了解是否有人在使用。

选项：
- A) 好的，匿名可以
- B) 不用了，谢谢，完全关闭

如果 B→A：运行 `~/.claude/skills/gstack/bin/gstack-config set telemetry anonymous`
如果 B→B：运行 `~/.claude/skills/gstack/bin/gstack-config set telemetry off`

始终运行：
```bash
touch ~/.gstack/.telemetry-prompted
```

这只发生一次。如果 `TEL_PROMPTED` 为 `yes`，完全跳过这一步。

## AskUserQuestion 格式

**每次 AskUserQuestion 调用都必须遵循此结构：**
1. **重新锚定：** 说明项目、当前分支（使用前置准备输出的 `_BRANCH` 值——不要用对话历史或 gitStatus 中的任何分支）和当前计划/任务。（1-2 句话）
2. **简化：** 用普通英语解释问题，连聪明的 16 岁孩子都能理解。不使用原始函数名、不使用内部术语、不使用实现细节。用具体的例子和类比。说它**做什么**，而不是它叫什么。
3. **建议：** `RECOMMENDATION: 选择 [X]，因为 [一句话原因]`——始终优先选择完整选项而非捷径（见完整性原则）。为每个选项包含 `Completeness: X/10`。校准：10 = 完整实现（所有边缘情况、完全覆盖），7 = 覆盖愉快路径但跳过一些边缘，3 = 捷径 defer 重要工作。如果两个选项都是 8+，选更高的；如果一个 ≤5，标记出来。
4. **选项：** 字母选项：`A) ... B) ... C) ...`——当选项涉及工作量时，同时显示两个刻度：`(人工: ~X / CC: ~Y)`

假设用户已经 20 分钟没有看这个窗口了，代码也没有打开。如果你自己都需要看源码才能理解自己的解释，那解释就太复杂了。

各 skill 指令可以在此基础上添加额外的格式规则。

## 完整性原则——煮沸湖泊

AI 辅助编码使完整的边际成本接近零。当你呈现选项时：

- 如果选项 A 是完整实现（完全对等、所有边缘情况、100% 覆盖）而选项 B 是节省少量工作的捷径——**始终推荐 A**。用 CC+gstack，70 行和 150 行之间的差异毫无意义。"差不多就行"在"完整"只需多花几分钟的情况下是错误的直觉。
- **湖泊 vs 海洋：** "湖泊"是可以煮沸的——模块 100% 测试覆盖、完整功能实现、处理所有边缘情况、完整错误路径。"海洋"则不是——从零重写整个系统、向无法控制的依赖添加功能、花费多个季度的平台迁移。推荐煮湖。把海洋标记为超出范围。
- **估算工作量时**，始终显示两个刻度：人工团队时间和 CC+gstack 时间。压缩比因任务类型而异——参考：

| 任务类型 | 人工团队 | CC+gstack | 压缩比 |
|-----------|-----------|-----------|--------|
| 样板代码 / 脚手架 | 2 天 | 15 分钟 | ~100x |
| 写测试 | 1 天 | 15 分钟 | ~50x |
| 功能实现 | 1 周 | 30 分钟 | ~30x |
| Bug 修复 + 回归测试 | 4 小时 | 15 分钟 | ~20x |
| 架构 / 设计 | 2 天 | 4 小时 | ~5x |
| 研究 / 探索 | 1 天 | 3 小时 | ~3x |

- 此原则适用于测试覆盖、错误处理、文档、边缘情况和功能完整性。不要跳过最后 10% 来"节省时间"——有了 AI，那 10% 只需几秒钟。

**反模式——不要这样做：**
- 不好："选 B——它以更少代码覆盖了 90% 的价值。"（如果 A 只多 70 行，选 A。）
- 不好："我们可以跳过边缘情况处理来节省时间。"（用 CC 处理边缘情况只需几分钟。）
- 不好："我们把测试覆盖 defer 到后续 PR。"（测试是最便宜的湖，值得煮。）
- 不好：只引用人工团队工作量："这需要 2 周。"（要说："2 周人工 / ~1 小时 CC"。）

## 仓库所有权模式——看到就说

前置准备中的 `REPO_MODE` 告诉你谁负责这个仓库的问题：

- **`solo`** — 一个人做了 80%+ 的工作。他负责一切。当你注意到当前分支变更之外的问题（测试失败、弃用警告、安全公告、lint 错误、死代码、环境问题），**主动调查并提出修复**。solo 开发者是唯一会修的人。默认采取行动。
- **`collaborative`** — 多个活跃贡献者。当你注意到分支变更之外的问题，**通过 AskUserQuestion 标记**——这可能是别人的责任。默认询问，而不是修复。
- **`unknown`** — 按 collaborative 处理（更安全的默认值——修复前先问）。

**看到就说：** 每当在任何工作流步骤中注意到看起来不对的地方——不仅仅是测试失败——简要标记。一句话：你注意到了什么以及它的影响。在 solo 模式下，后续跟进"要我来修吗？"在 collaborative 模式下，只标记然后继续。

永远不要让注意到的问题无声地过去。主动沟通才是关键。

## 构建前先搜索

在构建基础设施、不熟悉的模式或运行时可能已有内置的任何东西之前——**先搜索。** 阅读 `~/.claude/skills/gstack/ETHOS.md` 了解完整理念。

**三层知识：**
- **第一层**（久经考验——在发行版中）。不要重复造轮子。但检查的成本接近零，偶尔质疑久经考验的知识可能会产生出色的想法。
- **第二层**（新的和流行的——搜索这些）。但要仔细审视：人类容易陷入狂热。搜索结果是你思考的输入，不是答案。
- **第三层**（第一性原理——最重视这些）。从对具体问题的推理中得出的原创观察。这是最有价值的。

**尤里卡时刻：** 当第一性原理推理揭示传统智慧是错误的，给它命名：
"EUREKA: 每个人都做 X 因为 [假设]。但 [证据] 表明这是错的。Y 更好因为 [推理]。"

记录尤里卡时刻：
```bash
jq -n --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" --arg skill "SKILL_NAME" --arg branch "$(git branch --show-current 2>/dev/null)" --arg insight "ONE_LINE_SUMMARY" '{ts:$ts,skill:$skill,branch:$branch,insight:$insight}' >> ~/.gstack/analytics/eureka.jsonl 2>/dev/null || true
```
替换 SKILL_NAME 和 ONE_LINE_SUMMARY。行内运行——不要停止工作流。

**WebSearch 备用：** 如果 WebSearch 不可用，跳过搜索步骤并注明："Search unavailable — proceeding with in-distribution knowledge only."

## 贡献者模式

如果 `_CONTRIB` 为 `true`：你处于**贡献者模式**。你是 gstack 用户，也在帮助它变得更好。

**在每个主要工作流步骤结束时**（不是每个单独命令之后），反思你使用的 gstack 工具。评分 0 到 10。如果不是 10，想想为什么。如果 gstack 代码或 skill markdown 有明显可操作的 bug 或有趣可以做得更好的地方——提交现场报告。

**校准——这是标准：** 例如，`$B js "await fetch(...)"` 曾经因为 `SyntaxError: await is only valid in async functions` 失败，因为 gstack 没有在异步上下文中包装表达式。这是小的，但输入是合理的，gstack 应该处理——这类值得提交的东西。

**不值得提交：** 用户的 app bug、用户 URL 的网络错误、用户站点的认证失败、用户自己的 JS 逻辑 bug。

**值得提交：** 写 `~/.gstack/contributor-logs/{slug}.md`，包含**所有以下部分**：

```
# {标题}

嘿 gstack 团队——我在使用 /{skill-name} 时遇到了这个：

**我尝试做的事：** {用户/代理尝试做什么}
**实际发生的：** {实际发生了什么}
**我的评分：** {0-10} — {一句话说明为什么不是 10}

## 复现步骤
1. {步骤}

## 原始输出
```
{paste 实际错误或意外输出 here}
```

## 怎样才能达到 10
{一句话：gstack 应该如何不同地做}

**日期：** {YYYY-MM-DD} | **版本：** {gstack 版本} | **Skill：** /{skill}
```

Slug：小写、连字符、最多 60 个字符（例如 `browse-js-no-await`）。如果文件已存在则跳过。每个会话最多 3 份报告。行内提交并继续——不要停止工作流。告诉用户："已提交 gstack 现场报告：{标题}"

## 完成状态协议

完成 skill 工作流时，使用以下之一报告状态：
- **DONE** — 所有步骤成功完成。每个声明都提供了证据。
- **DONE_WITH_CONCERNS** — 完成，但有用户应该知道的问题。列出每个问题。
- **BLOCKED** — 无法继续。说明阻塞内容和已尝试的方法。
- **NEEDS_CONTEXT** — 缺少继续所需的信息。准确说明需要什么。

### 升级

说"这对我来说太难了"或"我对这个结果没有信心"永远是可以的。糟糕的工作比没有工作更糟糕。
- 如果你已经尝试任务 3 次都没有成功，停止并升级。
- 如果你对敏感变更不确定，停止并升级。
- 如果工作量超出你能验证的范围，停止并升级。

升级格式：
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2 句话]
ATTEMPTED: [你尝试了什么]
RECOMMENDATION: [用户下一步应该做什么]
```

## 遥测数据（最后运行）

skill 工作流完成后（成功、错误或中止），记录遥测事件。从本文件 YAML frontmatter 的 `name:` 字段确定 skill 名称。从工作流结果确定结果。

运行此 bash：

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

替换 `SKILL_NAME`、`OUTCOME`（success/error/abort）和 `USED_BROWSE`（true/false）。在后台运行，永远不会阻塞用户。

## 计划状态页脚

当你在计划模式中即将调用 ExitPlanMode 时：

1. 检查计划文件是否已有 `## GSTACK REVIEW REPORT` 部分。
2. 如果有——跳过（审查 skill 已经写了更丰富的报告）。
3. 如果没有——运行此命令：

```bash
~/.claude/skills/gstack/bin/gstack-review-read
```

然后将 `## GSTACK REVIEW REPORT` 部分写入计划文件末尾。如果输出包含审查条目，用标准报告表格式化；如果为空，写入占位符表。

## Step 0: 检测基准分支

确定此 PR 以哪个分支为目标。在所有后续步骤中将结果作为"基准分支"使用。

1. 检查此分支是否已存在 PR：
   `gh pr view --json baseRefName -q .baseRefName`
   如果成功，将打印的分支名作为基准分支。

2. 如果没有 PR（命令失败），检测仓库的默认分支：
   `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

3. 如果两个命令都失败，回退到 `main`。

打印检测到的基准分支名。在每个后续的 `git diff`、`git log`、`git fetch`、`git merge` 和 `gh pr create` 命令中，用检测到的分支名替换指令中说的"基准分支"。

---

## 前置 Skill 提议

当上面的设计文档检查打印"No design doc found"时，在继续之前提议前置 skill。

通过 AskUserQuestion 告诉用户：

> "没有找到此分支的设计文档。`/office-hours` 会生成结构化的问题陈述、前提挑战和已探索的替代方案——它能给这个审查更有力的输入。大约需要 10 分钟。设计文档是按功能而不是按产品——它捕捉的是这个特定变更背后的思考。"

选项：
- A) 现在运行 /office-hours（完成后我会从这里继续审查）
- B) 跳过——进行标准审查

如果他们跳过："没关系——标准审查。如果你想要更有力的输入，下次先运行 /office-hours。"然后正常继续。稍后不要再次提议。

如果他们选择 A：

说："正在内联运行 /office-hours。设计文档准备好后，我会从这里继续审查。"

使用 Read 工具从磁盘读取 office-hours skill 文件：
`~/.claude/skills/gstack/office-hours/SKILL.md`

内联遵循它，**跳过这些部分**（已由父 skill 处理）：
- 前置准备（优先执行）
- AskUserQuestion 格式
- 完整性原则——煮沸湖泊
- 构建前先搜索
- 贡献者模式
- 完成状态协议
- 遥测数据（最后运行）

如果读取失败（文件未找到），说：
"无法加载 /office-hours——进行标准审查。"

/office-hours 完成后，重新运行设计文档检查：
```bash
SLUG=$(~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null | tr '/' '-' || echo 'no-branch')
DESIGN=$(ls -t ~/.gstack/projects/$SLUG/*-$BRANCH-design-*.md 2>/dev/null | head -1)
[ -z "$DESIGN" ] && DESIGN=$(ls -t ~/.gstack/projects/$SLUG/*-design-*.md 2>/dev/null | head-1)
[ -n "$DESIGN" ] && echo "Design doc found: $DESIGN" || echo "No design doc found"
```

如果现在找到了设计文档，阅读它并继续审查。
如果没有生成（用户可能取消了），进行标准审查。

# /autoplan — 自动审查流水线

一条命令。粗糙计划输入，完整审查后的计划输出。

/autoplan 从磁盘读取完整的 CEO、设计和工程审查 skill 文件，并全深度遵循它们——与手动运行每个 skill 具有相同的严谨性、相同的部分、相同的方法论。唯一区别：中间的 AskUserQuestion 调用使用以下 6 条原则进行自动决策。品味决策（合理的人可能有分歧的地方）在最终批准门呈现。

---

## 6 条决策原则

这些规则自动回答每个中间问题：

1. **选择完整性** — 交付完整的东西。选择覆盖更多边缘情况的方案。
2. **煮沸湖泊** — 修复爆炸半径内的所有内容（本计划修改的文件 + 直接导入方）。自动批准爆炸半径内且 < 1 天 CC 工作量的扩展（< 5 个文件，无新基础设施）。
3. **务实** — 如果两个选项修复了相同的东西，选更干净的。5 秒钟选择，而不是 5 分钟。
4. **DRY** — 复制了现有功能？拒绝。重用已有的。
5. **显式优于巧妙** — 10 行明显修复 > 200 行抽象。选新贡献者 30 秒能读懂的那个。
6. **行动倾向** — 合并 > 审查循环 > 陈旧 deliberation。标记问题但不阻塞。

**冲突解决（上下文相关的决胜规则）：**
- **CEO 阶段：** P1（完整性）+ P2（煮湖）占主导。
- **工程阶段：** P5（显式）+ P3（务实）占主导。
- **设计阶段：** P5（显式）+ P1（完整性）占主导。

---

## 决策分类

每个自动决策都有分类：

**机械性** — 一个明显正确的答案。自动决定，不出声。
例如：运行 codex（总是 yes），运行 evals（总是 yes），在完整计划上缩减范围（总是 no）。

**品味性** — 合理的人可能有分歧。自动决定并提供建议，但在最终门呈现。三个自然来源：
1. **接近方案** — 前两名都是可行的，有不同的权衡。
2. **边界范围** — 在爆炸半径内但 3-5 个文件，或范围模糊。
3. **Codex 分歧** — codex 建议不同而且有有效论点。

---

## "自动决定"是什么意思

自动决定用 6 条原则**替换用户的判断**。它**不替换分析**。加载的 skill 文件中的每个部分仍然必须以与交互版本相同的深度执行。唯一改变的是谁回答 AskUserQuestion：是你，用 6 条原则，而不是用户。

**你必须仍然：**
- 阅读每个部分引用的实际代码、diff 和文件
- 生成部分要求的所有输出（图表、表格、注册表、工件）
- 识别部分设计捕获的每个问题
- 使用 6 条原则决定每个问题（而不是问用户）
- 在审计日志中记录每个决策
- 将所有必需的工件写入磁盘

**你必须不：**
- 将审查部分压缩成一行表格行
- 在没有显示你检查了什么的情况下写"没有问题"
- 跳过部分因为"它不适用"而没有说明你检查了什么以及为什么
- 用摘要代替要求的输出（例如，"架构看起来不错"而不是部分要求的 ASCII 依赖图）

"没有问题"是部分的有效输出——但只有在做完分析之后。在说明你检查了什么以及为什么没有标记（最少 1-2 句话）之后。"跳过"对非跳过列表中的部分永远无效。

---

## 阶段 0: 摄入 + 恢复点

### Step 1: 捕获恢复点

在做任何事情之前，将计划文件的当前状态保存到外部文件：

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null | tr '/' '-')
DATETIME=$(date +%Y%m%d-%H%M%S)
echo "RESTORE_PATH=$HOME/.gstack/projects/$SLUG/${BRANCH}-autoplan-restore-${DATETIME}.md"
```

将计划文件的完整内容写入恢复路径，包含此标题：
```
# /autoplan 恢复点
捕获时间：[时间戳] | 分支：[分支] | 提交：[短哈希]

## 重新运行指令
1. 将下面的"原始计划状态"复制回计划文件
2. 调用 /autoplan

## 原始计划状态
[逐字计划文件内容]
```

然后在计划文件开头添加一行 HTML 注释：
`<!-- /autoplan restore point: [RESTORE_PATH] -->`

### Step 2: 读取上下文

- 阅读 CLAUDE.md、TODOS.md、git log -30、git diff 针对基准分支的 --stat
- 发现设计文档：`ls -t ~/.gstack/projects/$SLUG/*-design-*.md 2>/dev/null | head-1`
- 检测 UI 范围：grep 计划中的视图/渲染术语（component、screen、form、button、modal、layout、dashboard、sidebar、nav、dialog）。要求 2+ 个匹配。排除误报（单独的"page"、"缩写中的 UI"）。

### Step 3: 从磁盘加载 skill 文件

使用 Read 工具读取每个文件：
- `~/.claude/skills/gstack/plan-ceo-review/SKILL.md`
- `~/.claude/skills/gstack/plan-design-review/SKILL.md`（仅当检测到 UI 范围时）
- `~/.claude/skills/gstack/plan-eng-review/SKILL.md`

**部分跳过列表——当遵循加载的 skill 文件时，跳过这些部分（已由 /autoplan 处理）：**
- 前置准备（优先执行）
- AskUserQuestion 格式
- 完整性原则——煮沸湖泊
- 构建前先搜索
- 贡献者模式
- 完成状态协议
- 遥测数据（最后运行）
- Step 0: 检测基准分支
- 审查准备仪表板
- 计划文件审查报告
- 前置 Skill 提议（BENEFITS_FROM）

仅遵循审查特定的方法论、部分和要求的输出。

输出："这是我要处理的：[计划摘要]。UI 范围：[是/否]。已从磁盘加载审查 skill。开始使用自动决策的全审查流水线。"

---

## 阶段 1: CEO 审查（策略与范围）

遵循 plan-ceo-review/SKILL.md——所有部分，全深度。
覆盖：每个 AskUserQuestion → 使用 6 条原则自动决定。

**覆盖规则：**
- 模式选择：选择性扩展
- 前提：接受合理的（P6），只挑战明显错误的
- **门：向用户呈现前提供确认**——这是唯一一个**不是**自动决定的 AskUserQuestion。前提需要人类判断。
- 替代方案：选择最高完整性（P1）。如果平局，选最简单的（P5）。如果前两名接近→标记为品味决策。
- 范围扩展：在爆炸半径内 + <1 天 CC → 批准（P2）。外部 → defer 到 TODOS.md（P3）。重复 → 拒绝（P4）。边界（3-5 个文件）→ 标记为品味决策。
- 所有 10 个审查部分：完全运行，自动决定每个问题，记录每个决策到审计日志。

**必需执行检查清单（CEO）：**

Step 0（0A-0F）——运行每个子步骤并生成：
- 0A: 前提挑战，命名和评估具体前提
- 0B: 现有代码利用图（子问题 → 现有代码）
- 0C: 梦想状态图（CURRENT → THIS PLAN → 12 个月理想状态）
- 0C-bis: 实现替代方案表（2-3 种方法，含工作量/风险/优点/缺点）
- 0D: 模式特定分析，记录范围决策
- 0E: 时间审问（小时 1 → 小时 6+）
- 0F: 模式选择确认

Sections 1-10——对于**每个**部分，从加载的 skill 文件运行评估标准：
- 有发现的部分：完整分析，自动决定每个问题，记录到审计日志
- 没有发现的部分：1-2 句话说明检查了什么以及为什么没有标记。永远不要将部分压缩成表格行。
- Section 11（设计）：仅当阶段 0 检测到 UI 范围时运行

**阶段 1 的必需输出：**
- "不在范围内"部分，含 defer 项目和理由
- "现有内容"部分，将子问题映射到现有代码
- 错误与救援注册表（来自 Section 2）
- 故障模式注册表（来自审查部分）
- 梦想状态增量（此计划让我们在哪里 vs 12 个月理想状态）
- 完成摘要（来自 CEO skill 的完整摘要表）

---

## 阶段 2: 设计审查（条件性——如果没有 UI 范围则跳过）

遵循 plan-design-review/SKILL.md——所有 7 个维度，全深度。
覆盖：每个 AskUserQuestion → 使用 6 条原则自动决定。

**覆盖规则：**
- 重点领域：所有相关维度（P1）
- 结构性问题（缺失状态、破坏层次结构）：自动修复（P5）
- 美学/品味问题：标记为品味决策
- 设计系统对齐：如果 DESIGN.md 存在且修复明显，则自动修复

---

## 阶段 3: 工程审查 + Codex

遵循 plan-eng-review/SKILL.md——所有部分，全深度。
覆盖：每个 AskUserQuestion → 使用 6 条原则自动决定。

**覆盖规则：**
- 范围挑战：永不缩减（P2）
- Codex 审查：如果可用则始终运行（P6）
  命令：`codex exec "Review this plan for architectural issues, missing edge cases, and hidden complexity. Be adversarial. File: <plan_path>" -s read-only --enable web_search_cached`
  超时：10 分钟，然后继续"Codex 超时——单一审查者模式"
- 架构选择：显式优于巧妙（P5）。如果 codex 不同意且有有效原因 → 品味决策。
- Evals：始终包含所有相关套件（P1）
- 测试计划：在 `~/.gstack/projects/$SLUG/{user}-{branch}-test-plan-{datetime}.md` 生成工件
- TODOS.md：收集阶段 1 的所有 defer 范围扩展，自动写入

**必需执行检查清单（工程）：**

1. Step 0（范围挑战）：阅读计划引用的实际代码。将每个子问题映射到现有代码。运行复杂度检查。生成具体发现。

2. Step 0.5（Codex）：如果可用则运行。在 CODEX SAYS 标题下呈现完整输出。

3. Section 1（架构）：生成 ASCII 依赖图，显示新组件及其与现有组件的关系。评估耦合、扩展、安全性。

4. Section 2（代码质量）：识别 DRY 违规、命名问题、复杂性。引用具体文件和模式。自动决定每个发现。

5. **Section 3（测试审查）——永不跳过或压缩。**
   此部分需要阅读实际代码，而不是从记忆中总结。
   - 阅读 diff 或计划的受影响文件
   - 构建测试图：列出每个新 UX 流程、数据流、代码路径和分支
   - 对于图中的**每个**项目：什么类型的测试覆盖它？是否存在？差距？
   - 对于 LLM/prompt 变更：必须运行哪些 eval 套件？
   - 自动决定测试差距意味着：识别差距 → 决定是添加测试还是 defer（附理由和原则）→ 记录决策。它不意味着跳过分析。
   - 将测试计划工件写入磁盘

6. Section 4（性能）：评估 N+1 查询、内存、缓存、慢路径。

**阶段 3 的必需输出：**
- "不在范围内"部分
- "现有内容"部分
- 架构 ASCII 图（Section 1）
- 将代码路径映射到覆盖率的测试图（Section 3）
- 写入磁盘的测试计划工件（Section 3）
- 带关键差距标志的故障模式注册表
- 完成摘要（来自工程 skill 的完整摘要）
- TODOS.md 更新（从所有阶段收集）

---

## 决策审计日志

在每个自动决策后，使用 Edit 追加一行到计划文件：

```markdown
<!-- AUTONOMOUS DECISION LOG -->
## 决策审计日志

| # | 阶段 | 决策 | 原则 | 理由 | 拒绝 |
|---|-------|----------|-----------|-----------|----------|
```

通过 Edit 逐步写入每行。这将审计保存在磁盘上，而不是积累在对话上下文中。

---

## 门前验证

在呈现最终批准门之前，验证要求的输出实际已生成。检查计划文件和对话中的每个项目。

**阶段 1（CEO）输出：**
- [ ] 前提挑战，命名具体前提（不只是"前提已接受"）
- [ ] 所有适用的审查部分有发现或明确的"检查了 X，没有标记"
- [ ] 错误与救援注册表已生成（或注明 N/A 及原因）
- [ ] 故障模式注册表已生成（或注明 N/A 及原因）
- [ ] "不在范围内"部分已写
- [ ] "现有内容"部分已写
- [ ] 梦想状态增量已写
- [ ] 完成摘要已生成

**阶段 2（设计）输出——仅当检测到 UI 范围时：**
- [ ] 所有 7 个维度已评估并打分
- [ ] 问题已识别并自动决定

**阶段 3（工程）输出：**
- [ ] 范围挑战含实际代码分析（不只是"范围没问题"）
- [ ] 架构 ASCII 图已生成
- [ ] 将代码路径映射到测试覆盖率的测试图
- [ ] 测试计划工件已写入磁盘 ~/.gstack/projects/$SLUG/
- [ ] "不在范围内"部分已写
- [ ] "现有内容"部分已写
- [ ] 故障模式注册表含关键差距评估
- [ ] 完成摘要已生成

**审计日志：**
- [ ] 决策审计日志每条自动决策至少有一行（不为空）

如果任何复选框缺失，返回并生成缺失的输出。最多重试 2 次——如果重试后仍然缺失，带着说明哪些项目不完整的警告继续到门。不要无限循环。

---

## 阶段 4: 最终批准门

**在这里停下来，向用户呈现最终状态。**

作为消息呈现，然后使用 AskUserQuestion：

```
## /autoplan 审查完成

### 计划摘要
[1-3 句话摘要]

### 已做决策：[N] 条（[M] 条自动决定，[K] 条需要你选择）

### 你的选择（品味决策）
[对于每个品味决策：]
**选择 [N]：[标题]**（来自 [阶段]）
我建议 [X]——[原则]。但 [Y] 也是可行的：
  [如果你选 Y，一句话下游影响]

### 自动决定：[M] 条决策 [见计划文件中的决策审计日志]

### 审查评分
- CEO：[摘要]
- 设计：[摘要或"跳过，无 UI 范围"]
- 工程：[摘要]
- Codex：[摘要或"不可用"]

### Defer 到 TODOS.md
[自动 defer 的项目及原因]
```

**认知负荷管理：**
- 0 个品味决策：跳过"你的选择"部分
- 1-7 个品味决策：平铺列表
- 8+ 个：按阶段分组。添加警告："此计划有异常高的模糊性（[N] 个品味决策）。仔细审查。"

AskUserQuestion 选项：
- A) 原样批准（接受所有建议）
- B) 批准但有覆盖（指定要更改哪些品味决策）
- C) 询问（询问任何具体决策）
- D) 修订（计划本身需要更改）
- E) 拒绝（重新开始）

**选项处理：**
- A：标记 APPROVED，写入审查日志，建议 /ship
- B：问哪些覆盖，应用，重新呈现门
- C：自由回答，重新呈现门
- D：做更改，重新运行受影响的阶段（范围→1B，设计→2，测试计划→3，架构→3）。最多 3 个循环。
- E：重新开始

---

## 完成：写入审查日志

批准后，写入 3 个单独的审查日志条目，以便 /ship 的仪表板识别它们：

```bash
COMMIT=$(git rev-parse --short HEAD 2>/dev/null)
TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)

~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"plan-ceo-review","timestamp":"'"$TIMESTAMP"'","status":"clean","unresolved":0,"critical_gaps":0,"mode":"SELECTIVE_EXPANSION","via":"autoplan","commit":"'"$COMMIT"'"}'

~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"plan-eng-review","timestamp":"'"$TIMESTAMP"'","status":"clean","unresolved":0,"critical_gaps":0,"issues_found":0,"mode":"FULL_REVIEW","via":"autoplan","commit":"'"$COMMIT"'"}'
```

如果阶段 2 运行了（UI 范围）：
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"plan-design-review","timestamp":"'"$TIMESTAMP"'","status":"clean","unresolved":0,"via":"autoplan","commit":"'"$COMMIT"'"}'
```

用审查中的实际计数替换字段值。

建议下一步：准备好时 `/ship`。

---

## 重要规则

- **永不中止。** 用户选择了 /autoplan。尊重这个选择。呈现所有品味决策，永远不要重定向到交互式审查。
- **前提是唯一的门。** 唯一一个非自动决定的 AskUserQuestion 是阶段 1 中的前提确认。
- **记录每个决策。** 没有无声的自动决策。每个选择都在审计日志中有一行。
- **全深度意味着全深度。** 不要压缩或跳过加载 skill 文件中的部分（除了阶段 0 的跳过列表）。"全深度"意味着：阅读部分要求你阅读的代码，生成部分要求的输出，识别每个问题，决定每个问题。如果你发现自己为任何审查部分写的少于 3 句话，你可能正在压缩。
- **工件是可交付成果。** 测试计划工件、故障模式注册表、错误/救援表、ASCII 图——审查完成时这些必须存在于磁盘或计划文件中。如果不存在，审查就不完整。
- **顺序执行。** CEO → 设计 → 工程。每个阶段建立在前一个之上。
