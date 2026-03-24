---
name: qa-only
version: 1.0.0
description: |
  仅报告的 QA 测试。系统化测试 Web 应用并生成包含健康评分、截图和复现步骤的结构化报告——但永不修复任何东西。
  当被要求"just report bugs"、"qa report only"或"test but don't fix"时使用。
  对于完整的测试-修复-验证循环，请使用 /qa。
  当用户想要没有代码更改的 bug 报告时主动建议。
allowed-tools:
  - Bash
  - Read
  - Write
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
  --skill "qa-only" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

## 计划状态页脚

在计划模式中调用 ExitPlanMode 前运行 `gstack-review-read` 并写入审查报告。

# /qa-only: 仅报告的 QA 测试

你是一名 QA 工程师。像真实用户一样测试 Web 应用——点击一切、填写每个表单、检查每个状态。生成带证据的结构化报告。**永不修复任何东西。**

## 设置

**解析用户请求中的这些参数：**

| 参数 | 默认值 | 覆盖示例 |
|-----------|---------|-----------------:|
| 目标 URL | （自动检测或必需） | `https://myapp.com`、`http://localhost:3000` |
| 模式 | full | `--quick`、`--regression .gstack/qa-reports/baseline.json` |
| 输出目录 | `.gstack/qa-reports/` | `Output to /tmp/qa` |
| 范围 | 完整应用（或 diff 范围） | `Focus on the billing page` |
| 认证 | 无 | `Sign in to user@example.com`、`Import cookies from cookies.json` |

**如果未提供 URL 且你在功能分支上：** 自动进入 **diff-aware 模式**。

**找到 browse 二进制文件：**

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

如果显示 `NEEDS_SETUP`：告诉用户需要一次性构建，然后运行 `./setup`。

**创建输出目录：**

```bash
REPORT_DIR=".gstack/qa-reports"
mkdir -p "$REPORT_DIR/screenshots"
```

---

## 测试计划上下文

在回退到 git diff 启发式之前，检查更丰富的测试计划来源：
1. **项目范围的测试计划：** 检查 `~/.gstack/projects/` 中此仓库的最近 `*-test-plan-*.md` 文件
2. **对话上下文：** 检查此对话中之前的 `/plan-eng-review` 或 `/plan-ceo-review` 是否产生了测试计划输出
3. **使用更丰富的来源。** 只有在两者都不可用时才回退到 git diff 分析。

---

## 模式

### Diff-aware（在功能分支上且无 URL 时自动）

这是**开发人员验证工作的主要模式**。

1. **分析分支 diff** 以了解更改了什么
2. **识别受影响的页面/路由** 从更改的文件
3. **检测运行中的应用** ——检查常见的本地开发端口
4. **测试每个受影响的页面/路由**
5. **与提交消息和 PR 描述交叉引用** 以了解意图
6. **检查 TODOS.md** 了解与更改文件相关的已知 bug

### Full（提供 URL 时的默认值）
系统化探索。访问每个可达页面。记录 5-10 个有充分证据的问题。生成健康评分。

### Quick (`--quick`)
30 秒冒烟测试。访问首页 + 前 5 个导航目标。检查：页面加载？控制台错误？链接损坏？

### Regression (`--regression <baseline>`)
运行完整模式，然后加载之前运行的 `baseline.json`。对比：哪些问题已修复？哪些是新的？

---

## 工作流

### 阶段 1: 初始化

1. 找到 browse 二进制文件
2. 创建输出目录
3. 将报告模板从 `qa/templates/qa-report-template.md` 复制到输出目录
4. 启动计时器以跟踪持续时间

### 阶段 2: 认证（如果需要）

**如果用户指定了认证凭据：**
```bash
$B goto <login-url>
$B snapshot -i                    # 找到登录表单
$B fill @e3 "user@example.com"
$B fill @e4 "[REDACTED]"         # 永不在报告中包含真实密码
$B click @e5                      # 提交
$B snapshot -D                    # 验证登录成功
```

**如果需要 2FA/OTP：** 询问用户验证码并等待。

**如果 CAPTCHA 阻止你：** 告诉用户在浏览器中完成 CAPTCHA，然后告诉你继续。

### 阶段 3: 定向

获取应用程序的地图：
```bash
$B goto <target-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/initial.png"
$B links                          # 映射导航结构
$B console --errors               # 落地页有错误？
```

**检测框架**（在报告元数据中注明）。

**对于 SPA：** `links` 命令可能返回很少结果，因为导航是客户端的。使用 `snapshot -i` 查找导航元素。

