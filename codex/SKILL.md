---
name: codex
version: 1.0.0
description: |
  OpenAI Codex CLI 封装——三种模式。代码审查：通过 codex review 进行独立 diff 审查，
  含 pass/fail 门。挑战：对抗模式，尝试打破你的代码。
  咨询：可以向 codex 提问任何问题，支持会话连续性以便跟进。
  "200 IQ 自闭症开发者"的第二意见。当被要求"codex review"、"codex challenge"、
  "ask codex"、"second opinion"或"consult codex"时使用。
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
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
echo '{"skill":"codex","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

如果 `PROACTIVE` 为 `"false"`，不要主动建议 gstack skill。
如果 `LAKE_INTRO` 为 `no`：介绍完整性原则。
如果 `TEL_PROMPTED` 为 `no`：询问遥测数据。

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

在构建基础设施之前——**先搜索。** 阅读 `~/.claude/skills/gstack/ETHOS.md`。

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
  --skill "codex" --duration "$_TEL_DUR" --outcome "OUTCOME" \
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

# /codex — 多AI第二意见

你正在运行 `/codex` skill。这是 OpenAI Codex CLI 的封装，从不同的 AI 系统获得独立、残酷诚实的第二意见。

Codex 是"200 IQ 自闭症开发者"——直接、简洁、技术精确、挑战假设、捕获你可能错过的东西。忠实地呈现它的输出，不要总结。

---

## Step 0: 检查 codex 二进制文件

```bash
CODEX_BIN=$(which codex 2>/dev/null || echo "")
[ -z "$CODEX_BIN" ] && echo "NOT_FOUND" || echo "FOUND: $CODEX_BIN"
```

如果 `NOT_FOUND`：停止并告诉用户：
"Codex CLI 未找到。安装：`npm install -g @openai/codex` 或访问 https://github.com/openai/codex"

---

## Step 1: 检测模式

解析用户输入确定运行哪种模式：

1. `/codex review` 或 `/codex review <instructions>` — **审查模式**（Step 2A）
2. `/codex challenge` 或 `/codex challenge <focus>` — **挑战模式**（Step 2B）
3. `/codex` 无参数 — **自动检测：**
   - 检查 diff（如果 origin 不可用则回退）：
     `git diff origin/<base> --stat 2>/dev/null | tail -1 || git diff <base> --stat 2>/dev/null | tail -1`
   - 如果存在 diff，使用 AskUserQuestion：
     ```
     Codex 检测到相对于基准分支有变更。它应该做什么？
     A) 审查 diff（带 pass/fail 门的代码审查）
     B) 挑战 diff（对抗性的——尝试打破它）
     C) 其他——我会提供一个 prompt
     ```
   - 如果没有 diff，检查计划文件：
     `ls -t ~/.claude/plans/*.md 2>/dev/null | xargs grep -l "$(basename $(pwd))" 2>/dev/null | head -1`
   - 如果存在计划文件，提供审查它
   - 否则问："你想问 Codex 什么？"
4. `/codex <其他内容>` — **咨询模式**（Step 2C），其余文本作为 prompt

---

## Step 2A: 审查模式

对当前分支 diff 运行 Codex 代码审查。

1. 创建临时文件用于输出捕获：
```bash
TMPERR=$(mktemp /tmp/codex-err-XXXXXX.txt)
```

2. 运行审查（5分钟超时）：
```bash
codex review --base <base> -c 'model_reasoning_effort="xhigh"' --enable web_search_cached 2>"$TMPERR"
```

使用 `timeout: 300000`。如果用户提供了自定义指令：
```bash
codex review "focus on security" --base <base> -c 'model_reasoning_effort="xhigh"' --enable web_search_cached 2>"$TMPERR"
```

3. 捕获输出。从 stderr 解析成本：
```bash
grep "tokens used" "$TMPERR" 2>/dev/null || echo "tokens: unknown"
```

4. 通过检查输出中的 critical 发现来确定门裁决。
   如果输出包含 `[P1]`——门是 **FAIL**。
   如果没有 `[P1]` 标记（只有 `[P2]` 或没有发现）——门是 **PASS**。

5. 呈现输出：

```
CODEX SAYS (代码审查):
════════════════════════════════════════════════════════════
<完整 codex 输出，逐字——不要截断或总结>
════════════════════════════════════════════════════════════
门: PASS                    Tokens: 14,331 | 估计成本: ~$0.12
```

或

```
门: FAIL (N 个 critical 发现)
```

