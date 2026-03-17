# Greptile Integration

[Greptile](https://greptile.com) is a YC company that reviews your PRs automatically. It catches real bugs — race conditions, security issues, things that pass CI and blow up in production. It has genuinely saved my ass more than once. I love these guys.

## Setup

Install Greptile on your GitHub repo at [greptile.com](https://greptile.com) — it takes about 30 seconds. Once it's reviewing your PRs, gstack picks up its comments automatically. No additional configuration.

## How it works

The problem with any automated reviewer is triage. Greptile is good, but not every comment is a real issue. Some are false positives. Some flag things you already fixed three commits ago. Without a triage layer, the comments pile up and you start ignoring them — which defeats the purpose.

gstack solves this. `/review` and `/ship` are now Greptile-aware. They read Greptile's comments, classify each one, and take action:

- **Valid issues** get added to the critical findings and fixed before shipping
- **Already-fixed issues** get an auto-reply acknowledging the catch
- **False positives** get pushed back — you confirm, and a reply goes out explaining why it's wrong

The result is a two-layer review: Greptile catches things asynchronously on the PR, then `/review` and `/ship` triage those findings as part of the normal workflow. Nothing falls through the cracks.

## Learning from history

Every false positive you confirm gets saved to `~/.gstack/greptile-history.md`. Future runs auto-skip known FP patterns for your codebase. And `/retro` tracks Greptile's batting average over time — so you can see whether the signal-to-noise ratio is improving.

## Two-tier reply system

- **Tier 1 (friendly):** First response to a comment. Includes inline diff, explanation, and evidence.
- **Tier 2 (firm):** When Greptile re-flags the same issue after a prior reply. Full evidence chain + re-rank request.

Every reply includes evidence — inline diffs, code references, suggested severity re-ranking. No vague one-line replies.

## Example

```
You:   /ship

Claude: [syncs main, runs tests, pre-landing review...]

        Greptile found 3 comments on this PR:

        [VALID] app/services/payment_service.rb:47 — Race condition:
        concurrent charges can double-debit. Recommend DB-level advisory
        lock around the charge block.
        → https://github.com/you/app/pull/42#discussion_r123

        [ALREADY FIXED] app/models/user.rb:12 — Missing null check
        on email before downcase.
        → Already fixed in commit abc1234. Auto-replying.

        [FALSE POSITIVE] lib/auth.rb:88 — "Token comparison should
        use constant-time comparison." We already use
        ActiveSupport::SecurityUtils.secure_compare here.
        → https://github.com/you/app/pull/42#discussion_r125

        Fixing the race condition in payment_service.rb...
        Applied advisory lock. Committed: fix: address Greptile
        review — advisory lock on concurrent charges

        The auth token comment is a false positive — we're already
        using secure_compare.

        Options for the false positive:
          A) Reply to Greptile explaining (recommended)
          B) Fix it anyway
          C) Ignore

You:    A

Claude: Replied to Greptile. Re-running tests after the fix...
        All tests pass. Continuing to version bump.

        [creates PR with Greptile Review section in body]
        PR: https://github.com/you/app/pull/42
```

Three Greptile comments. One real fix. One auto-acknowledged. One false positive pushed back with a reply. Total extra time: about 30 seconds. And the PR body has the full audit trail.
