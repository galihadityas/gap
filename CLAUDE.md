# Website files are off limits

This repo serves a live website. Never modify, rename, or delete the site files (`index.html`, `20260505_index.html`, `CNAME`, and any other file the site serves) without the owner's explicit permission in the current conversation. Ask first, and say exactly what you will change. Repo tooling under `.claude/` and this file are fine to change through a pull request.

# Model routing (always on, silent)

Standing instruction from the repo owner: on every request, route work to the cheapest model that will do it reliably. No trigger word is needed. Do not narrate routing unless asked.

Delegation pays only when a worker absorbs volume the main model would otherwise read or produce. Judge by that, not by how "easy" the task sounds. Each worker has a fixed startup cost (tens of thousands of cheap tokens), so delegate only when the material is large: roughly 10+ files, 1,000+ lines, or a long log. Below that, do it inline.

Keep inline (main model):
- Anything whose input is already in the conversation: grammar, rewriting, translation, short summaries, explanations, calculations, classification. Answering directly is cheaper than briefing a worker.
- Tiny work: one short file, one obvious edit, one search.
- Judgment: architecture, ambiguous or hard debugging (concurrency, distributed, intermittent), security, auth, credentials, permissions, deploys, destructive or irreversible operations, financial math, strategy, final integration.

Delegate to `light-worker` (Haiku):
- Searching the repo, locating references, reading many files to extract facts, summarizing long logs or files, uniform mechanical edits across many files (size alone does not make it hard).

Delegate to `medium-worker` (Sonnet):
- A contained feature or module refactor with a clear spec, tests following existing patterns, first-pass review of a small diff, multi-step transformations.

Mixed requests: keep planning, decisions and verification inline; send the mechanical slices down. Never send mechanical work to an Opus-class agent.

Briefs: task, context (paths, pattern to copy), what not to touch, done-when, compact return format. Run independent briefs in parallel.

Escalation: read the worker's final STATUS line.
- DONE: verify cheaply, never by redoing the work. For edits: one grep or the test suite plus a glance at one diff. For extractions: spot-check 1-2 items. Re-reading every file the worker read wipes out the saving.
- PARTIAL, ESCALATE, a missing STATUS line, failed checks, or a wrong result: re-brief one tier up (light-worker -> medium-worker -> main). Never retry the same tier on the same task.

The main model owns the final answer: one coherent response to the user.
