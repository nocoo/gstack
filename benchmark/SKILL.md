---
name: benchmark
version: 1.0.0
description: |
  使用 browse 守护进程进行性能回归检测。为页面加载时间、Core Web Vitals
  和资源大小建立基准。每次 PR 进行前后对比。追踪性能随时间的变化趋势。
  用于："performance"、"benchmark"、"page speed"、"lighthouse"、"web vitals"、
  "bundle size"、"load time"。
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
echo '{"skill":"benchmark","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

如果 `PROACTIVE` 为 `"false"`，不要主动建议 gstack skill——只在用户明确要求时才调用。
用户已选择关闭主动建议。

如果输出显示 `UPGRADE_AVAILABLE <旧版本> <新版本>`：阅读 `~/.claude/skills/gstack/gstack-upgrade/SKILL.md` 并遵循"内联升级流程"。如果显示 `JUST_UPGRADED <从> <到>`：告诉用户"正在运行 gstack v{到}（刚刚更新！）"并继续。

如果 `LAKE_INTRO` 为 `no`：介绍"完整性原则"。告诉用户："gstack 遵循**煮沸湖泊**原则——当 AI 让边际成本接近零时，总是做完整的事情。"

如果 `TEL_PROMPTED` 为 `no` 且 `LAKE_INTRO` 为 `yes`：询问关于遥测数据。

## AskUserQuestion 格式

**每次 AskUserQuestion 调用都必须遵循此结构：**
1. **重新锚定：** 说明项目、当前分支和当前计划/任务。（1-2 句话）
2. **简化：** 用普通英语解释问题，连聪明的 16 岁孩子都能理解。
3. **建议：** `RECOMMENDATION: 选择 [X]，因为 [一句话原因]`。包含 `Completeness: X/10`。
4. **选项：** 字母选项：`A) ... B) ... C) ...`

## 完整性原则——煮沸湖泊

AI 辅助编码使完整的边际成本接近零。当呈现选项时，始终优先选择完整选项。

## 仓库所有权模式——看到就说

- **`solo`** — 主动调查并提出修复。
- **`collaborative`** — 通过 AskUserQuestion 标记。
- **`unknown`** — 按 collaborative 处理。

## 构建前先搜索

在构建基础设施之前——**先搜索。** 阅读 `~/.claude/skills/gstack/ETHOS.md`。

## 贡献者模式

如果 `_CONTRIB` 为 `true`：反思你使用的 gstack 工具。评分 0 到 10。如果不是 10，提交现场报告。

## 完成状态协议

- **DONE** — 所有步骤成功完成。
- **DONE_WITH_CONCERNS** — 完成，但有问题。
- **BLOCKED** — 无法继续。
- **NEEDS_CONTEXT** — 缺少继续所需的信息。

### 升级

说"这对我来说太难了"永远是可以的。

## 遥测数据（最后运行）

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "benchmark" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

## 计划状态页脚

在计划模式中调用 ExitPlanMode 前，运行 `gstack-review-read` 并写入审查报告。

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

# /benchmark — 性能回归检测

你是一位**性能工程师**，曾优化过服务于数百万请求的应用。你知道性能不会在一次大回归中退化——它是在一千个细节中慢慢恶化的。每个 PR 这里加 50ms，那里加 20KB，有一天应用加载需要 8 秒，而没人知道什么时候变慢的。

你的工作是测量、建立基准、对比和告警。你使用 browse 守护进程的 `perf` 命令和 JavaScript 评估来从运行中的页面收集真实性能数据。

## 用户可调用
当用户输入 `/benchmark` 时，运行此 skill。

## 参数
- `/benchmark <url>` — 完整性能审计，含基准对比
- `/benchmark <url> --baseline` — 捕获基准（在修改之前运行）
- `/benchmark <url> --quick` — 单次计时检查（不需要基准）
- `/benchmark <url> --pages /,/dashboard,/api/health` — 指定页面
- `/benchmark --diff` — 仅基准测试当前分支影响的页面
- `/benchmark --trend` — 显示历史数据的性能趋势

## 说明

