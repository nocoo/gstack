---
name: ship
version: 1.0.0
description: |
  Ship 工作流：检测并合并基准分支、运行测试、审查 diff、bump VERSION、
  更新 CHANGELOG、提交、推送、创建 PR。
  当被要求"ship"、"deploy"、"push to main"、"create a PR"或"merge and push"时使用。
  当用户说代码已准备好或询问部署时主动建议。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
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
  --skill "ship" --duration "$_TEL_DUR" --outcome "OUTCOME" \
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

# /ship：Ship 工作流

你是发布工程师。负责高效交付代码到生产环境。

---

## 工作流

### Step 1: 预检

1. 检查 GitHub CLI 认证
2. 检测当前分支
3. 识别 PR 或准备创建

### Step 2: 检测并合并基准分支

```bash
git fetch origin <base> --quiet
git merge origin/<base> --no-edit
```

处理合并冲突。

### Step 3: 运行测试

```bash
npm test 2>&1 | tail -20
```

如果测试失败，停止。

### Step 4: 审查 Diff

分析相对于基准分支的变更：
- 新功能
- Bug 修复
- 重构
- 文档更新

### Step 5: Bump VERSION

如果需要版本升级：
```bash
# 读取当前版本
cat VERSION
# 升级版本
echo "1.0.0" > VERSION
```

### Step 6: 更新 CHANGELOG

追加新条目到 CHANGELOG.md：
```markdown
## [1.0.0] - YYYY-MM-DD
### Added
- [新功能描述]
### Fixed
- [修复描述]
```

### Step 7: 提交

```bash
git add .
git commit -m "feat: 新功能描述"
```

### Step 8: 推送

```bash
git push origin HEAD
```

### Step 9: 创建 PR

```bash
gh pr create --title "feat: 新功能描述" --body "## Changes
- 新功能
- 修复
" --base <base>
```

---

## Ship 报告

生成最终报告：
```
SHIP 报告
══════════════════
PR:       #<number>
分支:     <branch> → <base>
状态:     <SUCCESS / FAILED>
测试:     <PASSED / FAILED>
版本:     <旧版本> → <新版本>
提交:     <commit-sha>
```

---

## 重要规则

- **测试通过后才能 ship。**
- **每次 ship 都要更新 CHANGELOG。**
- **版本号遵循语义化版本。**
- **PR 描述要清晰。**
- **不要跳过任何步骤。**
