# gstack

**gstack turns Claude Code from one generic assistant into a team of specialists you can summon on demand.**

Thirteen opinionated workflow skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Plan review, design review, code review, one-command shipping, browser automation, QA testing, engineering retrospectives, and post-ship documentation — all as slash commands.

**The Completeness Principle:** Every gstack skill follows one rule — always recommend the complete implementation when AI makes the marginal cost near-zero. No more "Choose B because it's 90% of the value for half the effort" when option A is 70 more lines of code that a machine writes in seconds. We call this **boil the lake** — not boil the ocean (AGI is not here yet), but do the complete thing machines can now do, instead of the incomplete thing humans could only afford in the past. 100% test coverage, full edge case handling, complete implementations. Every recommendation shows both human-team and CC+gstack time so you can see the real cost of completeness.

### Without gstack

- The agent takes your request literally — it never asks if you're building the right thing
- It will implement exactly what you said, even when the real product is something bigger
- It recommends shortcuts to save you time — even when the complete version costs 2 minutes of machine time
- "Review my PR" gives inconsistent depth every time
- "Ship this" turns into a long back-and-forth about what to do
- The agent can write code but can't see your app — it's half blind
- You still do QA by hand: open browser, click around, check pages, squint at layouts
- Your project has no tests, and the agent never suggests setting them up

### With gstack

| Skill | Mode | What it does |
|-------|------|--------------|
| `/plan-ceo-review` | Founder / CEO | Rethink the problem. Find the 10-star product hiding inside the request. Four modes: Expansion, Selective Expansion, Hold Scope, Reduction. |
| `/plan-eng-review` | Eng manager / tech lead | Lock in architecture, data flow, diagrams, edge cases, and tests. |
| `/plan-design-review` | Senior product designer | Designer's eye audit. 80-item checklist, letter grades, AI Slop detection, DESIGN.md inference. Report only — never touches code. |
| `/design-consultation` | Design consultant | Build a complete design system from scratch. Browses competitors to get in the ballpark, proposes safe choices AND creative risks, generates realistic product mockups, and writes DESIGN.md. |
| `/review` | Paranoid staff engineer | Find the bugs that pass CI but blow up in production. Auto-fixes obvious issues. Triages Greptile review comments. Flags completeness gaps. |
| `/ship` | Release engineer | Sync main, run tests, audit coverage, resolve Greptile reviews, push, open PR. Auto-bootstraps test frameworks if your project doesn't have one. |
| `/browse` | QA engineer | Give the agent eyes. It logs in, clicks through your app, takes screenshots, catches breakage. Full QA pass in 60 seconds. |
| `/qa` | QA + fix engineer | Test app, find bugs, fix them with atomic commits, re-verify. Auto-generates regression tests for every fix. Three tiers: Quick, Standard, Exhaustive. |
| `/qa-only` | QA reporter | Report-only QA testing. Same methodology as /qa but never fixes anything. Use when you want a pure bug report without code changes. |
| `/qa-design-review` | Designer + frontend engineer | Same design audit as /plan-design-review, then fixes what it finds. Atomic `style(design):` commits, before/after screenshots, CSS-safe self-regulation. |
| `/setup-browser-cookies` | Session manager | Import cookies from your real browser (Comet, Chrome, Arc, Brave, Edge) into the headless session. Test authenticated pages without logging in manually. |
| `/retro` | Engineering manager | Team-aware retro: your deep-dive + per-person praise and growth opportunities for every contributor. Tracks test health. |
| `/document-release` | Technical writer | Update README, ARCHITECTURE, CONTRIBUTING, and project docs to match what you just shipped. |

**[Detailed skill guides with examples and philosophy](docs/skills.md)**

## Demo: one feature, five modes

