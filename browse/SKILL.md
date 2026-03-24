---
name: browse
version: 1.1.0
description: |
  用于 QA 测试和站点自测的快速无头浏览器。导航任意 URL、交互元素、
  验证页面状态、对比操作前后差异、拍摄带标注截图、检查响应式布局、
  测试表单和上传、处理对话框并断言元素状态。
  每条命令约 100ms。当需要测试功能、验证部署、体验用户流程，
  或用证据提交 bug 时使用。当被要求"在浏览器中打开"、"测试站点"、
  "截图"或"体验这个"时使用。
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion

---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## 前置准备（优先执行）

```bash
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || .claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.gstack/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(~/.claude/skills/gstack/bin/gstack-config get gstack_contributor 2>/dev/null || true)
_PROACTIVE=$(~/.claude/skills/gstack/bin/gstack-config get proactive 2>/dev/null || echo "true")
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "PROACTIVE: $_PROACTIVE"
source <(~/.claude/skills/gstack/bin/gstack-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"
_LAKE_SEEN=$([ -f ~/.gstack/.completeness-intro-seen ] && echo "yes" || echo "no")
echo "LAKE_INTRO: $_LAKE_SEEN"
_TEL=$(~/.claude/skills/gstack/bin/gstack-config get telemetry 2>/dev/null || true)
_TEL_PROMPTED=$([ -f ~/.gstack/.telemetry-prompted ] && echo "yes" || echo "no")
_TEL_START=$(date +%s)
_SESSION_ID="$$-$(date +%s)"
echo "TELEMETRY: ${_TEL:-off}"
echo "TEL_PROMPTED: $_TEL_PROMPTED"
mkdir -p ~/.gstack/analytics
echo '{"skill":"browse","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

如果 `PROACTIVE` 为 `"false"`，不要主动建议 gstack skill。
如果输出显示 `UPGRADE_AVAILABLE`：阅读 gstack-upgrade/SKILL.md 并遵循升级流程。
如果 `LAKE_INTRO` 为 `no`：介绍完整性原则。
如果 `TEL_PROMPTED` 为 `no`：询问遥测数据。

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

在构建基础设施之前——**先搜索。** 阅读 `~/.claude/skills/gstack/ETHOS.md`。

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
  --skill "browse" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

## 计划状态页脚

在计划模式中调用 ExitPlanMode 前运行 `gstack-review-read` 并写入审查报告。

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

如果显示 `NEEDS_SETUP`：
1. 告诉用户："gstack browse 需要一次性构建（约 10 秒）。可以继续吗？"然后停止并等待。
2. 运行：`cd <SKILL_DIR> && ./setup`
3. 如果没有安装 `bun`：`curl -fsSL https://bun.sh/install | bash`

# browse: QA 测试与自测

持久化无头 Chromium。首次调用自动启动（约 3 秒），之后每条命令约 100ms。
状态在调用之间保持（cookies、标签页、登录会话）。

## 核心 QA 模式

### 1. 验证页面正确加载
```bash
$B goto https://yourapp.com
$B text                          # 内容加载了？
$B console                       # JS 错误？
$B network                       # 失败的请求？
$B is visible ".main-content"    # 关键元素存在？
```

### 2. 测试用户流程
```bash
$B goto https://app.com/login
$B snapshot -i                   # 查看所有交互元素
$B fill @e3 "user@test.com"
$B fill @e4 "password"
$B click @e5                     # 提交
$B snapshot -D                   # diff: 提交后哪些变了？
$B is visible ".dashboard"       # 成功状态存在？
```

### 3. 验证操作成功
```bash
$B snapshot                      # 基线
$B click @e3                     # 做某事
$B snapshot -D                   # 统一 diff 显示具体变化
```

### 4. Bug 报告的视觉证据
```bash
$B snapshot -i -a -o /tmp/annotated.png   # 带标签截图
$B screenshot /tmp/bug.png                # 普通截图
$B console                                # 错误日志
```

### 5. 找到所有可点击元素（包括非 ARIA）
```bash
$B snapshot -C                   # 找到 cursor:pointer、onclick、tabindex 的 div
$B click @c1                     # 与它们交互
```

### 6. 断言元素状态
```bash
$B is visible ".modal"
$B is enabled "#submit-btn"
$B is disabled "#submit-btn"
$B is checked "#agree-checkbox"
$B is editable "#name-field"
$B is focused "#search-input"
$B js "document.body.textContent.includes('Success')"
```

### 7. 测试响应式布局
```bash
$B responsive /tmp/layout        # 移动 + 平板 + 桌面截图
$B viewport 375x812            # 或设置特定视口
$B screenshot /tmp/mobile.png
```

### 8. 测试文件上传
```bash
$B upload "#file-input" /path/to/file.pdf
$B is visible ".upload-success"
```

### 9. 测试对话框
```bash
$B dialog-accept "yes"           # 设置处理器
$B click "#delete-button"          # 触发对话框
$B dialog                        # 查看出现了什么
$B snapshot -D                    # 验证删除发生了
```

### 10. 对比环境
```bash
$B diff https://staging.app.com https://prod.app.com
```

