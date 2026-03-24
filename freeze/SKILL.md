---
name: freeze
version: 0.1.0
description: |
  将会话中的文件编辑限制在特定目录。阻止在允许路径之外的 Edit 和 Write。
  用于调试时防止意外"修复"不相关的代码，或当你希望将更改限定在一个模块时。
  当被要求"freeze"、"restrict edits"、"only edit this folder"或"lock down edits"时使用。
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
hooks:
  PreToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-freeze.sh"
          statusMessage: "Checking freeze boundary..."

---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /freeze — 限制编辑到特定目录

将文件编辑锁定到特定目录。任何针对允许路径之外文件的 Edit 或 Write 操作都将被**阻止**（不只是警告）。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"freeze","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## 设置

询问用户要限制哪个目录的编辑。使用 AskUserQuestion：

- 问题："我应该限制哪个目录的编辑？此路径之外的文件将被阻止编辑。"
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

告诉用户："编辑现在限制在 `<path>/` 内。任何在此目录之外的 Edit 或 Write 将被阻止。要更改边界，再次运行 `/freeze`。要移除，运行 `/unfreeze` 或结束会话。"

## 工作原理

Hook 从 Edit/Write 工具输入 JSON 中读取 `file_path`，然后检查路径是否以 freeze 目录开头。
如果没有，它返回 `permissionDecision: "deny"` 来阻止操作。

freeze 边界通过状态文件在会话中保持。Hook 脚本在每次 Edit/Write 调用时读取它。

## 注意

- freeze 目录上的尾随 `/` 防止 `/src` 匹配 `/src-old`
- Freeze 仅适用于 Edit 和 Write 工具——Read、Bash、Glob、Grep 不受影响
- 这防止意外编辑，不是安全边界——像 `sed` 这样的 Bash 命令仍然可以修改边界之外的文件
- 要停用，运行 `/unfreeze` 或结束对话