6. **跨模型比较：** 如果 `/review`（Claude 自己的审查）之前已在此对话中运行过，
   比较两组发现。

7. 保存审查结果：
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"codex-review","timestamp":"TIMESTAMP","status":"STATUS","gate":"GATE","findings":N,"findings_fixed":N}'
```

8. 清理临时文件：
```bash
rm -f "$TMPERR"
```

## 计划文件审查报告

在对话输出中显示审查准备仪表板后，也更新**计划文件**本身，以便任何阅读计划的人都能看到审查状态。

### 检测计划文件

1. 检查此对话中是否有活动计划文件。
2. 如果未找到，静默跳过——并非每个审查都在计划模式中运行。

### 生成报告

从审查准备仪表板步骤读取已拥有的审查日志输出。解析每个 JSONL 条目。

生成 markdown 表格并写入计划文件末尾。

---

## Step 2B: 挑战（对抗）模式

Codex 尝试打破你的代码——寻找边缘情况、竞争条件、安全漏洞和正常审查会错过的故障模式。

1. 构建对抗性 prompt。如果用户提供了重点领域（例如 `/codex challenge security`），包含它：

默认 prompt（无重点）：
"审查此分支相对于基准分支的变更。运行 `git diff origin/<base>` 查看 diff。你的工作是找出这段代码在生产中将如何失败。像攻击者和混沌工程师一样思考。寻找边缘情况、竞争条件、安全漏洞、资源泄漏、故障模式和静默数据损坏路径。保持对抗性。保持彻底。不需要赞美——只有问题。"

有重点（如 "security"）：
"审查此分支相对于基准分支的变更。运行 `git diff origin/<base>` 查看 diff。重点关注安全。你的工作是找出攻击者如何利用这段代码。考虑注入向量、身份验证绕过、权限提升、数据暴露和时序攻击。保持对抗性。"

2. 使用 **JSONL 输出**运行 codex exec 以捕获推理轨迹和工具调用（5分钟超时）。

3. 呈现完整流式输出。

---

## Step 2C: 咨询模式

向 Codex 询问关于代码库的任何问题。支持跟进时会话连续性。

1. **检查现有会话：**
```bash
cat .context/codex-session-id 2>/dev/null || echo "NO_SESSION"
```

2. 创建临时文件：
```bash
TMPRESP=$(mktemp /tmp/codex-resp-XXXXXX.txt)
TMPERR=$(mktemp /tmp/codex-err-XXXXXX.txt)
```

3. **计划审查自动检测：** 如果用户 prompt 是关于审查计划，或存在计划文件：
```bash
ls -t ~/.claude/plans/*.md 2>/dev/null | xargs grep -l "$(basename $(pwd))" 2>/dev/null | head -1
```
读取计划文件并前置 persona 到用户 prompt。

4. 使用 **JSONL 输出**运行 codex exec（5分钟超时）。

5. 从流式输出捕获会话 ID。保存到 `.context/codex-session-id`。

6. 呈现完整流式输出。

7. 呈现后，注意 Codex 分析与你自己理解不同的任何点。如果有分歧，标记它。

---

## 模型与推理

**模型：** 没有硬编码模型——codex 使用其当前的默认模型（前沿代理编码模型）。
这意味着随着 OpenAI 发布新模型，/codex 自动使用它们。
如果用户想要特定模型，通过 `-m` 传递给 codex。

**推理努力：** 所有模式都使用 `xhigh`——最大推理能力。

**网络搜索：** 所有 codex 命令都使用 `--enable web_search_cached`。

---

## 成本估算

从 stderr 解析 token 计数。显示为：`Tokens: N`

---

## 错误处理

- **二进制文件未找到：** Step 0 中检测到。停止并提供安装说明。
- **认证错误：** Codex 向 stderr 输出认证错误。呈现错误并提供 `codex login` 说明。
- **超时：** 如果 Bash 调用超时（5分钟），告诉用户。
- **空响应：** 如果响应为空，告诉用户检查 stderr 错误。
- **会话恢复失败：** 如果恢复失败，删除会话文件并重新开始。

---

## 重要规则

- **永不修改文件。** 此 skill 是只读的。Codex 以只读沙箱模式运行。
- **逐字呈现输出。** 在显示之前不要截断、总结或编辑 Codex 的输出。
- **先呈现，后综合。** 任何 Claude 评论都在完整输出之后。
- **5分钟超时** 在所有 codex 的 Bash 调用上。
- **不要双重审查。** 如果用户已经运行了 `/review`，Codex 提供第二个独立意见。不要重新运行 Claude Code 自己的审查。
