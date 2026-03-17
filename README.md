# gstack

Hi, I'm [Garry Tan](https://x.com/garrytan). I'm President & CEO of [Y Combinator](https://www.ycombinator.com/), where I've worked with thousands of startups. Before YC, I designed the Palantir logo and was one of the first eng manager/PM/designers there. I cofounded Posterous, a blog platform we sold to Twitter. I built Bookface, YC's internal social network, back in 2013. I've been building products for a long time.

And right now I am in the middle of something that feels like a new era entirely.

In the last 60 days I have written **over 600,000 lines of production code** — 35% tests — and I am doing **10,000 to 20,000 usable lines of code per day** as a part-time part of my day while doing all my duties as CEO of YC. That is not a typo. My last `/retro` across 3 projects: **140,751 lines added, 362 commits, ~115k net LOC** in one period. The models are getting dramatically better every week. We are at the dawn of something real — one person shipping at a scale that used to require a team of twenty.

**2026 — 1,237 contributions and counting:**

![GitHub contributions 2026 — 1,237 contributions, massive acceleration in Jan-Mar](docs/images/github-2026.png)

**2013 — when I built Bookface at YC (772 contributions):**

![GitHub contributions 2013 — 772 contributions building Bookface at YC](docs/images/github-2013.png)

Same person. Different era. The difference is the tooling.

**gstack is how I do it.** It is my open source software factory. It turns Claude Code into a virtual engineering team you actually manage — a CEO who rethinks the product, an eng manager who locks the architecture, a designer who catches AI slop, a paranoid reviewer who finds production bugs, a QA lead who opens a real browser and clicks through your app, and a release engineer who ships the PR. Thirteen specialists, all as slash commands, all Markdown, **all free, MIT license, available right now.**

I am learning how to get to the edge of what agentic systems can do as of March 2026, and this is my live experiment. I am sharing it because I want the whole world on this journey with me.

Fork it. Improve it. Make it yours. Don't player hate, appreciate.

---

## The team

| Skill | Your specialist | What they do |
|-------|----------------|--------------|
| `/plan-ceo-review` | **CEO / Founder** | Rethink the problem. Find the 10-star product hiding inside the request. Four modes: Expansion, Selective Expansion, Hold Scope, Reduction. |
| `/plan-eng-review` | **Eng Manager** | Lock in architecture, data flow, diagrams, edge cases, and tests. Forces hidden assumptions into the open. |
| `/plan-design-review` | **Senior Designer** | 80-item design audit with letter grades. AI Slop detection. Infers your design system. Report only — never touches code. |
| `/design-consultation` | **Design Partner** | Build a complete design system from scratch. Knows the landscape, proposes creative risks, generates realistic product mockups. Design at the heart of all other phases. |
| `/review` | **Staff Engineer** | Find the bugs that pass CI but blow up in production. Auto-fixes the obvious ones. Flags completeness gaps. |
| `/ship` | **Release Engineer** | Sync main, run tests, audit coverage, push, open PR. Bootstraps test frameworks if you don't have one. One command. |
| `/browse` | **QA Engineer** | Give the agent eyes. Real Chromium browser, real clicks, real screenshots. ~100ms per command. |
| `/qa` | **QA Lead** | Test your app, find bugs, fix them with atomic commits, re-verify. Auto-generates regression tests for every fix. |
| `/qa-only` | **QA Reporter** | Same methodology as /qa but report only. Use when you want a pure bug report without code changes. |
| `/qa-design-review` | **Designer Who Codes** | Same audit as /plan-design-review, then fixes what it finds. Atomic commits, before/after screenshots. |
| `/setup-browser-cookies` | **Session Manager** | Import cookies from your real browser (Chrome, Arc, Brave, Edge) into the headless session. Test authenticated pages. |
| `/retro` | **Eng Manager** | Team-aware weekly retro. Per-person breakdowns, shipping streaks, test health trends, growth opportunities. |
| `/document-release` | **Technical Writer** | Update all project docs to match what you just shipped. Catches stale READMEs automatically. |

**[Deep dives with examples and philosophy for every skill →](docs/skills.md)**

---

## What's new and why it matters

**Design is at the heart.** `/design-consultation` doesn't just pick fonts. It researches what's out there in your space, proposes safe choices AND creative risks, generates realistic mockups of your actual product, and writes `DESIGN.md` — and then `/qa-design-review` and `/plan-eng-review` read what you chose. Design decisions flow through the whole system.

**`/qa` was a massive unlock.** It let me go from 6 to 12 parallel workers. Claude Code saying *"I SEE THE ISSUE"* and then actually fixing it, generating a regression test, and verifying the fix — that changed how I work. The agent has eyes now.

**Smart review routing.** Just like at a well-run startup: CEO doesn't have to look at infra bug fixes, design review isn't needed for backend changes. gstack tracks what reviews are run, figures out what's appropriate, and just does the smart thing. The Review Readiness Dashboard tells you where you stand before you ship.

**Test everything.** `/ship` bootstraps test frameworks from scratch if your project doesn't have one. Every `/ship` run produces a coverage audit. Every `/qa` bug fix generates a regression test. 100% test coverage is the goal — tests make vibe coding safe instead of yolo coding.

**`/document-release` is the engineer you never had.** It reads every doc file in your project, cross-references the diff, and updates everything that drifted. README, ARCHITECTURE, CONTRIBUTING, CLAUDE.md, TODOS — all kept current automatically.

**People are already building on top.** Josh built his L8 software factory on gstack. Others are forking and customizing. That's the whole point.

---

## Demo

```
You:   I want to add seller photo upload to the listing app.

You:   /plan-ceo-review

Claude: "Photo upload" is not the feature. The real job is helping sellers
        create listings that actually sell. Here's the 10-star version:
        auto-identify the product, pull specs and comps from the web,
        draft the title and description, suggest the best hero image...

You:   /plan-eng-review

Claude: [Architecture diagram, state machine for upload → classify → enrich
        → draft pipeline, async job boundaries, failure modes, test matrix]

You:   [implement the plan]

You:   /review

Claude: Race condition: two tabs can overwrite cover-photo selection.
        Trust boundary: web data into draft generation = prompt injection.
        [AUTO-FIXED] 2 issues. 1 needs your input.

You:   /ship

Claude: Tests: 42 → 47 (+5 new). PR: github.com/you/app/pull/42

You:   /qa https://staging.myapp.com

Claude: All 3 affected routes working. Upload + enrichment flow passes
        end to end. Regression test generated.
```

Six commands. Plan → architecture → review → ship → QA. Done.

---

## My journey and why this exists

I have a full time busy job as CEO of Y Combinator. And I am coding a lot. Like, a lot a lot. I was stranded in Austin due to weather and I was coding. My mom was in the hospital and not too lucid for most of it, and I was coding by her bedside too. She's ok now.

Is it really possible for a CEO to be coding all the time? I think it will have to be. The CEO has to set the future of the company. All companies will need to adapt to a faster world and do more. It's not about doing less and cheaper. It's about doing more and making 10x better products and services.

Is 16k LOC/day sustainable for me? We're going to find out if I can manage to get to L8 software factory. But you can tell the models are about to get much much better. L8 is barely possible today. But everyone will be there soon.

I want to be one of the people who helps all of you do it with me.

gstack is how I do it. It's the exact system I use every day. I'm upgrading it live, in public, and sharing everything I learn. There is nothing to sell here. No monetization, no premium tier, no waitlist. Just me trying to figure out the future and bringing everyone along for the ride.

---

## 10 sessions at once

gstack is powerful with one session. It is transformative with ten.

[Conductor](https://conductor.build) runs multiple Claude Code sessions in parallel — each in its own isolated workspace. One session running `/qa` on staging, another doing `/review` on a PR, a third implementing a feature, and seven more on other branches. All at the same time.

One person, ten parallel agents, each with the right cognitive mode. That is a different way of building software.

---

## Install — takes 30 seconds

**Requirements:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Git](https://git-scm.com/), [Bun](https://bun.sh/) v1.0+

### Step 1: Install on your machine

Open Claude Code and paste this. Claude does the rest.

> Install gstack: run `git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup` then add a "gstack" section to CLAUDE.md that says to use the /browse skill from gstack for all web browsing, never use mcp\_\_claude-in-chrome\_\_\* tools, and lists the available skills: /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /review, /ship, /browse, /qa, /qa-only, /qa-design-review, /setup-browser-cookies, /retro, /document-release. Then ask the user if they also want to add gstack to the current project so teammates get it.

### Step 2: Add to your repo so teammates get it (optional)

> Add gstack to this project: run `cp -Rf ~/.claude/skills/gstack .claude/skills/gstack && rm -rf .claude/skills/gstack/.git && cd .claude/skills/gstack && ./setup` then add a "gstack" section to this project's CLAUDE.md that says to use the /browse skill from gstack for all web browsing, never use mcp\_\_claude-in-chrome\_\_\* tools, lists the available skills: /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /review, /ship, /browse, /qa, /qa-only, /qa-design-review, /setup-browser-cookies, /retro, /document-release, and tells Claude that if gstack skills aren't working, run `cd .claude/skills/gstack && ./setup` to build the binary and register skills.

Real files get committed to your repo (not a submodule), so `git clone` just works. Everything lives inside `.claude/`. Nothing touches your PATH or runs in the background.

---

```
+----------------------------------------------------------------------------+
|                                                                            |
|   Are you a great software engineer who wants to ship 10K+ LOC/day?       |
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

## Come ride the wave

This is **free, MIT licensed, open source, available now.** No premium tier. No waitlist. No strings.

I open sourced how I do development and I am actively upgrading my own software factory here. You can fork it and make it your own. That's the whole point. I want everyone on this journey.

The models are getting better fast. The people who figure out how to work with them now — really work with them, not just dabble — are going to have a massive advantage. This is that window. Let's go.

**[github.com/garrytan/gstack](https://github.com/garrytan/gstack)** — MIT License

---

## Docs

| Doc | What it covers |
|-----|---------------|
| [Skill Deep Dives](docs/skills.md) | Philosophy, examples, and workflow for every skill |
| [Greptile Integration](docs/greptile.md) | PR review triage with [Greptile](https://greptile.com) |
| [Contributor Mode](docs/contributor-mode.md) | How to help improve gstack |
| [Browser Reference](BROWSER.md) | Full command reference for `/browse` |
| [Architecture](ARCHITECTURE.md) | Design decisions and system internals |
| [Contributing](CONTRIBUTING.md) | Dev setup, testing, and dev mode |
| [Changelog](CHANGELOG.md) | What's new in every version |

## Troubleshooting

**Skill not showing up?** `cd ~/.claude/skills/gstack && ./setup`

**`/browse` fails?** `cd ~/.claude/skills/gstack && bun install && bun run build`

**Stale install?** Run `/gstack-upgrade` — or set `auto_upgrade: true` in `~/.gstack/config.yaml`

## License

MIT. Free forever. Go build something.
