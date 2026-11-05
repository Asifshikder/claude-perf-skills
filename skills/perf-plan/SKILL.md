---
name: perf-plan
description: Turn any input (requirement, prompt, screenshot/mockup, spec, bug report, URL) into a verified, step-by-step implementation plan using parallel research and design agents plus a critic pass. Writes the plan to a file. Use when the user runs /perf-plan or asks to plan a feature, change, or fix before building.
---

# /perf-plan

Produce an executable implementation plan from `$ARGUMENTS`: free text, a requirement doc, a prompt, image/mockup paths, URLs, or a prior `/perf-analysis` report. If empty, ask what to plan. **Plan only: never modify source files.** The only file you may write is the plan.

## Process

### 1. Ingest (you, inline)
- Read every referenced file and view every image (use Read on image paths). Extract: goal, user-visible behavior, constraints, non-goals.
- Read project instructions (CLAUDE.md, AGENTS.md); honor them (e.g. read local framework docs if they warn of breaking changes).
- If a decision is truly the user's and blocks the plan, ask (AskUserQuestion) once, batching questions. Otherwise pick sensible defaults and list them as assumptions.

### 2. Research (parallel agents, one message)
Launch concurrently, each self-contained, read-only, returning `file:line` evidence in under 400 words:

| Agent | Task |
|---|---|
| Codebase mapper (`Explore`) | Files, modules, routes, schemas, components the change touches; existing patterns and utilities to reuse. |
| Dependencies & data (`Explore`) | Data model/migration impact, env/config, external APIs, auth, state, affected consumers. |
| Test & verify (`Explore`) | Existing test setup, lint/typecheck/build commands, how to verify this change. |
| Requirement breakdown (`general-purpose`) | Turn the input into numbered acceptance criteria; list edge cases and ambiguities. |

Add a UI agent when images/mockups are involved (layout, components, states, responsive behavior) and a research agent (WebSearch/docs) for unfamiliar libraries or APIs.

### 3. Design (agent)
Launch a `Plan` agent with the research results and ask for: approach, alternatives considered with a recommendation, ordered steps, risks. For large or risky work, run 2 `Plan` agents with different angles (e.g. minimal-change vs. clean-architecture) and choose or merge.

### 4. Critique (agent)
Launch a fresh `general-purpose` reviewer with the draft plan: find missing steps, wrong file references, ordering/dependency errors, unhandled edge cases, scope creep, and unverifiable steps. Verify any cited path actually exists. Fold valid findings into the plan.

### 5. Write the plan
Save to `PLAN-<kebab-slug>.md` in the project's `docs/` folder if it exists, else the project root (or the path the user names). Structure:

```
# Plan: <title>
## Goal & scope          (incl. non-goals)
## Assumptions & open questions
## Acceptance criteria   (numbered, testable)
## Approach              (chosen design + why; alternatives rejected)
## Steps                 (numbered; each: files, change, depends-on, parallel-safe? yes/no, verify)
## Verification          (exact lint/type/test/build commands + manual checks)
## Risks & rollback
```

Steps must be small, ordered, and concrete (real paths, function names). Mark steps that touch disjoint files as `parallel-safe: yes` so `/perf-execute` can run them concurrently.

### 6. Hand off
Reply with: the plan path, a 5-line summary, assumptions the user should confirm, and the next command: `/perf-execute <plan path>`. Do not start implementing.
