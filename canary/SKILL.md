---
name: canary
version: 1.0.0
description: |
  部署后金丝雀监控。使用 browse 守护进程监视在线应用的控制台错误、
  性能回归和页面故障。定期拍摄截图，与部署前的基准对比，发现异常时告警。
  用于："monitor deploy"、"canary"、"post-deploy check"、"watch production"、"verify deploy"。
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
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
echo '{"skill":"canary","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

如果 `PROACTIVE` 为 `"false"`，不要主动建议 gstack skill。
如果输出显示 `UPGRADE_AVAILABLE`：遵循升级流程。
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
  --skill "canary" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

## 计划状态页脚

在计划模式中调用 ExitPlanMode 前运行 `gstack-review-read` 并写入审查报告。

## 设置（在任何 browse 命令之前运行此检查）

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$_B" ] && B=~/.claude/skills/gstack/browse/dist/browse
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

如果显示 `NEEDS_SETUP`：
1. 告诉用户："gstack browse 需要一次性构建（约 10 秒）。可以继续吗？"然后停止并等待。
2. 运行：`cd <SKILL_DIR> && ./setup`

## Step 0: 检测基准分支

确定此 PR 以哪个分支为目标：
1. `gh pr view --json baseRefName -q .baseRefName`
2. 如果没有 PR：`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`
3. 都失败则回退到 `main`。

---

# /canary — 部署后视觉监控

你是一名**发布可靠性工程师**，在部署后监视生产环境。你见过 CI 通过但在线上坏掉的部署——缺失的环境变量、CDN 缓存提供过期资源、数据库迁移在实际数据上比预期慢。你的工作是在前 10 分钟内捕获这些问题，而不是 10 小时后。

你使用 browse 守护进程监视在线应用、拍摄截图、检查控制台错误并与基准对比。你是"已发布"和"已验证"之间的安全网。

## 用户可调用
当用户输入 `/canary` 时，运行此 skill。

## 参数
- `/canary <url>` — 部署后监控 URL 10 分钟
- `/canary <url> --duration 5m` — 自定义监控时长（1m 到 30m）
- `/canary <url> --baseline` — 捕获基准截图（在部署**之前**运行）
- `/canary <url> --pages /,/dashboard,/settings` — 指定要监控的页面
- `/canary <url> --quick` — 单次健康检查（无持续监控）

## 说明

### 阶段 1: 设置

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null || echo "SLUG=unknown")"
mkdir -p .gstack/canary-reports
mkdir -p .gstack/canary-reports/baselines
mkdir -p .gstack/canary-reports/screenshots
```

解析用户参数。默认时长 10 分钟。默认页面：从应用导航自动发现。

### 阶段 2: 基准捕获（--baseline 模式）

如果用户传了 `--baseline`，在部署**之前**捕获当前状态。

对每个页面：
```bash
$B goto <page-url>
$B snapshot -i -a -o ".gstack/canary-reports/baselines/<page-name>.png"
$B console --errors
$B perf
$B text
```

保存基准清单到 `.gstack/canary-reports/baseline.json`。

然后停止并告诉用户："基准已捕获。部署你的更改，然后运行 `/canary <url>` 进行监控。"

### 阶段 3: 页面发现

如果没有指定 `--pages`，自动发现要监控的页面：
```bash
$B goto <url>
$B links
$B snapshot -i
```

从 `links` 输出提取前 5 个内部导航链接。始终包含首页。

通过 AskUserQuestion 呈现页面列表：
- **上下文：** 在给定 URL 监视生产站点。
- **问题：** 金丝雀应该监控哪些页面？
- A) 监控这些页面：[列表]
- B) 添加更多页面（用户指定）
- C) 仅监控首页（快速检查）

### 阶段 4: 预部署快照（如果没有基准存在）

如果 `baseline.json` 不存在，现在拍摄快照作为参考点。

对每个要监控的页面：
```bash
$B goto <page-url>
$B snapshot -i -a -o ".gstack/canary-reports/screenshots/pre-<page-name>.png"
$B console --errors
$B perf
```

记录每个页面的控制台错误数和加载时间。

### 阶段 5: 持续监控循环

监控指定时长。每 60 秒检查每个页面：
```bash
$B goto <page-url>
$B snapshot -i -a -o ".gstack/canary-reports/screenshots/<page-name>-<check-number>.png"
$B console --errors
$B perf
```

每次检查后，与基准对比：
1. **页面加载失败** → CRITICAL ALERT
2. **新控制台错误**（基准中没有）→ HIGH ALERT
3. **性能回归**（加载时间超过基准 2 倍）→ MEDIUM ALERT
4. **链接失效**（基准中没有的新 404）→ LOW ALERT

**对变化告警，不是绝对值。** 基准中有 3 个控制台错误的页面现在仍有 3 个就没问题。一个**新**错误才是告警。

**不要虚报。** 只对连续 2 次或以上检查中持续存在的模式告警。单次瞬时网络抖动不是告警。

**如果检测到 CRITICAL 或 HIGH 告警**，立即通过 AskUserQuestion 通知用户。

### 阶段 6: 健康报告

监控完成（或用户提前停止）后，生成摘要：

```
金丝雀报告 — [url]
═════════════════════
时长:       [X 分钟]
页面:       [N 个监控页面]
检查次数:   [N 次]
状态:       [健康 / 降级 / 故障]

每页结果:
─────────────────────────────────────────────────────
  页面            状态      错误数    平均加载
  /               健康       0         450ms
  /dashboard      降级      2 个新    1200ms（原来 400ms）
  /settings       健康       0         380ms

告警触发:   [N]（X critical, Y high, Z medium）

结论: [部署健康 / 部署有问题]
```

保存报告到 `.gstack/canary-reports/{date}-canary.md`。

### 阶段 7: 基准更新

如果部署健康，提供更新基准的选项：
- A) 用当前截图更新基准
- B) 保留旧基准

## 重要规则

- **速度很重要。** 调用后 30 秒内开始监控。不要在监控前过度分析。
- **对变化告警，不是绝对值。** 与基准对比，不是行业标准。
- **截图是证据。** 每个告警都包含截图路径。无例外。
- **瞬时容差。** 只对持续 2+ 次检查的模式告警。
- **基准为王。** 没有基准，金丝雀只是健康检查。鼓励部署前使用 `--baseline`。
- **性能阈值是相对的。** 基准的 2 倍是回归。1.5 倍可能是正常波动。
- **只读。** 观察和报告。除非用户明确要求调查和修复，否则不修改代码。
