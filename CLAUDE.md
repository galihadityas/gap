# Website files are off limits

This repo serves a live website. Never modify, rename, or delete the site files (`index.html`, `20260505_index.html`, `CNAME`, and any other file the site serves) without the owner's explicit permission in the current conversation. Ask first, and say exactly what you will change. Repo tooling under `.claude/` and this file are fine to change through a pull request.

# Model routing (always on)

This is a standing request from the repo owner: route work to the cheapest model that can do it correctly, without waiting to be asked. Follow `.claude/skills/delegate/SKILL.md` for every task in this repo.

On every request, before acting:

1. Size the work. If it takes 1-2 tool calls, or it is a plain question you can answer from context, do it inline. No subagent.
2. Split it. Separate the judgment (design, debugging, tradeoffs, final answer) from the mechanical parts (searching, reading many files, summarizing, pattern-following edits).
3. Route the mechanical parts:
   - Read-only lookup or summarizing: `Explore` or `general-purpose` with `model: "haiku"`.
   - Small edits that copy an existing pattern (1-3 files): `general-purpose` with `model: "sonnet"`.
   - More than 3 files, subtle logic (regex, dates, concurrency), or a failed Haiku run: go up one tier.
4. Keep the judgment inline. Never delegate to `opus`, and never delegate risky work (secrets, deletions, force-push, deploys).
5. Verify what comes back (read the diff, spot-check a cited `file:line`) before using it.
6. End the reply with one line naming what went to which model, or "All inline" when nothing was delegated.
