---
name: delegate
description: Offload light, well-scoped work to a subagent on a cheaper model (Haiku or Sonnet) so the main session keeps its context for hard decisions. Use when a task, or a slice of a larger task, is mechanical, read-only, or low-judgment - file/code search, summarizing files or logs, bulk renames, boilerplate, formatting, simple single-file edits, writing straightforward tests, checking links, gathering facts. Also use when the user says "delegate", "use a subagent", "offload this", or "/delegate". Do not use for design decisions, debugging unknown causes, security-sensitive changes, or anything under ~2 tool calls.
---

# Delegate light work to the right model

Goal: spend expensive-model tokens only on judgment. Push everything mechanical to a subagent running the cheapest model that can do it correctly.

## 1. Decide: delegate or do it inline

Do it inline (no subagent) when any of these hold:
- It takes 1-2 tool calls (one grep, one small edit). A subagent's cold start costs more than the work.
- You need the raw output in your own context to make the next decision.
- The task needs conversation context that is hard to hand over in a short prompt.
- It is risky: auth, secrets, payments, data deletion, migrations, force-push, anything irreversible.

Delegate when all of these hold:
- The task has a clear, checkable definition of done.
- It fits in a self-contained prompt (under ~300 words of context).
- You only need a conclusion or a small diff back, not every file it read.

## 2. Pick the model

Pick the cheapest tier that will get it right the first time. A failed cheap run followed by a retry costs more than one correct mid-tier run.

| Tier | `model` | Use for | Examples |
|---|---|---|---|
| Light | `haiku` | Read-only lookup, extraction, pattern-following, no design choices | Find where X is defined; list all TODOs; summarize a log; count usages; check which files import Y; convert a list to JSON; fix typos |
| Medium | `sonnet` | Small writes that follow an existing pattern, needs some reasoning | Edit 1-3 files to a spec; add a test mirroring existing tests; rename across the repo; write a doc section from code; draft a commit message from a diff |
| Heavy | stay inline | Anything ambiguous or high-stakes | Architecture, root-causing unknown bugs, security review, cross-cutting refactors |

Rules:
- Default to `haiku` for read-only work. Default to `sonnet` for any task that edits files.
- Upgrade one tier if the task involves more than 3 files, unfamiliar code, or subtle correctness (off-by-one, concurrency, regex, dates/timezones).
- Never delegate "light" work to `opus`. If it needs opus, it is not light: keep it inline.
- If a `haiku` run returns wrong or incomplete work once, rerun on `sonnet`. Do not retry `haiku`.

## 3. Pick the agent type

- `Explore`: read-only searches across many files. Pair with `haiku`.
- `general-purpose`: anything that edits files or runs commands. Pair with `haiku` or `sonnet` per the table.
- Use `isolation: "worktree"` when the subagent edits files in parallel with you or with another subagent.

## 4. Write the prompt

The subagent starts cold. Give it everything it needs and nothing else:

```
Task: <one sentence, imperative>
Context: <paths, function names, the pattern to copy, constraints>
Do not: <files or areas to leave alone; no commits/pushes unless asked>
Done when: <checkable condition, e.g. "tests in X pass", "list every file path">
Return: <exact shape: file:line list, a unified diff summary, <=10 bullet points>
```

Keep the return format small. The whole point is to protect your context.

## 5. Run it

- Independent light tasks: launch them in one message so they run in parallel.
- If your next step depends on the result, set `run_in_background: false`. Otherwise leave it in the background and keep working.
- Never fabricate a result while a subagent is still running.

## 6. Verify before trusting

Subagent output is a claim, not a fact.
- Edits: read the diff (`git diff`) and run the relevant check yourself.
- Search results: spot-check at least one cited `file:line`.
- If it is wrong, fix inline or escalate one tier (step 2). Do not loop the same model.

## 7. Report

Tell the user in one line which tasks you delegated and to which model, e.g. "Delegated the import search to Haiku (Explore) and the test scaffold to Sonnet."
