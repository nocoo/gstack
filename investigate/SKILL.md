---
name: investigate
version: 1.0.0
description: |
  系统化调试与根因调查。四个阶段：调查、分析、假设、实施。
  铁律：没有根因就不修复。
  用于"debug this"、"fix this bug"、"why is this broken"、"investigate this error"或"root cause analysis"。
  当用户报告错误、意外行为或排查某事为何停止工作时主动建议。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion
  - WebSearch
hooks:
  PreToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Checking debug scope boundary..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Checking debug scope boundary..."

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
  --skill "investigate" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

## 计划状态页脚

在计划模式中调用 ExitPlanMode 前运行 `gstack-review-read` 并写入审查报告。

# 系统化调试

## 铁律

**没有根因调查就不能修复。**

修复症状会制造打地鼠式调试。每个不解决根因的修复都会让下一个 bug 更难找到。找到根因，然后修复它。

---

## 阶段 1: 根因调查

在形成任何假设之前收集上下文。

1. **收集症状：** 阅读错误消息、堆栈跟踪和复现步骤。如果用户没有提供足够的上下文，通过 AskUserQuestion 一次问一个问题。

2. **阅读代码：** 从症状追溯到潜在原因。使用 Grep 查找所有引用，使用 Read 理解逻辑。

3. **检查最近更改：**
   ```bash
   git log --oneline -20 -- <affected-files>
   ```
   这之前能工作吗？什么变了？回归意味着根因在 diff 中。

4. **复现：** 你能确定性地触发 bug 吗？如果不能，在继续之前收集更多证据。

输出：**"根因假设：..."** — 关于什么错了以及为什么的具体、可测试的声明。

---

## 范围锁定

形成根因假设后，将编辑锁定到受影响的模块以防止范围蔓延。

```bash
[ -x "${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh" ] && echo "FREEZE_AVAILABLE" || echo "FREEZE_UNAVAILABLE"
```

**如果 FREEZE_AVAILABLE：** 识别包含受影响文件的最窄目录。写入 freeze 状态文件。

告诉用户："编辑限制在 `<dir>/` 内进行此调试会话。这防止更改不相关的代码。运行 `/unfreeze` 移除限制。"

**如果 FREEZE_UNAVAILABLE：** 跳过范围锁定。编辑不受限制。

---

## 阶段 2: 模式分析

检查此 bug 是否匹配已知模式：

| 模式 | 特征 | 哪里查找 |
|---------|-----------|---------------|
| 竞争条件 | 间歇性、依赖时序 | 共享状态的并发访问 |
| Nil/null 传播 | NoMethodError、TypeError | 可选值上缺少保护 |
| 状态损坏 | 数据不一致、部分更新 | 事务、回调、钩子 |
| 集成失败 | 超时、意外响应 | 外部 API 调用、服务边界 |
| 配置漂移 | 本地工作、staging/prod 失败 | 环境变量、功能标志、DB 状态 |
| 缓存过期 | 显示旧数据、清除缓存后修复 | Redis、CDN、浏览器缓存、Turbo |

还要检查：
- `TODOS.md` 了解相关的已知问题
- `git log` 查看同一区域的先前修复——**同一文件中的重复 bug 是架构异味**，不是巧合

**外部模式搜索：** 如果 bug 不匹配上面的已知模式，WebSearch：
- "{framework} {generic error type}"——**先清理：** 去除主机名、IP、文件路径、SQL、客户数据。搜索错误类别，不是原始消息。

---

## 阶段 3: 假设测试

在写任何修复之前，验证你的假设。

1. **确认假设：** 在怀疑的根因处添加临时日志语句、断言或调试输出。运行复现。证据匹配吗？

2. **如果假设错误：** 在形成下一个假设之前，考虑搜索错误。**先清理**——从错误消息中去除主机名、IP、文件路径、SQL 片段、客户标识符和任何内部/专有数据。只搜索通用错误类型和框架上下文。然后返回阶段 1。收集更多证据。不要猜测。

3. **三振出局规则：** 如果 3 个假设失败，**停止**。使用 AskUserQuestion：
   ```
   3 个假设已测试，都不匹配。这可能是架构问题
   而不是简单的 bug。
   
   A) 继续调查——我有一个新假设：[描述]
   B) 升级人工审查——这需要了解系统的人
   C) 添加日志并等待——在该区域埋点，下次捕获
   ```

**危险信号**——如果看到任何这些，慢下来：
- "暂时快速修复"——没有"暂时"。要么正确修复，要么升级。
- 在追踪数据流之前提出修复——你在猜测。
- 每个修复都在其他地方暴露新问题——错误的层，不是错误的代码。

---

## 阶段 4: 实施

一旦根因确认：

1. **修复根因，不是症状。** 消除实际问题的最小更改。

2. **最小 diff：** 触及最少的文件，更改最少的行。抵制重构相邻代码的冲动。

3. **写回归测试**：
   - **失败**没有修复（证明测试有意义）
   - **通过**有修复（证明修复有效）

4. **运行完整测试套件。** 粘贴输出。不允许回归。

5. **如果修复触及 >5 个文件：** 使用 AskUserQuestion 标记爆炸半径：
   ```
   此修复触及 N 个文件。对于 bug 修复来说爆炸半径很大。
   A) 继续——根因确实跨越这些文件
   B) 拆分——现在修复关键路径，推迟其余
   C) 重新思考——也许有更有针对性的方法
   ```

---

## 阶段 5: 验证与报告

**全新验证：** 复现原始 bug 场景并确认已修复。这不是可选的。

运行测试套件并粘贴输出。

输出结构化调试报告：
```
调试报告
════════════════════════════════════════
症状:         [用户观察到的]
根因:      [实际错误的]
修复:             [更改了什么，含 file:line 引用]
证据:        [测试输出、显示修复有效的复现尝试]
回归测试: [file:line 新测试]
相关:         [TODOS.md 项目、同一区域的先前 bug、架构说明]
状态:          DONE | DONE_WITH_CONCERNS | BLOCKED
════════════════════════════════════════
```

---

## 重要规则

- **3+ 次失败修复尝试 → 停止并质疑架构。** 错误的架构，不是失败的假设。
- **永不应用你无法验证的修复。** 如果无法复现和确认，不要发布。
- **永不说"这应该能修复它"。** 验证并证明它。运行测试。
- **如果修复触及 >5 个文件 → AskUserQuestion** 关于爆炸半径再继续。
- **完成状态：**
  - DONE — 找到根因、应用修复、写回归测试、所有测试通过
  - DONE_WITH_CONCERNS — 修复但无法完全验证（如间歇性 bug、需要 staging）
  - BLOCKED — 调查后根因不清楚，已升级
