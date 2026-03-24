---
name: gstack
version: 1.1.0
description: |
  用于 QA 测试和站点自测的快速无头浏览器。导航页面、交互元素、验证状态、
  对比前后差异、拍摄带标注截图、测试响应式布局、表单、上传、对话框，
  以及捕获 bug 证据。当被要求打开或测试站点、验证部署、体验用户流程，
  或用截图提交 bug 时使用。
  也可按阶段建议相邻的 gstack skill：头脑风暴 /office-hours；策略 /plan-ceo-review；
  架构 /plan-eng-review；设计 /plan-design-review 或 /design-consultation；
  自动审查 /autoplan；调试 /investigate；QA /qa；代码审查 /review；
  视觉审查 /design-review；发布 /ship；文档 /document-release；复盘 /retro；
  二次确认 /codex；生产安全 /careful 或 /guard；局部编辑 /freeze 或 /unfreeze；
  gstack 升级 /gstack-upgrade。如果用户选择关闭建议，执行 gstack-config set proactive false；
  如果重新开启，执行 gstack-config set proactive true。
allowed-tools:
  - Bash
  - Read
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
echo '{"skill":"gstack","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

如果 `PROACTIVE` 为 `"false"`，不要主动建议 gstack skill——只在用户明确要求时才调用。
用户已选择关闭主动建议。

如果输出显示 `UPGRADE_AVAILABLE <旧版本> <新版本>`：阅读 `~/.claude/skills/gstack/gstack-upgrade/SKILL.md` 并遵循"内联升级流程"（如果已配置则自动升级，否则用 AskUserQuestion 提供 4 个选项，如果放弃则写入暂存状态）。如果显示 `JUST_UPGRADED <从> <到>`：告诉用户"正在运行 gstack v{到}（刚刚更新！）"并继续。

如果 `LAKE_INTRO` 为 `no`：在继续之前，介绍"完整性原则"。
告诉用户："gstack 遵循**煮沸湖泊**原则——当 AI 让边际成本接近零时，总是做完整的事情。
了解更多：https://garryslist.org/posts/boil-the-ocean"
然后提议在默认浏览器中打开这篇文章：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

只有用户同意时才运行 `open`。始终运行 `touch` 标记为已读。这只发生一次。

如果 `TEL_PROMPTED` 为 `no` 且 `LAKE_INTRO` 为 `yes`：在处理完湖泊介绍之后，
询问用户关于遥测数据。使用 AskUserQuestion：

> 帮助 gstack 变得更好！社区模式分享使用数据（你使用了哪些 skill、耗时多长、
> 崩溃信息）和一个稳定的设备 ID，以便我们追踪趋势并更快修复 bug。
> 不会发送任何代码、文件路径或仓库名称。
> 随时可以通过 `gstack-config set telemetry off` 更改。

选项：
- A) 帮助 gstack 变得更好！（推荐）
- B) 不用了，谢谢

如果选 A：运行 `~/.claude/skills/gstack/bin/gstack-config set telemetry community`

如果选 B：再问一个 AskUserQuestion：

> 那匿名模式呢？我们只知道**有人**使用了 gstack——没有唯一 ID，
> 无法关联会话。只是一个计数器，帮助我们了解是否有人在使用。

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
"EUREKA：每个人都做 X 因为 [假设]。但 [证据] 表明这是错的。Y 更好因为 [推理]。"

记录尤里卡时刻：
```bash
jq -n --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" --arg skill "SKILL_NAME" --arg branch "$(git branch --show-current 2>/dev/null)" --arg insight "ONE_LINE_SUMMARY" '{ts:$ts,skill:$skill,branch:$branch,insight:$insight}' >> ~/.gstack/analytics/eureka.jsonl 2>/dev/null || true
```
替换 SKILL_NAME 和 ONE_LINE_SUMMARY。行内运行——不要停止工作流。

**WebSearch 备用：** 如果 WebSearch 不可用，跳过搜索步骤并注明："Search unavailable — proceeding with in-distribution knowledge only."

## 贡献者模式

如果 `_CONTRIB` 为 `true`：你处于**贡献者模式**。你是 gstack 用户，也在帮助它变得更好。

