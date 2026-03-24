---
name: land-and-deploy
version: 1.0.0
description: |
  Land and deploy 工作流。合并 PR，等待 CI 和部署，通过 canary 检查验证生产健康。
  在 /ship 创建 PR 后接管。当被要求"merge"、"land"、"deploy"、
  "merge and verify"、"land it"、"ship it to production"时使用。
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
  --skill "land-and-deploy" --duration "$_TEL_DUR" --outcome "OUTCOME" \
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

# /land-and-deploy — 合并、部署、验证

你是发布工程师。负责高效合并、智能等待、彻底验证，给用户明确的结论。

此 skill 在 `/ship` 创建 PR 后接管。`/ship` 创建 PR，你合并它、等待部署、验证生产。

---

## 参数
- `/land-and-deploy` — 自动检测 PR，无 post-deploy URL
- `/land-and-deploy <url>` — 自动检测 PR，验证此 URL
- `/land-and-deploy #123` — 指定 PR 编号
- `/land-and-deploy #123 <url>` — 指定 PR + 验证 URL

---

## 工作流

### Step 1: 预检

1. 检查 GitHub CLI 认证
2. 解析参数
3. 从当前分支检测 PR
4. 验证 PR 状态

### Step 2: 合并前检查

检查 CI 状态和合并就绪情况。

### Step 3: 等待 CI（如果待定）

等待所需检查完成。使用 15 分钟超时。

### Step 3.5: 合并前就绪门

**关键安全检查。** 收集所有证据，建立就绪报告，获得明确用户确认。

检查：
- 审查陈旧性（审查后提交次数）
- 测试结果（免费测试、E2E 测试、LLM 评估）
- PR 正文准确性
- 文档发布检查

生成并显示就绪报告：
```
╔══════════════════════════════════════════════════════════╗
║              PRE-MERGE READINESS REPORT                  ║
╠══════════════════════════════════════════════════════════╣
║  PR: #NNN — title                                        ║
║  Branch: feature → main                                  ║
║                                                          ║
║  REVIEWS                                                 ║
║  ├─ Eng Review:    CURRENT / STALE (N commits) / —       ║
║  ├─ CEO Review:    CURRENT / — (optional)                ║
║  ├─ Design Review: CURRENT / — (optional)                ║
║  └─ Codex Review:  CURRENT / — (optional)                ║
║                                                          ║
║  TESTS                                                   ║
║  ├─ Free tests:    PASS / FAIL (blocker)               ║
║  ├─ E2E tests:     N/N pass / NOT RUN                 ║
║  └─ LLM evals:     PASS / NOT RUN                      ║
║                                                          ║
║  WARNINGS: N  |  BLOCKERS: N                             ║
╚══════════════════════════════════════════════════════════╝
```

使用 AskUserQuestion 确认。

### Step 4: 合并 PR

记录开始时间戳。尝试自动合并：
```bash
gh pr merge --auto --delete-branch
```

### Step 5: 部署策略检测

检测项目类型和如何验证部署：
- GitHub Actions 工作流
- Fly.io
- Render
- Vercel/Netlify
- Heroku
- 自定义

### Step 6: 等待部署（如果适用）

根据检测到的平台等待部署完成。

### Step 7: Canary 验证（条件深度）

根据 diff 范围确定 canary 深度：
- 仅文档 → 跳过
- 仅配置 → 冒烟检查
- 仅后端 → 控制台错误 + 性能检查
- 前端（任意）→ 完整 canary

### Step 8: 回滚（如果需要）

如果需要回滚：
```bash
git checkout <base>
git revert <merge-commit-sha> --no-edit
git push origin <base>
```

### Step 9: 部署报告

创建并显示 ASCII 摘要：
```
LAND & DEPLOY REPORT
═════════════════════
PR:           #<number> — <title>
Merged:       <timestamp>
Timing:
  CI wait:    <duration>
  Queue:      <duration>
  Deploy:     <duration>
  Canary:     <duration>
  Total:      <end-to-end duration>

CI:           <PASSED / SKIPPED>
Deploy:       <PASSED / FAILED / NO WORKFLOW>
Verification: <HEALTHY / DEGRADED / SKIPPED / REVERTED>

VERDICT: <DEPLOYED AND VERIFIED / REVERTED>
```

### Step 10: 建议后续操作

- 运行 `/canary <url>` 进行扩展监控
- 运行 `/benchmark <url>` 进行深度性能审计
- 运行 `/document-release` 更新项目文档

---

## 重要规则

- **永远不要强制推送。** 使用 `gh pr merge` 是安全的。
- **永远不要跳过 CI。** 如果检查失败，停止。
- **自动检测一切。** PR 编号、合并方法、部署策略、项目类型。只有在真正无法推断时才询问。
- **带退避的轮询。** 不要轰炸 GitHub API。CI/部署每 30 秒轮询一次。
- **回滚始终是一种选择。** 在每个失败点，提供回滚作为逃生舱。
- **单次验证，不是持续监控。** `/land-and-deploy` 检查一次。`/canary` 做扩展监控循环。
