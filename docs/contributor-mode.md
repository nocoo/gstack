# Contributor Mode

gstack is open source. If something annoys you, you can fix it yourself. Contributor mode makes that easier by automatically detecting friction and writing up what went wrong.

## Enable it

```
gstack-config set gstack_contributor true
```

## How it works

At the end of each major workflow step, the agent reflects on the gstack tooling it just used. It rates the experience 0-10. If it wasn't a 10, it thinks about why. If there's an obvious, actionable bug or an insightful improvement, it files a **field report** to `~/.gstack/contributor-logs/`.

Reports include:
- What the user/agent was trying to do
- What actually happened instead
- A 0-10 rating with one-sentence explanation
- Steps to reproduce
- Raw output (the actual error or unexpected behavior)
- "What would make this a 10" — one sentence focusing on the actionable fix

## Calibration

The bar is set at real-but-small bugs. Example: `$B js "await fetch(...)"` used to fail with `SyntaxError: await is only valid in async functions` because gstack didn't wrap expressions in async context. The input was reasonable, gstack should have handled it — that's worth filing.

Things NOT worth filing: your app's bugs, network errors to your URL, auth failures on your site, your own JS logic bugs. Those aren't gstack's fault.

## What to do with reports

Browse `~/.gstack/contributor-logs/` periodically. Each report is a self-contained bug report. Fork gstack, fix the issue, and submit a PR. Max 3 reports per session to avoid noise.

## Session awareness

When you have 3+ gstack sessions open simultaneously, every question tells you which project, which branch, and what's happening. No more staring at a question thinking "wait, which window is this?"

Every question — in every skill, even in a single session — states the project and branch, explains the problem in plain English, and gives an opinionated recommendation. The format is consistent across all 13 skills.
