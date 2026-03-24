---
name: setup-browser-cookies
version: 1.0.0
description: |
  从你的真实浏览器（Comet、Chrome、Arc、Brave、Edge）导入 cookies 到无头浏览器会话。
  打开交互式选择器 UI，让你选择要导入的 cookie 域名。
  在 QA 测试需要认证的页面之前使用。当被要求"import cookies"、"login to the site"或"authenticate the browser"时使用。
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
echo '{"skill":"setup-browser-cookies","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

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
  --skill "setup-browser-cookies" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

## 计划状态页脚

在计划模式中调用 ExitPlanMode 前运行 `gstack-review-read` 并写入审查报告。

# 设置浏览器 Cookies

从你的真实 Chromium 浏览器导入已登录的会话到无头浏览器会话。

## 工作原理

1. 找到 browse 二进制文件
2. 运行 `cookie-import-browser` 检测已安装的浏览器并打开选择器 UI
3. 用户在浏览器中选择要导入的 cookie 域名
4. Cookies 被解密并加载到 Playwright 会话中

## 步骤

### 1. 找到 browse 二进制文件

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

### 2. 打开 cookie 选择器

```bash
$B cookie-import-browser
```

这会自动检测已安装的 Chromium 浏览器（Comet、Chrome、Arc、Brave、Edge）并在你的默认浏览器中打开交互式选择器 UI，你可以：
- 在已安装的浏览器之间切换
- 搜索域名
- 点击 "+" 导入一个域名的 cookies
- 点击垃圾桶删除已导入的 cookies

告诉用户：**"Cookie 选择器已打开——在浏览器中选择你想导入的域名，完成后告诉我。"**

### 3. 直接导入（替代方案）

如果用户直接指定了域名（例如 `/setup-browser-cookies github.com`），跳过 UI：

```bash
$B cookie-import-browser comet --domain github.com
```

如果指定了其他浏览器，将 `comet` 替换为相应的浏览器。

### 4. 验证

用户确认完成后：

```bash
$B cookies
```

向用户显示已导入 cookies 的摘要（域名计数）。

## 注意

- 每个浏览器的首次导入可能会触发 macOS 钥匙串对话框——点击"允许"/"始终允许"
- Cookie 选择器在与 browse 服务器相同的端口上运行（没有额外进程）
- UI 中只显示域名和 cookie 计数——不暴露 cookie 值
- Browse 会话在命令之间保持 cookies，所以导入的 cookies 立即生效
