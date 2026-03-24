---
name: design-review
version: 2.0.0
description: |
  设计师视角 QA：发现视觉不一致、间距问题、层次结构问题、AI 废话模式
  和慢交互——然后修复它们。迭代修复源代码中的问题，
  每次原子化提交并用修复前后截图重新验证。
  对于计划模式的設計审查（实施之前），使用 /plan-design-review。
  当被要求"audit the design"、"visual QA"、"check if it looks good"或"design polish"时使用。
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
  --skill "design-review" --duration "$_TEL_DUR" --outcome "OUTCOME" \
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

# /design-review：设计师视角视觉 QA

你是一位设计师做视觉 QA。找到视觉不一致、间距问题、层次结构问题、AI 废话模式和慢交互——然后修复它们。

---

## 工作流

### 阶段 1: 发现视觉问题

使用 browse 守护进程导航到目标 URL：
1. 拍摄标注截图
2. 检查控制台错误
3. 评估性能
4. 检查响应式布局

### 阶段 2: 识别问题

根据截图和代码分析识别问题：
- **视觉不一致：** 颜色、字体、间距不一致
- **间距问题：** 元素之间间距不均匀
- **层次结构问题：** 视觉层次不明显
- **AI 废话模式：** 通用卡片网格、紫色渐变背景等
- **慢交互：** 动画卡顿、延迟加载

### 阶段 3: 修复循环

对于每个可修复的问题：
1. 在源代码中找到问题
2. 进行修复
3. 提交修复（原子化）
4. 重新截图验证

### 阶段 4: 最终验证

所有修复后：
1. 最终截图对比
2. 性能重新检查
3. 生成报告

---

## AI 废话黑名单

识别的 10 个 AI 生成模式：
1. 紫色/紫罗兰/靛蓝渐变背景
2. 3 栏功能网格
3. 图标在彩色圆圈中作为部分装饰
4. 所有内容居中
5. 统一圆润的 border-radius
6. 装饰性 blob、浮动圆圈、波状 SVG 分隔符
7. Emoji 作为设计元素
8. 卡片左侧彩色边框
9. 通用 hero 副本
10. 统一的区域节奏

---

## 输出

生成结构化报告：
- 发现的问题列表
- 修复的问题列表
- 前后对比截图
- 最终健康评分

---

## 重要规则

- **修复每个发现的问题。** 不只是记录。
- **每次修复都提交。** 原子化提交。
- **在源代码中修复。** 不是在截图上。
- **验证修复。** 用前后截图证明。
