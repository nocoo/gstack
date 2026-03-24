---
name: setup-deploy
version: 1.0.0
description: |
  为 /land-and-deploy 配置部署设置。检测你的部署平台（Fly.io、Render、Vercel、Netlify、Heroku、GitHub Actions、自定义）、
  生产 URL、健康检查端点和部署状态命令。将配置写入 CLAUDE.md 以便未来所有部署都自动进行。
  当被要求"setup deploy"、"configure deployment"、"set up land-and-deploy"、
  "how do I deploy with gstack"、"add deploy config"时使用。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
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
  --skill "setup-deploy" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

## 计划状态页脚

在计划模式中调用 ExitPlanMode 前运行 `gstack-review-read` 并写入审查报告。

# /setup-deploy — 配置 gstack 的部署

你正在帮助用户配置他们的部署，以便 `/land-and-deploy` 自动工作。
你的工作是检测部署平台、生产 URL、健康检查和部署状态命令——然后将所有内容持久化到 CLAUDE.md。

运行一次后，`/land-and-deploy` 读取 CLAUDE.md 并完全跳过检测。

## 用户可调用
当用户输入 `/setup-deploy` 时，运行此 skill。

## 说明

### Step 1: 检查现有配置

```bash
grep -A 20 "## Deploy Configuration" CLAUDE.md 2>/dev/null || echo "NO_CONFIG"
```

如果配置已存在，显示并询问：
- **上下文：** CLAUDE.md 中已存在部署配置。
- **RECOMMENDATION：** 如果你的设置已更改，选择 A 更新。
- A) 从头重新配置（覆盖现有）
- B) 编辑特定字段
- C) 完成——配置看起来正确

如果用户选 C，停止。

### Step 2: 检测平台

运行部署 bootstrap 中的平台检测。

### Step 3: 平台特定设置

#### Fly.io

如果检测到 `fly.toml`：
1. 提取应用名称
2. 检查 `fly` CLI 是否已安装
3. 推断 URL：`https://{app}.fly.dev`
4. 设置部署状态命令：`fly status --app {app}`
5. 设置健康检查

#### Render

如果检测到 `render.yaml`：
1. 提取服务名称和类型
2. 检查 Render API key
3. 推断 URL：`https://{service-name}.onrender.com`
4. Render 在推送到连接分支时自动部署

#### Vercel

如果检测到 vercel.json 或 .vercel：
1. 检查 `vercel` CLI
2. Vercel 在推送时自动部署

#### Netlify

如果检测到 `netlify.toml`：
1. 提取站点信息
2. Netlify 在推送时自动部署

#### 仅 GitHub Actions

如果检测到部署工作流但没有平台配置：
1. 读取工作流文件以了解其作用
2. 询问用户生产 URL

#### 自定义/手动

如果未检测到任何内容：
使用 AskUserQuestion 收集信息：
1. **部署如何触发？**
2. **生产 URL 是什么？**
3. **gstack 如何检查部署是否成功？**
4. **有任何 pre-merge 或 post-merge hooks 吗？**

### Step 4: 写入配置

读取或创建 CLAUDE.md。找到并替换 `## Deploy Configuration` 部分（如果存在），或在末尾附加。

```markdown
## Deploy Configuration (configured by /setup-deploy)
- Platform: {platform}
- Production URL: {url}
- Deploy workflow: {workflow file or "auto-deploy on push"}
- Deploy status command: {command or "HTTP health check"}
- Merge method: {squash/merge/rebase}
- Project type: {web app / API / CLI / library}
- Post-deploy health check: {health check URL or command}

### Custom deploy hooks
- Pre-merge: {command or "none"}
- Deploy trigger: {command or "automatic on push to main"}
- Deploy status: {command or "poll production URL"}
- Health check: {URL or command}
```

### Step 5: 验证

写入后，验证配置是否有效：
1. 如果配置了健康检查 URL，尝试它
2. 如果配置了部署状态命令，尝试它

报告结果。如果任何失败，注明但不阻塞——配置仍然有用。

### Step 6: 摘要

```
部署配置 — 完成
════════════════════════════════
平台:      {platform}
URL:           {url}
健康检查:  {health check}
状态命令:    {status command}
合并方法:  {merge method}

已保存到 CLAUDE.md。/land-and-deploy 将自动使用这些设置。

下一步：
- 运行 /land-and-deploy 合并并部署你当前的 PR
- 编辑 CLAUDE.md 中的 "## Deploy Configuration" 部分以更改设置
- 再次运行 /setup-deploy 以重新配置
```

## 重要规则

- **永不暴露秘密。** 不要打印完整的 API keys、tokens 或 passwords。
- **与用户确认。** 写入前始终显示检测到的配置并要求确认。
- **CLAUDE.md 是事实来源。** 所有配置都放在那里——不是在单独的配置文件。
- **幂等。** 多次运行 /setup-deploy 会干净地覆盖之前的配置。
- **平台 CLI 是可选的。** 如果未安装 `fly` 或 `vercel` CLI，回退到基于 URL 的健康检查。
