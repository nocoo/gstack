---
name: guard
version: 0.1.0
description: |
  完全安全模式：危险命令警告 + 目录范围的编辑限制。
  结合 /careful（在 rm -rf、DROP TABLE、force-push 等之前警告）
  和 /freeze（阻止在指定目录之外编辑）。用于接触生产环境或调试在线系统时的最大安全性。
  当被要求"guard mode"、"full safety"、"lock it down"或"maximum safety"时使用。
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../careful/bin/check-careful.sh"
          statusMessage: "Checking for destructive commands..."
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."

---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /guard — 完全安全模式

同时激活危险命令警告和目录范围的编辑限制。
这是 `/careful` + `/freeze` 在单个命令中的组合。

**依赖说明：** 此 skill 引用同级 `/careful` 和 `/freeze` skill 目录中的 hook 脚本。
两者都必须安装（gstack 设置脚本会一起安装它们）。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"guard","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## 设置

询问用户要限制哪个目录的编辑。使用 AskUserQuestion：

- 问题："守护模式：编辑应该限制在哪个目录？危险命令警告始终开启。所选路径之外的文件将被阻止编辑。"
- 文本输入（非多选）——用户输入路径。

用户提供了目录路径后：

1. 解析为绝对路径：
```bash
FREEZE_DIR=$(cd "<user-provided-path>" 2>/dev/null && pwd)
echo "$FREEZE_DIR"
```

2. 确保有尾随斜杠并保存到 freeze 状态文件：
```bash
FREEZE_DIR="${FREEZE_DIR%/}/"
STATE_DIR="${CLAUDE_PLUGIN_DATA:-$HOME/.gstack}"
mkdir -p "$STATE_DIR"
echo "$FREEZE_DIR" > "$STATE_DIR/freeze-dir.txt"
echo "Freeze boundary set: $FREEZE_DIR"
```

告诉用户：
- "**守护模式已激活。** 现在运行两项保护："
- "1. **危险命令警告** — rm -rf、DROP TABLE、force-push 等将在执行前警告（你可以覆盖）"
- "2. **编辑边界** — 文件编辑限制在 `<path>/` 内。此目录之外的编辑被阻止。"
- "要移除编辑边界，运行 `/unfreeze`。要停用所有内容，结束会话。"

## 受保护的内容

参见 `/careful` 了解危险命令模式和安全例外的完整列表。
参见 `/freeze` 了解编辑边界强制执行的工作原理。
