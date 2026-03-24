---
name: gstack-upgrade
version: 1.1.0
description: |
  升级 gstack 到最新版本。检测全局 vs vendored 安装，
  运行升级，显示新功能。当被要求"upgrade gstack"、"update gstack"或"get latest version"时使用。
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion

---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /gstack-upgrade

升级 gstack 到最新版本并显示新功能。

## 内联升级流程

此部分被所有 skill 前置准备在检测到 `UPGRADE_AVAILABLE` 时引用。

### Step 1: 询问用户（或自动升级）

首先，检查是否启用了自动升级：
```bash
_AUTO=""
[ "${GSTACK_AUTO_UPGRADE:-}" = "1" ] && _AUTO="true"
[ -z "$_AUTO" ] && _AUTO=$(~/.claude/skills/gstack/bin/gstack-config get auto_upgrade 2>/dev/null || true)
echo "AUTO_UPGRADE=$_AUTO"
```

**如果 `AUTO_UPGRADE=true` 或 `AUTO_UPGRADE=1`：** 跳过 AskUserQuestion。记录 "Auto-upgrading gstack v{old} → v{new}..." 并直接进入 Step 2。如果 `./setup` 在自动升级期间失败，从备份（`.bak` 目录）恢复并警告用户："Auto-upgrade failed — restored previous version. Run `/gstack-upgrade` manually to retry."

**否则**，使用 AskUserQuestion：
- 问题: "gstack **v{new}** 可用（你当前是 v{old}）。现在升级吗？"
- 选项: ["Yes, upgrade now", "Always keep me up to date", "Not now", "Never ask again"]

**如果 "Yes, upgrade now":** 进入 Step 2。

**如果 "Always keep me up to date":** 运行：
```bash
~/.claude/skills/gstack/bin/gstack-config set auto_upgrade true
```
然后进入 Step 2。

**如果 "Not now":** 写入暂存状态：
```bash
mkdir -p ~/.gstack
echo "$(date +%s)" > ~/.gstack/upgrade-snooze
```
告诉用户："Will remind you later. Run `/gstack-upgrade` when you."

**如果 "Never ask again":** 运行：
```bash
~/.claude/skills/gstack/bin/gstack-config set update_check false
```
告诉用户："Update checks disabled. Run `~/.claude/skills/gstack/bin/gstack-config set auto_upgrade false` to re-enable."

### Step 2: 检测安装类型

```bash
if [ -d "$HOME/.claude/skills/gstack/.git" ]; then
  install_type="global-git"
  install_dir="$HOME/.claude/skills/gstack"
else
  install_type="vendored"
  # 查找项目根目录中的 .claude/skills/gstack
  for dir in .claude/skills/gstack ~/workspace/*/gstack; do
    if [ -d "$dir" ]; then
      install_dir=$(dirname "$dir" | xargs basename)
      break
    fi
  done
  install_dir=${install_dir:-unknown}
fi
echo "INSTALL_TYPE=$install_type INSTALL_DIR=$install_dir"
```

### Step 3: 运行升级

对于 vendored 安装： 进入包含 gstack 的仓库根目录，拉取最新更改并运行 `./setup`。

对于全局安装： 在临时目录中克隆仓库，然后运行 `./setup`。

**设置临时目录：**
```bash
tmpdir=$(mktemp -d)
git clone --depth 1 https://github.com/garrytan/gstack "$tmpdir/gstack"
cd "$tmpdir/gstack"
```

**运行 setup:**
```bash
./setup
```

### Step 4: 检查是否成功

```bash
old_version=$(cat "$install_dir/VERSION" 2>/dev/null || echo "unknown")
new_version=$(cat VERSION 2>/dev/null || echo "unknown")
echo "old=$old_version new=$new_version"
```

**如果 `$stash_output` 包含 `saved working directory` 或没有错误:**
升级成功！

**否则:**
升级失败。恢复备份并显示错误。

### Step 5: 清理

```bash
cd /
rm -rf "$tmpdir"
```

### Step 6: 显示新功能

读取 `$install_dir/changelog.md`。找到所有版本条目。将每个版本之间的内容分组。跳过内部 refactors 除非用户请求完整日志。使用 5-7 个 bullets 作为主要变化的摘要。

格式：
```
gstack v{new} — upgraded from v{old}!

What's new:
- [bullet 1]
- [bullet 2]
- ...

Happy shipping!
```

## 独立使用

当直接调用 `/gstack-upgrade` 时（不是从前置准备中）：

1. 强制更新检查（绕过缓存）：
```bash
~/.claude/skills/gstack/bin/gstack-update-check --force 2>/dev/null || \
.claude/skills/gstack/bin/gstack-update-check --force 2>/dev/null || true
```

2. 如果有可用升级： 按照步骤 2-6 执行。

3. 如果没有输出（已是最新版本）： 告诉用户 "gstack is already up to date (v{version})."

## 依赖说明

此 skill 依赖 `git` 啽令。如果 git 不可用，将无法升级 vendored 安装。