```
You:   [enter plan mode, describe the feature]
       I want to add seller photo upload to the listing app. Sellers
       should be able to upload a photo of their item and we create
       the listing from it.

You:   /plan-ceo-review

Claude: "Photo upload" is not the feature. The real job is helping sellers
        create listings that actually sell. Here's what the 10-star version
        looks like: auto-identify the product from the photo, pull specs and
        pricing comps from the web, draft the title and description, suggest
        the best hero image...

You:   /plan-eng-review

Claude: [Architecture diagram, state machine for upload → classify → enrich
        → draft pipeline, async job boundaries, failure modes, test matrix]

You:   [exit plan mode, implement the plan]

You:   /review

Claude: Race condition: two tabs can overwrite cover-photo selection.
        Trust boundary: pulling web data into draft generation creates
        a prompt injection vector. Missing: orphan cleanup for failed uploads.
        [AUTO-FIXED] 2 issues. 1 needs your input.

You:   (fix the remaining issue)

You:   /ship

Claude: [Syncs main, runs tests, coverage audit, pushes branch, opens PR]
        Tests: 42 → 47 (+5 new). PR: github.com/you/app/pull/42

You:   /qa

Claude: Analyzing branch diff... 8 files changed, 3 routes affected.
        [Tests /listings/new, /listings/:id, /api/listings against localhost:3000]
        All 3 routes working. Upload + enrichment flow passes end to end.
        Regression test generated: test/qa-regression-001.test.ts

You:   /plan-design-review https://staging.myapp.com

Claude: Design Score: B  |  AI Slop Score: C
        "The site communicates competence but not confidence."
        Top issues: generic typography, AI slop patterns, flat heading scale.
        [Full report with letter grades, 12 findings, inferred design system]
        Want me to save this as your DESIGN.md?

You:   /qa-design-review

Claude: [Runs the same audit, then fixes 8 design issues]
        Design Score: B → A-  |  AI Slop Score: C → A
```

## Who this is for

You already use Claude Code heavily and want consistent, high-rigor workflows instead of one mushy generic mode. You want to tell the model what kind of brain to use right now — founder taste, engineering rigor, paranoid review, or fast execution.

This is not a prompt pack for beginners. It is an operating system for people who ship.

## How to fly: 10 sessions at once

gstack is powerful with one Claude Code session. It is transformative with ten.

