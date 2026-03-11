---
name: gstack-browse
version: 2.0.0
description: |
  Fast web browsing via persistent headless Chromium daemon. Navigate to any URL,
  read page content, click elements, fill forms, run JavaScript, take screenshots,
  inspect CSS/DOM, capture console/network logs, and more. ~100ms per command after
  first call. Use when you need to check a website, verify a deployment, read docs,
  or interact with any web page. No MCP, no Chrome extension — just fast CLI.
allowed-tools:
  - Bash
  - Read
---

# Browse v2: Persistent Browser for Claude Code

Persistent headless Chromium daemon. First call auto-starts the server (~3s).
Every subsequent call: ~100-200ms. Auto-shuts down after 30 min idle.

## IMPORTANT

- Use `~/.claude/skills/gstack-browse/dist/browse` (compiled binary) or `bun run ~/.claude/skills/gstack-browse/src/cli.ts` via Bash.
- NEVER use `mcp__claude-in-chrome__*` tools. They are slow and unreliable.
- The browser persists between calls — cookies, tabs, and state carry over.
- The server auto-starts on first command. No setup needed.

## Quick Reference

```bash
B=~/.claude/skills/gstack-browse/dist/browse

# Navigate to a page
$B goto https://example.com

# Read cleaned page text
$B text

# Take a screenshot (then Read the image)
$B screenshot /tmp/page.png

# Run JavaScript
$B js "document.title"

# Get all links
$B links

# Click something
$B click "button.submit"

# Fill a form
$B fill "#email" "test@test.com"
$B fill "#password" "abc123"
$B click "button[type=submit]"

# Get HTML of an element
$B html "main"

# Get computed CSS
$B css "body" "font-family"

# Get element attributes
$B attrs "nav"

# Wait for element to appear
$B wait ".loaded"

# Accessibility tree
$B accessibility

# Set viewport
$B viewport 375x812

# Set cookies / headers
$B cookie "session=abc123"
$B header "Authorization:Bearer token123"
```

## Command Reference

### Navigation
```
browse goto <url>         Navigate current tab
browse back               Go back
browse forward            Go forward
browse reload             Reload page
browse url                Print current URL
```

### Content extraction
```
browse text               Cleaned page text (no scripts/styles)
browse html [selector]    innerHTML of element, or full page HTML
browse links              All links as "text → href"
browse forms              All forms + fields as JSON
browse accessibility      Accessibility tree snapshot (ARIA)
```

### Interaction
```
browse click <selector>        Click element
browse fill <selector> <value> Fill input field
browse select <selector> <val> Select dropdown value
browse hover <selector>        Hover over element
browse type <text>             Type into focused element
browse press <key>             Press key (Enter, Tab, Escape, etc.)
browse scroll [selector]       Scroll element into view, or page bottom
browse wait <selector>         Wait for element to appear (max 10s)
browse viewport <WxH>          Set viewport size (e.g. 375x812)
```

### Inspection
```
browse js <expression>         Run JS, print result
browse eval <js-file>          Run JS file against page
browse css <selector> <prop>   Get computed CSS property
browse attrs <selector>        Get element attributes as JSON
browse console                 Dump captured console messages
browse console --clear         Clear console buffer
browse network                 Dump captured network requests
browse network --clear         Clear network buffer
browse cookies                 Dump all cookies as JSON
browse storage                 localStorage + sessionStorage as JSON
browse storage set <key> <val> Set localStorage value
browse perf                    Page load performance timings
```

### Visual
```
browse screenshot [path]       Screenshot (default: /tmp/browse-screenshot.png)
browse pdf [path]              Save as PDF
browse responsive [prefix]     Screenshots at mobile/tablet/desktop
```

### Compare
```
browse diff <url1> <url2>      Text diff between two pages
```

### Multi-step (chain)
```
echo '[["goto","https://example.com"],["fill","#email","test@test.com"],["click","#submit"],["screenshot","/tmp/result.png"]]' | browse chain
```

### Tabs
```
browse tabs                    List tabs (id, url, title)
browse tab <id>                Switch to tab
browse newtab [url]            Open new tab
browse closetab [id]           Close tab
```

### Server management
```
browse status                  Server health, uptime, tab count
browse stop                    Shutdown server
browse restart                 Kill + restart server
```

## Speed Rules

1. **Navigate once, query many times.** `goto` loads the page; then `text`, `js`, `css`, `screenshot` all run against the loaded page instantly.
2. **Use `js` for precision.** `js "document.querySelector('.price').textContent"` is faster than parsing full page text.
3. **Use `links` to survey.** Faster than `text` when you just need navigation structure.
4. **Use `chain` for multi-step flows.** Avoids CLI overhead per step.
5. **Use `responsive` for layout checks.** One command = 3 viewport screenshots.

## When to Use What

| Task | Commands |
|------|----------|
| Read a page | `goto <url>` then `text` |
| Check if element exists | `js "!!document.querySelector('.thing')"` |
| Extract specific data | `js "document.querySelector('.price').textContent"` |
| Visual check | `screenshot /tmp/x.png` then Read the image |
| Fill and submit form | `fill "#email" "val"` → `click "#submit"` → `screenshot` |
| Check CSS | `css "selector" "property"` |
| Inspect DOM | `html "selector"` or `attrs "selector"` |
| Debug console errors | `console` |
| Check network requests | `network` |
| Check local dev | `goto http://127.0.0.1:3000` |
| Check staging | `goto https://staging.garryslist.org` |
| Check production | `goto https://garryslist.org` |
| Compare two pages | `diff <url1> <url2>` |
| Mobile layout check | `responsive /tmp/prefix` |
| Multi-step flow | `echo '[...]' \| browse chain` |

## Multi-step Workflow Example

```bash
B=~/.claude/skills/gstack-browse/dist/browse

# Login flow
$B goto https://example.com/login
$B fill "#email" "user@test.com"
$B fill "#password" "pass123"
$B click "button[type=submit]"
$B wait ".dashboard"
$B screenshot /tmp/dashboard.png

# Or as a chain (single call):
echo '[
  ["goto", "https://example.com/login"],
  ["fill", "#email", "user@test.com"],
  ["fill", "#password", "pass123"],
  ["click", "button[type=submit]"],
  ["wait", ".dashboard"],
  ["screenshot", "/tmp/dashboard.png"]
]' | $B chain
```

## Architecture

- Persistent Chromium daemon on localhost (port 9400-9410)
- Bearer token auth per session
- State file: `/tmp/browse-server.json`
- Console log: `/tmp/browse-console.log`
- Network log: `/tmp/browse-network.log`
- Auto-shutdown after 30 min idle
- Chromium crash → server exits → auto-restarts on next command
