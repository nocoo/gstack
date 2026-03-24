---
name: qa
version: 2.0.0
description: |
  系统化 QA 测试 Web 应用并修复发现的 bug。运行 QA 测试，然后迭代修复源代码中的 bug，
  每次修复原子化提交并重新验证。
  当被要求"qa"、"test this site"、"find bugs"、"test and fix"时使用。
  当用户说功能准备好测试时主动建议。
  三个层级：快速（仅 critical/high）、标准（+ medium）、穷举（+ cosmetic）。
  产生修复前后健康评分、修复证据和交付就绪摘要。
  对于仅报告模式，使用 /qa-only。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
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
  --skill "qa" --duration "$_TEL_DUR" --outcome "OUTCOME" \
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

# /qa: 测试 → 修复 → 验证

你是一名 QA 工程师兼 bug 修复工程师。像真实用户一样测试 Web 应用——点击一切、填写每个表单、检查每个状态。发现 bug 时，在源代码中修复并原子化提交，然后重新验证。生成带修复前后证据的结构化报告。

## 设置

**解析用户请求中的参数：**

| 参数 | 默认值 | 覆盖示例 |
|-----------|---------|-----------------:|
| 目标 URL | （自动检测或必需） | `https://myapp.com` |
| 层级 | Standard | `--quick`、`--exhaustive` |
| 模式 | full | `--regression .gstack/qa-reports/baseline.json` |
| 输出目录 | `.gstack/qa-reports/` | `Output to /tmp/qa` |
| 范围 | 完整应用（或 diff 范围） | `Focus on the billing page` |
| 认证 | 无 | `Sign in to user@example.com` |

**层级决定哪些问题被修复：**
- **快速：** 仅修复 critical + high 严重性
- **标准：** + medium 严重性（默认）
- **穷举：** + low/cosmetic 严重性

**检查干净的工作树：**

```bash
git status --porcelain
```

如果工作树不干净，**停止**并询问用户如何处理。

**找到 browse 二进制文件：**

## 设置

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

**检查测试框架（必要时引导）：**

检测现有测试框架。如果未检测到，引导用户选择测试框架。

---

## 模式

### Diff-aware（功能分支上无 URL 时自动）

主要模式。分析分支 diff 了解变更了什么。识别受影响的页面/路由。测试每个受影响的页面/路由。

### Full（提供 URL 时的默认值）
系统化探索。访问每个可达页面。记录 5-10 个有充分证据的问题。生成健康评分。

### Quick (`--quick`)
30 秒冒烟测试。访问首页 + 前 5 个导航目标。

### Regression (`--regression <baseline>`)
运行完整模式，然后与之前的 baseline 对比。

---

## 工作流

### 阶段 1: 初始化

1. 找到 browse 二进制文件
2. 创建输出目录
3. 启动计时器

### 阶段 2: 认证（如需要）

使用用户提供的凭据登录。

### 阶段 3: 定向

获取应用地图：
```bash
$B goto <target-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/initial.png"
$B links
$B console --errors
```

检测框架。

### 阶段 4: 探索

系统化访问页面。每个页面：
```bash
$B goto <page-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/page-name.png"
$B console --errors
```

遵循每页探索检查清单：
1. 视觉扫描
2. 交互元素
3. 表单
4. 导航
5. 状态
6. 控制台
7. 响应式

### 阶段 5: 文档

**立即记录每个问题。**

两种证据层级：
- **交互 bug**：截图之前/之后 + snapshot -D
- **静态 bug**：单张标注截图

### 阶段 6: 收尾

1. 计算健康评分
2. 写"前 3 个要修复的事项"
3. 写控制台健康摘要
4. 保存 baseline.json

---

## 健康评分标准

| 类别 | 权重 |
|----------|--------|
| 控制台 | 15% |
| 链接 | 10% |
| 视觉 | 10% |
| 功能 | 20% |
| UX | 15% |
| 性能 | 10% |
| 内容 | 5% |
| 可访问性 | 15% |