**在每个主要工作流步骤结束时**（不是每个单独命令之后），反思你使用的 gstack 工具。评分 0 到 10。如果不是 10，想想为什么。如果 gstack 代码或 skill markdown 有明显可操作的 bug 或有趣可以做得更好的地方——提交现场报告。也许我们的贡献者会帮助我们变得更好！

**校准——这是标准：** 例如，`$B js "await fetch(...)"` 曾经因为 `SyntaxError: await is only valid in async functions` 失败，因为 gstack 没有在异步上下文中包装表达式。这是小的，但输入是合理的，gstack 应该处理——这类值得提交的东西。

**不值得提交：** 用户的 app bug、用户 URL 的网络错误、用户站点的认证失败、用户自己的 JS 逻辑 bug。

**值得提交：** 写 `~/.gstack/contributor-logs/{slug}.md`，包含**所有以下部分**（不要截断——包含每个部分直到日期/版本页脚）：

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

说"这对我来说太难了"或"我对这个结果没有信心"永远是可以的。

糟糕的工作比没有工作更糟糕。你不会因为升级而受到惩罚。
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

skill 工作流完成后（成功、错误或中止），记录遥测事件。
从本文件 YAML frontmatter 的 `name:` 字段确定 skill 名称。
从工作流结果确定结果（如果正常完成则为 success，如果失败则为 error，如果用户中断则为 abort）。

**计划模式例外——始终运行：** 此命令将遥测写入
`~/.gstack/analytics/`（用户配置目录，不是项目文件）。skill 前置准备已经写入同一目录——这是相同的模式。
跳过此命令会丢失会话时长和结果数据。

运行此 bash：

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

用实际 skill 名称替换 `SKILL_NAME`，用 success/error/abort 替换 `OUTCOME`，
根据是否使用了 `$B` 替换 `USED_BROWSE` 为 true/false。如果无法确定结果，使用"unknown"。
这在后台运行，永远不会阻塞用户。

## 计划状态页脚

当你在计划模式中即将调用 ExitPlanMode 时：

1. 检查计划文件是否已有 `## GSTACK REVIEW REPORT` 部分。
2. 如果有——跳过（审查 skill 已经写了更丰富的报告）。
3. 如果没有——运行此命令：

```bash
~/.claude/skills/gstack/bin/gstack-review-read
```

然后将 `## GSTACK REVIEW REPORT` 部分写入计划文件末尾：

- 如果输出包含审查条目（`---CONFIG---` 之前的 JSONL 行）：用每次运行/状态/发现的标准报告表格式化，与审查 skill 使用的格式相同。
- 如果输出为 `NO_REVIEWS` 或为空：写入此占位符表：

```markdown
## GSTACK REVIEW REPORT

| 审查 | 触发原因 | 为什么 | 运行次数 | 状态 | 发现 |
|--------|---------|-----|------|--------|------|
| CEO 审查 | `/plan-ceo-review` | 范围与策略 | 0 | — | — |
| Codex 审查 | `/codex review` | 独立的第二意见 | 0 | — | — |
| 工程审查 | `/plan-eng-review` | 架构与测试（必需） | 0 | — | — |
| 设计审查 | `/plan-design-review` | UI/UX 差距 | 0 | — | — |

**结论：** 尚无审查——运行 `/autoplan` 获取完整审查流程，或运行上方的各个审查。
```

**计划模式例外——始终运行：** 这会写入计划文件，这是你在计划模式下唯一允许编辑的文件。
计划文件审查报告是计划活动状态的一部分。

如果 `PROACTIVE` 为 `false`：在本会话期间不要主动建议其他 gstack skill。
只运行用户明确调用的 skill。此偏好通过 `gstack-config` 跨会话持久化。

# gstack browse：QA 测试与自测

持久化无头 Chromium。首次调用自动启动（约 3 秒），之后每次命令约 100-200ms。
空闲 30 分钟后自动关闭。状态在调用之间保持（cookies、标签页、会话）。

## 设置（在任何 browse 命令之前运行此检查）

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B=~/.claude/skills/gstack/browse/dist/browse
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

如果显示 `NEEDS_SETUP`：
1. 告诉用户："gstack browse 需要一次性构建（约 10 秒）。可以继续吗？"然后停止并等待。
2. 运行：`cd <SKILL_DIR> && ./setup`
3. 如果没有安装 `bun`：`curl -fsSL https://bun.sh/install | bash`

