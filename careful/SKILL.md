---
name: careful
version: 0.1.0
description: |
  危险命令的安全护栏。在 rm -rf、DROP TABLE、force-push、git reset --hard、
  kubectl delete 等危险操作之前发出警告。用户可以覆盖每个警告。
  用于接触生产环境、调试在线系统或在共享环境中工作。
  当被要求"小心"、"安全模式"、"prod 模式"或"careful 模式"时使用。
allowed-tools:
  - Bash
  - Read
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-careful.sh"
          statusMessage: "Checking for destructive commands..."

---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /careful — 危险命令护栏

安全模式现已**激活**。每个 bash 命令在运行前都会检查危险模式。
如果检测到危险命令，将发出警告，你可以选择继续或取消。

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"careful","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## 受保护的内容

| 模式 | 示例 | 风险 |
|---------|---------|------|
| `rm -rf` / `rm -r` / `rm --recursive` | `rm -rf /var/data` | 递归删除 |
| `DROP TABLE` / `DROP DATABASE` | `DROP TABLE users;` | 数据丢失 |
| `TRUNCATE` | `TRUNCATE orders;` | 数据丢失 |
| `git push --force` / `-f` | `git push -f origin main` | 历史重写 |
| `git reset --hard` | `git reset --hard HEAD~3` | 未提交工作丢失 |
| `git checkout .` / `git restore .` | `git checkout .` | 未提交工作丢失 |
| `kubectl delete` | `kubectl delete pod` | 生产环境受影响 |
| `docker rm -f` / `docker system prune` | `docker system prune -a` | 容器/镜像丢失 |

## 安全例外

以下模式无需警告：
- `rm -rf node_modules` / `.next` / `dist` / `__pycache__` / `.cache` / `build` / `.turbo` / `coverage`

## 工作原理

Hook 从工具输入 JSON 中读取命令，根据上述模式检查它，
如果发现匹配则返回 `permissionDecision: "ask"` 并附带警告消息。
你始终可以覆盖警告并继续。

要停用，请结束对话或开始新对话。Hook 是会话范围的。