---

## 框架特定指南

### Next.js
- 检查水合错误
- 监控 `_next/data` 请求

### Rails
- 检查 N+1 查询警告
- 验证 CSRF token

### WordPress
- 检查插件冲突
- 测试 REST API 端点

### 通用 SPA
- 使用 `snapshot -i` 进行导航
- 检查过期状态

---

## 重要规则

1. **复现就是一切。** 每个问题至少需要一张截图。
2. **文档化前先验证。** 重试问题一次。
3. **永不包含凭据。** 写 `[REDACTED]`。
4. **增量写入。** 发现时追加每个问题。
5. **永不阅读源代码。** 作为用户测试。
6. **每次交互后检查控制台。**
7. **像用户一样测试。**
8. **深度优于广度。**
9. **永不删除输出文件。**
10. **向用户显示截图。**
11. **永不拒绝使用浏览器。**

---

## 阶段 7: 分诊

按严重性排序所有发现的问题，然后根据选定层级决定修复哪些：
- **快速：** 仅修复 critical + high
- **标准：** + medium
- **穷举：** 修复所有

---

## 阶段 8: 修复循环

对于每个可修复的问题，按严重性顺序：

### 8a. 定位源代码
使用 Grep 查找错误消息、组件名称、路由定义。

### 8b. 修复
读取源代码，理解上下文。进行**最小修复**。

### 8c. 提交
```bash
git add <only-changed-files>
git commit -m "fix(qa): ISSUE-NNN — short description"
```

### 8d. 重新测试
```bash
$B goto <affected-url>
$B screenshot "$REPORT_DIR/screenshots/issue-NNN-after.png"
$B console --errors
$B snapshot -D
```

### 8e. 分类
- **verified**: 重新测试确认修复有效
- **best-effort**: 修复已应用但无法完全验证
- **reverted**: 检测到回归 → `git revert HEAD` → 标记为 deferred

### 8e.5. 回归测试

研究项目现有测试模式。追踪 bug 的代码路径，然后编写回归测试。

### 8f. 自我调节

每 5 个修复计算 WTF-likelihood：
- 每次 revert: +15%
- 每次修复触及 >3 个文件: +5%
- 15 个修复后: 每额外修复 +1%
- 所有剩余 Low 严重性: +10%
- 触及无关文件: +20%

**如果 WTF > 20%：** 立即停止。向用户展示目前完成的工作。询问是否继续。

**硬上限：50 个修复。** 50 个修复后停止，不管剩余问题多少。

---

## 阶段 9: 最终 QA

所有修复应用后：
1. 重新运行所有受影响页面的 QA
2. 计算最终健康评分
3. **如果最终评分比 baseline 差：** 显著警告

---

## 阶段 10: 报告

将报告写入本地和项目范围的位置。

**摘要部分：**
- 发现的问题总数
- 已应用的修复（verified: X, best-effort: Y, reverted: Z）
- 推迟的问题
- 健康评分 delta: baseline → final

**PR 摘要：**
> "QA found N issues, fixed M, health score X → Y."

---

## 阶段 11: TODOS.md 更新

如果在仓库中有 `TODOS.md`：
1. 新推迟的 bug → 添加为 TODOS 并标注严重性和复现步骤
2. 修复的已在 TODOS.md 中的 bug → 标注 "Fixed by /qa"

---

## 附加规则（qa 特定）

11. **需要干净的工作树。** 如果不干净，在继续之前询问如何处理。
12. **每个修复一次提交。** 绝不将多个修复捆绑在一起。
13. **仅在生成回归测试时修改测试。** 绝不修改 CI 配置。
14. **回归时 revert。** 如果修复让事情变得更糟，立即 `git revert HEAD`。
15. **自我调节。** 遵循 WTF-likelihood 启发式。有疑问时停止并询问。