[Conductor](https://conductor.build) runs multiple Claude Code sessions in parallel — each in its own isolated workspace. That means you can have one session running `/qa` on staging, another doing `/review` on a PR, a third implementing a feature, and seven more working on other branches. All at the same time.

Each workspace gets its own isolated browser instance automatically — separate Chromium process, cookies, tabs, and logs stored in `.gstack/` inside each project root. No port collisions, no shared state, no configuration needed. `/browse` and `/qa` sessions never interfere with each other, even across ten parallel workspaces.

This is the setup I use. One person, ten parallel agents, each with the right cognitive mode for its task. That is not incremental improvement. That is a different way of building software.

## Install

**Requirements:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Git](https://git-scm.com/), [Bun](https://bun.sh/) v1.0+. `/browse` compiles a native binary — works on macOS and Linux (x64 and arm64).

### Step 1: Install on your machine

Open Claude Code and paste this. Claude will do the rest.

> Install gstack: run `git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup` then add a "gstack" section to CLAUDE.md that says to use the /browse skill from gstack for all web browsing, never use mcp\_\_claude-in-chrome\_\_\* tools, and lists the available skills: /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /review, /ship, /browse, /qa, /qa-only, /qa-design-review, /setup-browser-cookies, /retro, /document-release. Then ask the user if they also want to add gstack to the current project so teammates get it.

### Step 2: Add to your repo so teammates get it (optional)

> Add gstack to this project: run `cp -Rf ~/.claude/skills/gstack .claude/skills/gstack && rm -rf .claude/skills/gstack/.git && cd .claude/skills/gstack && ./setup` then add a "gstack" section to this project's CLAUDE.md that says to use the /browse skill from gstack for all web browsing, never use mcp\_\_claude-in-chrome\_\_\* tools, lists the available skills: /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /review, /ship, /browse, /qa, /qa-only, /qa-design-review, /setup-browser-cookies, /retro, /document-release, and tells Claude that if gstack skills aren't working, run `cd .claude/skills/gstack && ./setup` to build the binary and register skills.

Real files get committed to your repo (not a submodule), so `git clone` just works. The binary and node\_modules are gitignored — teammates just need to run `cd .claude/skills/gstack && ./setup` once to build (or `/browse` handles it automatically on first use).

### What gets installed

- Skill files (Markdown prompts) in `~/.claude/skills/gstack/` (or `.claude/skills/gstack/` for project installs)
- Symlinks at `~/.claude/skills/browse`, `~/.claude/skills/qa`, `~/.claude/skills/review`, etc. pointing into the gstack directory
- Browser binary at `browse/dist/browse` (~58MB, gitignored)
- `node_modules/` (gitignored)
- `/retro` saves JSON snapshots to `.context/retros/` in your project for trend tracking

Everything lives inside `.claude/`. Nothing touches your PATH or runs in the background.

---

```
+----------------------------------------------------------------------------+
|                                                                            |
|   Are you a great software engineer who loves to write 10K LOC/day         |
|   and land 10 PRs a day like Garry?                                        |
|                                                                            |
|   Come work at YC: ycombinator.com/software                                |
|                                                                            |
|   Extremely competitive salary and equity.                                 |
|   Now hiring in San Francisco, Dogpatch District.                          |
|   Come join the revolution.                                                |
|                                                                            |
+----------------------------------------------------------------------------+
```

---

## How I use these skills

Created by [Garry Tan](https://x.com/garrytan), President & CEO of [Y Combinator](https://www.ycombinator.com/).

I built gstack because I do not want AI coding tools stuck in one mushy mode.

Planning is not review. Review is not shipping. Founder taste is not engineering rigor. If you blur all of that together, you usually get a mediocre blend of all four.

I want explicit gears.

These skills let me tell the model what kind of brain I want right now. I can switch cognitive modes on demand — founder, eng manager, paranoid reviewer, release machine. That is the unlock.

**[Read the full philosophy and per-skill deep dives](docs/skills.md)**

---

## Greptile integration

[Greptile](https://greptile.com) is a YC company that reviews your PRs automatically. gstack triages Greptile's comments as part of `/review` and `/ship` — valid issues get fixed, false positives get pushed back with evidence, already-fixed issues get auto-acknowledged.

**Setup:** Install Greptile on your GitHub repo at [greptile.com](https://greptile.com). gstack picks up its comments automatically.

**[Full Greptile integration guide](docs/greptile.md)**

---

## Contributor mode

Turn on contributor mode (`gstack-config set gstack_contributor true`) and gstack automatically files bug reports when something goes wrong — what you were doing, what broke, repro steps. Fork gstack and fix it yourself.

**[Contributor mode guide](docs/contributor-mode.md)**

---

## Troubleshooting

**Skill not showing up in Claude Code?**
Run `cd ~/.claude/skills/gstack && ./setup` (or `cd .claude/skills/gstack && ./setup` for project installs). This rebuilds symlinks so Claude can discover the skills.

**`/browse` fails or binary not found?**
Run `cd ~/.claude/skills/gstack && bun install && bun run build`. This compiles the browser binary. Requires Bun v1.0+.

**Project copy is stale?**
Run `/gstack-upgrade` — it updates both the global install and any vendored project copy automatically.

**`bun` not installed?**
Install it: `curl -fsSL https://bun.sh/install | bash`

## Upgrading

Run `/gstack-upgrade` in Claude Code. It detects your install type (global or vendored), upgrades, syncs any project copies, and shows what's new.

Or set `auto_upgrade: true` in `~/.gstack/config.yaml` to upgrade automatically whenever a new version is available.

## Uninstalling

Paste this into Claude Code:

> Uninstall gstack: remove the skill symlinks by running `for s in browse plan-ceo-review plan-eng-review plan-design-review design-consultation review ship retro qa qa-only qa-design-review setup-browser-cookies document-release; do rm -f ~/.claude/skills/$s; done` then run `rm -rf ~/.claude/skills/gstack` and remove the gstack section from CLAUDE.md. If this project also has gstack at .claude/skills/gstack, remove it by running `for s in browse plan-ceo-review plan-eng-review plan-design-review design-consultation review ship retro qa qa-only qa-design-review setup-browser-cookies document-release; do rm -f .claude/skills/$s; done && rm -rf .claude/skills/gstack` and remove the gstack section from the project CLAUDE.md too.

## Documentation

| Doc | What it covers |
|-----|---------------|
| [Skill Deep Dives](docs/skills.md) | Philosophy, examples, and workflow for every skill |
| [Greptile Integration](docs/greptile.md) | Setup and triage workflow |
| [Contributor Mode](docs/contributor-mode.md) | How to help improve gstack |
| [Browser Reference](BROWSER.md) | Full command reference for `/browse` |
| [Architecture](ARCHITECTURE.md) | Design decisions and system internals |
| [Contributing](CONTRIBUTING.md) | Dev setup, testing, and dev mode |
| [Changelog](CHANGELOG.md) | What's new in every version |

### Testing

```bash
bun test                     # free static tests (<5s)
EVALS=1 bun run test:evals   # full E2E + LLM evals (~$4, ~20min)
bun run eval:watch            # live dashboard during E2E runs
```

E2E tests stream real-time progress, write machine-readable diagnostics, and persist partial results that survive kills. See CONTRIBUTING.md for the full eval infrastructure.

## License

MIT