## 重要提示

- 通过 Bash 使用编译好的二进制文件：`$B <command>`
- 永远不要使用 `mcp__claude-in-chrome__*` 工具。它们慢且不可靠。
- 浏览器在调用之间保持状态——cookies、登录会话和标签页会保留。
- 对话框（alert/confirm/prompt）默认自动接受——浏览器不会卡住。
- **显示截图：** 在 `$B screenshot`、`$B snapshot -a -o` 或 `$B responsive` 之后，始终使用 Read 工具读取输出 PNG，以便用户能看到。没有这个，截图是看不见的。

## QA 工作流

### 测试用户流程（登录、注册、结账等）

```bash
# 1. 前往页面
$B goto https://app.example.com/login

# 2. 查看哪些可交互
$B snapshot -i

# 3. 使用 refs 填写表单
$B fill @e3 "test@example.com"
$B fill @e4 "password123"
$B click @e5

# 4. 验证结果
$B snapshot -D              # diff 显示点击后的变化
$B is visible ".dashboard"  # 断言仪表盘出现了
$B screenshot /tmp/after-login.png
```

### 验证部署 / 检查生产环境

```bash
$B goto https://yourapp.com
$B text                          # 读取页面——加载了吗？
$B console                       # 有 JS 错误吗？
$B network                       # 有失败的请求吗？
$B js "document.title"           # 标题正确吗？
$B is visible ".hero-section"    # 关键元素存在吗？
$B screenshot /tmp/prod-check.png
```

### 端到端体验功能

```bash
# 导航到功能
$B goto https://app.example.com/new-feature

# 拍摄带标注截图——显示每个带标签的交互元素
$B snapshot -i -a -o /tmp/feature-annotated.png

# 找到所有可点击的东西（包括 cursor:pointer 的 div）
$B snapshot -C

# 走完流程
$B snapshot -i          # 基线
$B click @e3            # 交互
$B snapshot -D          # 哪些变了？（统一 diff）

# 检查元素状态
$B is visible ".success-toast"
$B is enabled "#next-step-btn"
$B is checked "#agree-checkbox"

# 交互后检查控制台错误
$B console
```

### 测试响应式布局

```bash
# 快速：移动/平板/桌面 3 张截图
$B goto https://yourapp.com
$B responsive /tmp/layout

# 手动：特定视口
$B viewport 375x812     # iPhone
$B screenshot /tmp/mobile.png
$B viewport 1440x900    # 桌面
$B screenshot /tmp/desktop.png

# 元素截图（裁剪到特定元素）
$B screenshot "#hero-banner" /tmp/hero.png
$B snapshot -i
$B screenshot @e3 /tmp/button.png

# 区域裁剪
$B screenshot --clip 0,0,800,600 /tmp/above-fold.png

# 仅视口（不滚动）
$B screenshot --viewport /tmp/viewport.png
```

### 测试文件上传

```bash
$B goto https://app.example.com/upload
$B snapshot -i
$B upload @e3 /path/to/test-file.pdf
$B is visible ".upload-success"
$B screenshot /tmp/upload-result.png
```

### 测试带验证的表单

```bash
$B goto https://app.example.com/form
$B snapshot -i

# 提交空表单——检查验证错误出现
$B click @e10                        # 提交按钮
$B snapshot -D                       # diff 显示错误信息出现
$B is visible ".error-message"

# 填写并重新提交
$B fill @e3 "valid input"
$B click @e10
$B snapshot -D                       # diff 显示错误消失，成功状态
```

### 测试对话框（删除确认、提示）

```bash
# 在触发之前设置对话框处理
$B dialog-accept              # 将自动接受下一个 alert/confirm
$B click "#delete-button"     # 触发确认对话框
$B dialog                     # 查看出现了什么对话框
$B snapshot -D                # 验证项目被删除

# 对于需要输入的提示
$B dialog-accept "my answer"  # 带文本接受
$B click "#rename-button"      # 触发提示
```

### 测试需要认证的页面（导入真实浏览器 cookies）

```bash
# 从真实浏览器导入 cookies（打开交互式选择器）
$B cookie-import-browser

# 或直接导入特定域名
$B cookie-import-browser comet --domain .github.com

# 现在测试需要认证的页面
$B goto https://github.com/settings/profile
$B snapshot -i
$B screenshot /tmp/github-profile.png
```

