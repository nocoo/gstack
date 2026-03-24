---
name: retro
version: 2.0.0
description: |
  每周工程回顾。分析提交历史、工作模式和代码质量指标，具有持久历史和趋势追踪。
  团队感知：分解每个人的贡献，包含表扬和成长领域。
  当被要求"weekly retro"、"what did we ship"或"engineering retrospective"时使用。
  在工作周或 sprint 结束时主动建议。
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
  --skill "retro" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

## 计划状态页脚

在计划模式中调用 ExitPlanMode 前运行 `gstack-review-read` 并写入审查报告。

## 检测默认分支

确定仓库的默认分支名称：
`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`
如果失败，回退到 `main`。

---

# /retro — 每周工程回顾

生成全面的工程回顾，分析提交历史、工作模式和代码质量指标。团队感知：识别运行命令的用户，然后分析每个贡献者的贡献，提供个人表扬和成长机会。

## 参数
- `/retro` — 默认：过去 7 天
- `/retro 24h` — 过去 24 小时
- `/retro 14d` — 过去 14 天
- `/retro 30d` — 过去 30 天
- `/retro compare` — 比较当前周期与上一个相同长度周期
- `/retro global` — 跨项目回顾（跨所有 AI 编码工具，7d 默认）
- `/retro global 14d` — 跨项目，明确周期

---

## 工作流

### Step 1: 收集原始数据

1. 获取 origin 并识别当前用户：`git config user.name`
2. 运行多个 git 命令收集数据：
   - 所有提交（时间戳、主题、哈希、作者、文件更改、插入、删除）
   - 每个提交的测试与总 LOC 分解
   - 提交时间戳（用于会话检测）
   - 最频繁更改的文件（热点分析）
   - PR 编号
   - 每个作者的热点
   - 每个作者的提交计数
   - Greptile 分类历史
   - TODOS.md 待办
   - 测试文件计数
   - 回归测试提交
   - gstack skill 使用遥测

### Step 2: 计算指标

计算并显示摘要表：
- 提交数、贡献者、PR 合并数
- 总插入/删除、净 LOC
- 测试 LOC 和比例
- 版本范围、活动天数、检测到的会话数
- 平均 LOC/会话小时
- Greptile 信号
- 测试健康

### Step 3: 提交时间分布

显示每小时直方图（本地时区）：
- 峰值小时
- 死区
- 模式：双峰（早/晚）还是连续

### Step 4: 工作会话检测

使用 **45 分钟间隔** 阈值检测会话。报告：
- 开始/结束时间、提交数、持续时间
- 分类：深度会话（50+ 分钟）、中等会话（20-50 分钟）、微型会话（<20 分钟）
- 计算总活动时间、平均会话长度、LOC/小时

### Step 5: 提交类型分解

按常规提交前缀分类（feat/fix/refactor/test/chore/docs）。

### Step 6: 热点分析

显示前 10 个最频繁更改的文件。标记：
- 更改 5+ 次的文件（churn 热点）
- 测试文件 vs 生产文件

### Step 7: PR 大小分布

估计并分组 PR 大小：小（<100 LOC）、中（100-500 LOC）、大（500-1500 LOC）、XL（1500+ LOC）。

### Step 8: 专注评分 + 本周 Ship

**专注评分：** 计算触及最频繁更改的顶层目录的提交百分比。

**本周 Ship：** 自动识别窗口中最高 LOC 的 PR。

### Step 9: 团队成员分析

对每个贡献者计算：
1. 提交数和 LOC
2. 专注领域
3. 提交类型分布
4. 会话模式
5. 测试纪律
6. 最大 Ship

对当前用户：最深入的处理，包含所有细节。
对每个队友：2-3 句话覆盖他们工作内容和模式，然后：
- **表扬**（1-2 件具体的事）：锚定在实际的提交中
- **成长机会**（1 件具体的事）：作为成长建议，不是批评

### Step 10: 周对比趋势

如果窗口 >= 14 天，分成每周桶并显示趋势。

### Step 11: 连胜追踪

计算连续天数（至少 1 次提交）：
- 团队连胜
- 个人连胜

### Step 12: 加载历史并比较

检查之前的回顾历史并计算关键指标的 delta。

### Step 13: 保存回顾历史

保存 JSON 快照到 `.context/retros/`。

### Step 14: 写叙事

结构化输出：
- Tweetable 摘要
- 总结表
- 与上次回顾的趋势对比
- 时间和会话模式
- 交付速度
- 代码质量信号
- 测试健康
- 专注和亮点
- 你的本周（个人深度分析）
- 团队分解
- 前 3 名团队胜利
- 3 件改进事项
- 3 个下周习惯

---

## 全局回顾模式

当用户运行 `/retro global` 时，跳过正常的 repo 范围回顾，改为跨项目分析。

### 全局步骤 1: 计算时间窗口

午夜对齐逻辑，默认 7d。

### 全局步骤 2: 运行发现

定位并运行发现脚本。

### 全局步骤 3: 对每个 repo 运行 git log

### 全局步骤 4: 计算全局交付连胜

### 全局步骤 5: 计算上下文切换指标

### 全局步骤 6: 每工具生产力模式

### 全局步骤 7: 聚合和生成叙事

**分享的个人卡片**（第一）：
```
╔═══════════════════════════════════════════════════════════════
║  [USER NAME] — Week of [date]
╠═══════════════════════════════════════════════════════════════
║
║  [N] commits across [M] projects
║  +[X]k LOC added · [Y]k LOC deleted · [Z]k net
║  [N] AI coding sessions (CC: X, Codex: Y, Gemini: Z)
║  [N]-day shipping streak 🔥
╚═══════════════════════════════════════════════════════════════
```

### 全局步骤 8: 加载历史并比较

### 全局步骤 9: 保存快照

---

## 比较模式

当用户运行 `/retro compare` 时：
1. 计算当前窗口的指标
2. 计算上一个相同长度周期的指标（无重叠）
3. 显示并排比较表
4. 写简短叙事

---

## 语调

- 鼓励但坦诚
- 具体和可操作——始终锚定在实际提交/代码中
- 跳过泛泛的表扬
- 将改进框定为成长，不是批评
- 永远不要负面比较队友
- 总输出约 3000-4500 字

---

## 重要规则

- 所有叙事输出直接发送给用户。唯一写入的文件是 JSON 快照。
- 使用 `origin/<default>` 进行所有 git 查询
- 所有时间戳以用户本地时区显示
- 如果窗口没有提交，建议不同的窗口
- 全局模式：不需要在 git repo 内。保存快照到 `~/.gstack/retros/`。
