---
name: perf-execute
description: Implement a plan file or a direct prompt using parallel implementation agents for independent steps, then verify with lint/type/test/build and an independent review agent. Use when the user runs /perf-execute or asks to build, implement, or carry out a plan.
---

# /perf-execute

Implement `$ARGUMENTS`: a path to a plan (e.g. from `/perf-plan`), or a direct prompt. If empty, look for the most recent `PLAN-*.md` (in `docs/` or root) and confirm it with the user; if none, ask.

## Process

### 1. Prepare (you, inline)
- Read the plan fully (or, for a bare prompt, do a quick scoping pass and write a short step list yourself; for anything non-trivial suggest `/perf-plan` first).
- Read project instructions (CLAUDE.md, AGENTS.md) and follow them.
- Check `git status`. Note pre-existing uncommitted changes so you never clobber or revert them. Do not commit, push, or run destructive commands unless the user asked.
- Create a task list from the plan steps.

### 2. Sequence the work
Group steps into waves by dependency. Steps marked parallel-safe, or touching disjoint files, go in the same wave. Steps sharing files, schemas, or types run sequentially. Shared foundations (types, schema, config) go first, in the main thread or a single agent.

### 3. Implement (agents per wave)
For each wave, launch one `general-purpose` agent per independent step **in a single message**. Each prompt must be self-contained:
- the step text, acceptance criteria, and exact files it owns
- conventions to follow and utilities to reuse (with paths)
- "edit only your owned files; do not commit; do not touch unrelated code; no unrequested features; run the quick check for your files; report files changed and anything surprising, under 200 words"

Use `isolation: "worktree"` only if steps might collide; otherwise work in place. After each wave, read the agents' reports, inspect the actual diffs (`git diff`), resolve conflicts or gaps yourself, then start the next wave. If a step's premise is wrong (file missing, plan outdated), adapt minimally and note the deviation; stop and ask if the deviation changes scope.

### 4. Verify
Run the project's own checks (from the plan or package scripts): typecheck, lint, tests, build, as applicable. Fix failures by root cause; never weaken checks, skip tests, or use `--no-verify`. In parallel, launch:
- a **reviewer** agent (`general-purpose`) given the plan and `git diff`: finds bugs, missed acceptance criteria, regressions, security issues, leftover debug code. Read-only.
- a **tester** agent when the change is runnable UI/API: exercise it (existing tests, dev server, or browser tools) against the acceptance criteria.

Triage findings: fix real issues, rerun checks. Repeat at most twice; report anything still open.

### 5. Report
Final message, plain and faithful:
- What was implemented, mapped to plan steps/acceptance criteria
- Files changed
- Checks run with actual results (state failures and skipped checks explicitly)
- Deviations from the plan, and follow-ups
- Not committed; offer to commit/PR if wanted

Never claim something works that you did not run or observe.