### 11. 向用户展示截图
在 `$B screenshot`、`$B snapshot -a -o` 或 `$B responsive` 之后，始终使用 Read 工具读取输出 PNG，以便用户能看到。

## 用户交接

当遇到无头模式无法处理的事情时（CAPTCHA、复杂认证、多因素登录），交接给用户：

```bash
# 1. 在当前页面打开可见 Chrome
$B handoff "Stuck on CAPTCHA at login page"

# 2. 通过 AskUserQuestion 告诉用户发生了什么
#    "我已在登录页面打开 Chrome。请解决 CAPTCHA，
#     完成后告诉我。"

# 3. 用户说"完成"后，重新快照并继续
$B resume
```

**何时使用交接：**
- CAPTCHA 或机器人检测
- 多因素认证（短信、认证器 app）
- 需要用户交互的 OAuth 流程
- 3 次尝试后 AI 仍无法处理的复杂交互

浏览器在交接期间保持所有状态（cookies、localStorage、标签页）。
`resume` 后，你获得用户离开位置的全新快照。

## 快照标志

快照是你理解和交互页面的主要工具。

```
-i        --interactive           仅交互元素（按钮、链接、输入框），带 @e refs
-c        --compact               紧凑（无空结构节点）
-d <N>    --depth                 限制树深度（0 = 仅根，默认：无限）
-s <sel>  --selector              作用域限定为 CSS 选择器
-D        --diff                  与前一个快照的统一 diff（首次调用存储基线）
-a        --annotate              带红色覆盖框和 ref 标签的标注截图
-o <path> --output                标注截图输出路径（默认：<temp>/browse-annotated.png）
-C        --cursor-interactive    光标可交互元素（@c refs）
```

所有标志可以自由组合。示例：`$B snapshot -i -a -C -o /tmp/annotated.png`

**Ref 编号：** @e refs 按树顺序分配。@c refs 单独编号。

## 完整命令列表

### 导航
| 命令 | 描述 |
|---------|-------------|
| `back` | 历史后退 |
| `forward` | 历史前进 |
| `goto <url>` | 导航到 URL |
| `reload` | 重新加载页面 |
| `url` | 打印当前 URL |

### 读取
| 命令 | 描述 |
|---------|-------------|
| `accessibility` | 完整 ARIA 树 |
| `forms` | 表单字段（JSON） |
| `html [selector]` | 选择器的 innerHTML |
| `links` | 所有链接 |
| `text` | 清理后的页面文本 |

### 交互
| 命令 | 描述 |
|---------|-------------|
| `click <sel>` | 点击元素 |
| `cookie <name>=<value>` | 设置 cookie |
| `cookie-import <json>` | 从 JSON 导入 cookies |
| `cookie-import-browser [browser] [--domain d]` | 从浏览器导入 cookies |
| `dialog-accept [text]` | 自动接受下一个对话框 |
| `dialog-dismiss` | 自动关闭下一个对话框 |
| `fill <sel> <val>` | 填写输入框 |
| `header <name>:<value>` | 设置自定义请求头 |
| `hover <sel>` | 悬停 |
| `press <key>` | 按键 |
| `scroll [sel]` | 滚动元素到视口 |
| `select <sel> <val>` | 选择下拉选项 |
| `type <text>` | 输入到焦点元素 |
| `upload <sel> <file>` | 上传文件 |
| `useragent <string>` | 设置 user agent |
| `viewport <WxH>` | 设置视口大小 |
| `wait <sel|--networkidle|--load>` | 等待 |

### 检查
| 命令 | 描述 |
|---------|-------------|
| `attrs <sel|@ref>` | 元素属性 |
| `console [--clear|--errors]` | 控制台消息 |
| `cookies` | 所有 cookies |
| `css <sel> <prop>` | CSS 值 |
| `dialog [--clear]` | 对话框消息 |
| `eval <file>` | 运行 JavaScript 文件 |
| `is <prop> <sel>` | 状态检查 |
| `js <expr>` | 运行 JavaScript 表达式 |
| `network [--clear]` | 网络请求 |
| `perf` | 页面加载计时 |
| `storage [set k v]` | localStorage/sessionStorage |

### 视觉
| 命令 | 描述 |
|---------|-------------|
| `diff <url1> <url2>` | 两页面文本 diff |
| `pdf [path]` | 保存为 PDF |
| `responsive [prefix]` | 多视口截图 |
| `screenshot [...]` | 截图 |

### 快照
| 命令 | 描述 |
|---------|-------------|
| `snapshot [flags]` | 带 @refs 的可访问性树 |

### Meta
| 命令 | 描述 |
|---------|-------------|
| `chain` | 从 JSON stdin 运行命令 |

### 标签页
| 命令 | 描述 |
|---------|-------------|
| `closetab [id]` | 关闭标签页 |
| `newtab [url]` | 打开新标签页 |
| `tab <id>` | 切换标签页 |
| `tabs` | 列出打开的标签页 |

### 服务器
| 命令 | 描述 |
|---------|-------------|
| `handoff [message]` | 打开可见 Chrome 供用户接管 |
| `restart` | 重启服务器 |
| `resume` | 用户接管后重新快照 |
| `status` | 健康检查 |
| `stop` | 关闭服务器 |
