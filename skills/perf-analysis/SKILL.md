---
name: perf-analysis
description: Deep, read-only multi-agent analysis of a codebase against a requirement, feature, bug or area. Maps requirements, implementation, functionality, data flow, risks and gaps, then returns one evidence-backed report. Use when the user runs /perf-analysis or asks to analyze, audit, or understand how something is (or should be) implemented.
---

# /perf-analysis

Analyze a requirement, feature, or area of the codebase and report what is required, what exists, what works, and what is missing or risky. **Read-only: never edit, create, or delete project files.**

Input: `$ARGUMENTS` (text, file paths, image paths, URLs, issue links). If empty, ask the user what to analyze. Read any referenced files/images yourself first.

## Process

### 1. Frame (you, inline)
Restate the target in 2-4 lines: goal, scope, assumptions. Read project instructions (CLAUDE.md, AGENTS.md, README) and note stack, conventions, and any warnings (e.g. framework versions that differ from your training data: read the local docs before judging code).

### 2. Fan out (parallel agents)
Launch these in ONE message so they run concurrently. Use `Explore` agents (read-only), thoroughness "medium" or "very thorough" by scope. Give each a self-contained prompt containing the framed target, what to return, and "report file:line evidence; do not modify files; under 400 words".

| Agent | Question |
|---|---|
| Requirements | Break the target into discrete, testable requirements/acceptance criteria. Flag ambiguities. |
| Implementation map | Which files, modules, components, routes, schemas, and APIs implement it? Entry points and call graph. |
| Functionality | Does the current behavior satisfy each requirement? Trace actual execution paths, edge cases, error handling. |
| Data & integration | Data models, migrations, external services, env vars, auth/permissions, state flow. |
| Quality & risk | Bugs, security issues, performance hotspots, duplication, dead code, missing tests, type-safety holes. |
| Prior art | Existing utilities/patterns to reuse; conventions a change must follow. |

Drop agents that clearly don't apply (e.g. no data layer). Add one for UI/UX, a11y, or i18n when the target is user-facing.

### 3. Verify
Spot-check the 3-5 most important or surprising claims by reading the cited lines yourself. Discard or correct anything unsupported. Do not forward agent claims you haven't reconciled with each other.

### 4. Report
Output a single report:

1. **Summary**: 3-5 sentence verdict.
2. **Requirements vs. status** table: requirement | status (Done / Partial / Missing / Broken) | evidence `file:line`.
3. **How it works today**: concise implementation walkthrough.
4. **Gaps & risks**: ranked High/Med/Low, each with evidence and impact.
5. **Reuse opportunities**.
6. **Open questions** for the user.
7. **Suggested next step**: usually `/perf-plan <target>`.

Keep it tight; cite `path:line` for every claim. Say plainly what you could not verify.