### 对比两个页面 / 环境

```bash
$B diff https://staging.app.com https://prod.app.com
```

### 多步骤链（长流程的高效方式）

```bash
echo '[
  ["goto","https://app.example.com"],
  ["snapshot","-i"],
  ["fill","@e3","test@test.com"],
  ["fill","@e4","password"],
  ["click","@e5"],
  ["snapshot","-D"],
  ["screenshot","/tmp/result.png"]
]' | $B chain
```

## 快速断言模式

```bash
# 元素存在且可见
$B is visible ".modal"

# 按钮可用/不可用
$B is enabled "#submit-btn"
$B is disabled "#submit-btn"

# 复选框状态
$B is checked "#agree"

# 输入框可编辑
$B is editable "#name-field"

# 元素获得焦点
$B is focused "#search-input"

# 页面包含文本
$B js "document.body.textContent.includes('Success')"

# 元素数量
$B js "document.querySelectorAll('.list-item').length"

# 特定属性值
$B attrs "#logo"    # 以 JSON 返回所有属性

# CSS 属性
$B css ".button" "background-color"
```

## 快照系统

快照是你理解和交互页面的主要工具。

```
-i        --interactive           仅交互元素（按钮、链接、输入框），带 @e refs
-c        --compact               紧凑（无空结构节点）
-d <N>    --depth                 限制树深度（0 = 仅根，默认：无限）
-s <sel>  --selector              作用域限定为 CSS 选择器
-D        --diff                  与前一个快照的统一 diff（首次调用存储基线）
-a        --annotate              带红色覆盖框和 ref 标签的标注截图
-o <path> --output                标注截图输出路径（默认：<temp>/browse-annotated.png）
-C        --cursor-interactive    光标可交互元素（@c refs——带 pointer 光标和 onclick 的 div）
```

所有标志可以自由组合。`-o` 仅在同时使用 `-a` 时适用。
示例：`$B snapshot -i -a -C -o /tmp/annotated.png`

**Ref 编号：** @e refs 按树顺序顺序分配（@e1, @e2, ...）。
来自 `-C` 的 @c refs 单独编号（@c1, @c2, ...）。

快照后，使用 @refs 作为选择器：
```bash
$B click @e3       $B fill @e4 "value"     $B hover @e1
$B html @e2        $B css @e5 "color"      $B attrs @e6
$B click @c1       # 光标可交互 ref（来自 -C）
```

**输出格式：** 带 @ref ID 的缩进可访问性树，每个元素一行。
```
  @e1 [heading] "Welcome" [level=1]
  @e2 [textbox] "Email"
  @e3 [button] "Submit"
```

导航后 refs 失效——在 `goto` 后再次运行 `snapshot`。

## 命令参考

### 导航
| 命令 | 描述 |
|---------|-------------|
| `back` | 历史后退 |
| `forward` | 历史前进 |
| `goto <url>` | 导航到 URL |
| `reload` | 重新加载页面 |
| `url` | 打印当前 URL |

### 读取
| 命令 | 描述 |
|---------|-------------|
| `accessibility` | 完整 ARIA 树 |
| `forms` | 表单字段（JSON 格式） |
| `html [selector]` | 选择器的 innerHTML（未找到则抛出），无选择器则返回完整页面 HTML |
| `links` | 所有链接，格式为 "text → href" |
| `text` | 清理后的页面文本 |

### 交互
| 命令 | 描述 |
|---------|-------------|
| `click <sel>` | 点击元素 |
| `cookie <name>=<value>` | 在当前页面域名设置 cookie |
| `cookie-import <json>` | 从 JSON 文件导入 cookies |
| `cookie-import-browser [browser] [--domain d]` | 从 Comet、Chrome、Arc、Brave 或 Edge 导入 cookies（打开选择器，或用 --domain 直接导入） |
| `dialog-accept [text]` | 自动接受下一个 alert/confirm/prompt。可选文本作为 prompt 响应发送 |
| `dialog-dismiss` | 自动关闭下一个对话框 |
| `fill <sel> <val>` | 填写输入框 |
| `header <name>:<value>` | 设置自定义请求头（冒号分隔，敏感值自动编辑） |
| `hover <sel>` | 悬停在元素上 |
| `press <key>` | 按键——Enter、Tab、Escape、ArrowUp/Down/Left/Right、Backspace、Delete、Home、End、PageUp、PageDown，或修饰符如 Shift+Enter |
| `scroll [sel]` | 将元素滚动到视口，或如果没有选择器则滚动到页面底部 |
| `select <sel> <val>` | 通过 value、label 或可见文本选择下拉选项 |
| `type <text>` | 键入到焦点元素 |
| `upload <sel> <file> [file2...]` | 上传文件 |
| `useragent <string>` | 设置 user agent |
| `viewport <WxH>` | 设置视口大小 |
| `wait <sel|--networkidle|--load>` | 等待元素、网络空闲或页面加载（超时：15s） |