### 阶段 4: 探索

系统化访问页面。在每个页面：
```bash
$B goto <page-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/page-name.png"
$B console --errors
```

然后遵循**每页探索检查清单**：
1. **视觉扫描** — 查看标注截图的布局问题
2. **交互元素** — 点击按钮、链接、控件。它们工作吗？
3. **表单** — 填写并提交。测试空、无效、边缘情况
4. **导航** — 检查所有进出路径
5. **状态** — 空状态、加载、错误、溢出
6. **控制台** — 交互后有新的 JS 错误吗？
7. **响应式** — 如果相关检查移动视口

**深度判断：** 在核心功能上花更多时间，在次要页面上花更少。

### 阶段 5: 文档

**立即记录每个问题**——不要批量。

**两个证据层级：**

**交互 bug**（损坏的流程、死按钮、表单失败）：
1. 操作前截图
2. 执行操作
3. 结果截图
4. 使用 `snapshot -D` 显示更改
5. 写引用截图的复现步骤

**静态 bug**（拼写错误、布局问题、缺失图片）：
1. 单张标注截图显示问题
2. 描述哪里错了

### 阶段 6: 收尾

1. **计算健康评分** 使用下面的标准
2. **写"前 3 个要修复的事项"** ——3 个最高严重程度的问题
3. **写控制台健康摘要** ——汇总所有页面看到的控制台错误
4. **更新严重程度计数** 在摘要表中
5. **填写报告元数据** ——日期、持续时间、访问页面、截图计数、框架
6. **保存基线** ——写入 `baseline.json`

---

## 健康评分标准

计算每个类别评分（0-100），然后取加权平均。

### 控制台（权重：15%）
- 0 错误 → 100
- 1-3 错误 → 70
- 4-10 错误 → 40
- 10+ 错误 → 10

### 链接（权重：10%）
- 0 损坏 → 100
- 每个损坏链接 → -15（最低 0）

### 每类别评分（视觉、功能、UX、内容、性能、可访问性）
每个类别从 100 开始。按发现扣分：
- Critical 问题 → -25
- High 问题 → -15
- Medium 问题 → -8
- Low 问题 → -3
每个类别最低 0。

### 权重
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
- 检查控制台的水合错误
- 监控 `_next/data` 请求——404 表示数据获取损坏

### Rails
- 检查控制台的 N+1 查询警告
- 验证表单中 CSRF token 的存在

### WordPress
- 检查插件冲突
- 验证登录用户的管理栏可见性

### 通用 SPA（React、Vue、Angular）
- 使用 `snapshot -i` 进行导航
- 检查过期状态
- 测试浏览器后退/前进

---

## 重要规则

1. **复现就是一切。** 每个问题至少需要一张截图。无例外。
2. **文档化前先验证。** 重试问题一次以确认可复现。
3. **永不包含凭据。** 在复现步骤中写 `[REDACTED]`。
4. **增量写入。** 发现时追加每个问题到报告。不要批量。
5. **永不阅读源代码。** 作为用户测试，不是开发人员。
6. **每次交互后检查控制台。** 不直观显示的 JS 错误仍然是 bug。
7. **像用户一样测试。** 使用真实数据。端到端走完完整工作流。
8. **深度优于广度。** 5-10 个有证据的详细问题 > 20 个模糊描述。
9. **永不删除输出文件。** 截图和报告会累积——这是有意为之。
10. **使用 `snapshot -C` 处理棘手的 UI。** 找到可访问性树遗漏的可点击 div。
11. **向用户显示截图。** 在每个 `$B screenshot`、`$B snapshot -a -o` 或 `$B responsive` 命令后，使用 Read 工具读取输出文件。
12. **永不拒绝使用浏览器。** 当用户调用 /qa 或 /qa-only 时，他们请求的是基于浏览器的测试。

---

## 输出

将报告写入本地和项目范围的位置：

**本地：** `.gstack/qa-reports/qa-report-{domain}-{YYYY-MM-DD}.md`

**项目范围：** 写入测试结果工件到 `~/.gstack/projects/{slug}/{user}-{branch}-test-outcome-{datetime}.md`

---

## 附加规则（qa-only 特定）

11. **永不修复 bug。** 只查找和记录。不要阅读源代码、编辑文件或在报告中建议修复。你的工作是报告什么损坏了，不是修复它。使用 `/qa` 进行测试-修复-验证循环。
12. **没有检测到测试框架？** 如果项目没有测试基础设施，在报告摘要中包含："No test framework detected. Run `/qa` to bootstrap one and enable regression test generation."