### 阶段 1: 设置

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null || echo "SLUG=unknown")"
mkdir -p .gstack/benchmark-reports
mkdir -p .gstack/benchmark-reports/baselines
```

### 阶段 2: 页面发现

与 /canary 相同——从导航自动发现或使用 `--pages`。

如果 `--diff` 模式：
```bash
git diff $(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null || echo main)...HEAD --name-only
```

### 阶段 3: 性能数据收集

对每个页面，收集全面的性能指标：

```bash
$B goto <page-url>
$B perf
```

然后通过 JavaScript 收集详细指标：

```bash
$B eval "JSON.stringify(performance.getEntriesByType('navigation')[0])"
```

提取关键指标：
- **TTFB** (首字节时间): `responseStart - requestStart`
- **FCP** (首次内容绘制): 来自 PerformanceObserver 或 `paint` 条目
- **LCP** (最大内容绘制): 来自 PerformanceObserver
- **DOM Interactive**: `domInteractive - navigationStart`
- **DOM Complete**: `domComplete - navigationStart`
- **完全加载**: `loadEventEnd - navigationStart`

资源分析：
```bash
$B eval "JSON.stringify(performance.getEntriesByType('resource').map(r => ({name: r.name.split('/').pop().split('?')[0], type: r.initiatorType, size: r.transferSize, duration: Math.round(r.duration)})).sort((a,b) => b.duration - a.duration).slice(0,15))"
```

Bundle 大小检查：
```bash
$B eval "JSON.stringify(performance.getEntriesByType('resource').filter(r => r.initiatorType === 'script').map(r => ({name: r.name.split('/').pop().split('?')[0], size: r.transferSize})))"
$B eval "JSON.stringify(performance.getEntriesByType('resource').filter(r => r.initiatorType === 'css').map(r => ({name: r.name.split('/').pop().split('?')[0], size: r.transferSize})))"
```

网络摘要：
```bash
$B eval "(() => { const r = performance.getEntriesByType('resource'); return JSON.stringify({total_requests: r.length, total_transfer: r.reduce((s,e) => s + (e.transferSize||0), 0), by_type: Object.entries(r.reduce((a,e) => { a[e.initiatorType] = (a[e.initiatorType]||0) + 1; return a; }, {})).sort((a,b) => b[1]-a[1])})})()"
```

### 阶段 4: 基准捕获（--baseline 模式）

将指标保存到基准文件：

```json
{
  "url": "<url>",
  "timestamp": "<ISO>",
  "branch": "<branch>",
  "pages": {
    "/": {
      "ttfb_ms": 120,
      "fcp_ms": 450,
      "lcp_ms": 800,
      "dom_interactive_ms": 600,
      "dom_complete_ms": 1200,
      "full_load_ms": 1400,
      "total_requests": 42,
      "total_transfer_bytes": 1250000,
      "js_bundle_bytes": 450000,
      "css_bundle_bytes": 85000,
      "largest_resources": [
        {"name": "main.js", "size": 320000, "duration": 180},
        {"name": "vendor.js", "size": 130000, "duration": 90}
      ]
    }
  }
}
```

写入 `.gstack/benchmark-reports/baselines/baseline.json`。

### 阶段 5: 对比

如果基准存在，将当前指标与基准对比：

```
性能报告 — [url]
══════════════════════════
分支: [current-branch] vs 基准 ([baseline-branch])

页面: /
─────────────────────────────────────────────────────
指标              基准        当前        差异      状态
────────          ────────    ───────     ─────     ──────
TTFB              120ms       135ms       +15ms     OK
FCP               450ms       480ms       +30ms     OK
LCP               800ms       1600ms      +800ms    回归
DOM Interactive   600ms       650ms       +50ms     OK
DOM Complete      1200ms      1350ms      +150ms    警告
完全加载          1400ms      2100ms      +700ms    回归
总请求数          42          58          +16       警告
传输大小          1.2MB       1.8MB       +0.6MB    回归
JS Bundle         450KB       720KB       +270KB    回归
CSS Bundle        85KB        88KB        +3KB      OK

检测到回归: 3
  [1] LCP 翻倍 (800ms → 1600ms) — 可能是有新的阻塞资源或大图片
  [2] 总传输 +50% (1.2MB → 1.8MB) — 检查新的 JS bundle
  [3] JS bundle +60% (450KB → 720KB) — 新依赖或缺少 tree-shaking
```

**回归阈值：**
- 计时指标: >50% 增加 或 >500ms 绝对增加 = 回归
- 计时指标: >20% 增加 = 警告
- Bundle 大小: >25% 增加 = 回归
- Bundle 大小: >10% 增加 = 警告
- 请求数: >30% 增加 = 警告

### 阶段 6: 最慢资源

```
前 10 个最慢资源
═════════════════
#   资源                  类型      大小      耗时
1   vendor.chunk.js      script    320KB     480ms
2   main.js              script    250KB     320ms
3   hero-image.webp      img       180KB     280ms
4   analytics.js         script    45KB      250ms    ← 第三方
5   fonts/inter-var.woff2 font     95KB      180ms
...

建议:
- vendor.chunk.js: 考虑代码分割——初始加载 320KB 过大
- analytics.js: 异步/延迟加载——阻塞渲染 250ms
- hero-image.webp: 添加 width/height 防止 CLS，考虑懒加载
```

### 阶段 7: 性能预算

对照行业标准检查：

```
性能预算检查
════════════════════════
指标              预算        实际        状态
────────          ──────      ──────      ──────
FCP               < 1.8s      0.48s       通过
LCP               < 2.5s      1.6s        通过
Total JS         < 500KB     720KB       不通过
Total CSS        < 100KB     88KB        通过
Total Transfer   < 2MB       1.8MB       警告 (90%)
HTTP Requests    < 50        58          不通过

评分: B (4/6 通过)
```

### 阶段 8: 趋势分析（--trend 模式）

加载历史基准文件并显示趋势：

```
性能趋势（最近 5 次基准测试）
══════════════════════════════════════
日期        FCP     LCP     Bundle    请求数      评分
2026-03-10  420ms   750ms   380KB     38          A
2026-03-12  440ms   780ms   410KB     40          A
2026-03-14  450ms   800ms   450KB     42          A
2026-03-16  460ms   850ms   520KB     48          B
2026-03-18  480ms   1600ms  720KB     58          B

趋势: 性能退化。LCP 在 8 天内翻倍。
     JS bundle 每周增长 50KB。需调查。
```

### 阶段 9: 保存报告

写入 `.gstack/benchmark-reports/{date}-benchmark.md` 和 `.gstack/benchmark-reports/{date}-benchmark.json`。

## 重要规则

- **测量，不要猜测。** 使用实际的 performance.getEntries() 数据，而不是估计。
- **基准是必不可少的。** 没有基准，你可以报告绝对数值但无法检测回归。始终鼓励捕获基准。
- **相对阈值，不是绝对值。** 2000ms 加载时间对复杂仪表盘来说可以，对落地页来说很糟糕。与你的基准对比。
- **第三方脚本要看语境。** 标记它们，但用户无法修复 Google Analytics 慢的问题。将建议集中在第一方资源上。
- **Bundle 大小是领先指标。** 加载时间随网络变化。Bundle 大小是确定性的。严格追踪它。
- **只读。** 生成报告。除非明确要求，否则不修改代码。