### 检查
| 命令 | 描述 |
|---------|-------------|
| `attrs <sel|@ref>` | 元素属性（JSON 格式） |
| `console [--clear|--errors]` | 控制台消息（--errors 过滤为 error/warning） |
| `cookies` | 所有 cookies（JSON 格式） |
| `css <sel> <prop>` | 计算后的 CSS 值 |
| `dialog [--clear]` | 对话框消息 |
| `eval <file>` | 从文件运行 JavaScript 并将结果作为字符串返回（路径必须在 /tmp 或 cwd 下） |
| `is <prop> <sel>` | 状态检查（visible/hidden/enabled/disabled/checked/editable/focused） |
| `js <expr>` | 运行 JavaScript 表达式并将结果作为字符串返回 |
| `network [--clear]` | 网络请求 |
| `perf` | 页面加载计时 |
| `storage [set k v]` | 读取所有 localStorage + sessionStorage（JSON），或设置 <key> <value> 来写入 localStorage |

### 视觉
| 命令 | 描述 |
|---------|-------------|
| `diff <url1> <url2>` | 两个页面之间的文本 diff |
| `pdf [path]` | 保存为 PDF |
| `responsive [prefix]` | 移动端（375x812）、平板（768x1024）、桌面（1280x720）截图。保存为 {prefix}-mobile.png 等 |
| `screenshot [--viewport] [--clip x,y,w,h] [selector|@ref] [path]` | 保存截图（支持通过 CSS/@ref 裁剪元素、--clip 区域、--viewport） |

### 快照
| 命令 | 描述 |
|---------|-------------|
| `snapshot [flags]` | 带 @e refs 的可访问性树，用于元素选择。标志：-i 仅交互、-c 紧凑、-d N 深度限制、-s sel 作用域、-D 与前一个 diff、-a 标注截图、-o 路径输出、-C 光标可交互 @c refs |

### Meta
| 命令 | 描述 |
|---------|-------------|
| `chain` | 从 JSON stdin 运行命令。格式：[["cmd","arg1",...],...] |

### 标签页
| 命令 | 描述 |
|---------|-------------|
| `closetab [id]` | 关闭标签页 |
| `newtab [url]` | 打开新标签页 |
| `tab <id>` | 切换到标签页 |
| `tabs` | 列出打开的标签页 |

### 服务器
| 命令 | 描述 |
|---------|-------------|
| `handoff [message]` | 在当前页面打开可见 Chrome 以便用户接管 |
| `restart` | 重启服务器 |
| `resume` | 在用户接管后重新快照，将控制权返回给 AI |
| `status` | 健康检查 |
| `stop` | 关闭服务器 |

## 技巧

1. **导航一次，查询多次。** `goto` 加载页面；然后 `text`、`js`、`screenshot` 都立即作用于已加载的页面。
2. **先使用 `snapshot -i`。** 查看所有交互元素，然后用 ref 点击/填写。不用猜 CSS 选择器。
3. **使用 `snapshot -D` 来验证。** 基线 → 操作 → diff。看到具体哪些变了。
4. **使用 `is` 做断言。** `is visible .modal` 比解析页面文本更快更可靠。
5. **使用 `snapshot -a` 采集证据。** 标注截图非常适合 bug 报告。
6. **对复杂 UI 使用 `snapshot -C`。** 找到可访问性树遗漏的可点击 div。
7. **操作后检查 `console`。** 捕捉不直观显示的 JS 错误。
8. **长流程使用 `chain`。** 单命令，无每步 CLI 开销。
