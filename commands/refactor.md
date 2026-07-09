---
description: "Refactor / requirement-change pipeline: impact analysis → behavior-pinning tests → proposal with ADRs → plan → subagent implementation → regression"
argument-hint: "<change description and/or file path(s): .md .txt .docx .doc .pdf>"
---

# /refactor — Refactoring & Requirement-Change Pipeline

Process the following input:

$ARGUMENTS

## Phase 0 — Input Parsing (automatic, no gate)

`$ARGUMENTS` may be a **textual change description**, **one or more file paths** (.md / .txt / .docx / .doc / .pdf — e.g. change requests, review feedback, legacy design docs), or both.

1. Classify: tokens that look like paths and exist on disk are input files; the rest is supplementary notes.
2. Extract by extension: `.md`/`.txt` read directly; `.docx` via `pandoc <file> -t gfm -o -` (python-docx fallback); `.doc` via `libreoffice --headless --convert-to docx` first; `.pdf` via `pdftotext -layout` (pdfplumber fallback). If a scanned PDF yields nothing, stop and request a text version.
3. Archive extracted content verbatim to `docs/pipeline/<refactor-slug>/00-source.md` (header: source + method), the authoritative change-request source for later phases.
4. Merge with supplementary notes as the change request for Phase 1; notes win on conflict, confirmed explicitly during disambiguation. Pure-text input goes straight to Phase 1.

## Core Invariant

**Behavior changes only under control; structure may be rewritten.** Pure refactoring = zero behavior change. Requirement change = only the explicitly approved behavioral deltas; everything else stays identical. In both cases, pin down the "must not change" behavior with tests BEFORE touching code. Refactoring without a behavioral baseline is gambling — forbidden.

## Global Rules

1. This command is an orchestrator; methodology lives in the referenced skills.
2. At every Gate, **stop completely and await explicit user approval**.
3. Artifacts go to `docs/pipeline/<refactor-slug>/` with `status: draft | approved` frontmatter.
4. If any referenced `superpowers:*` skill is unavailable, stop and instruct the user to install the superpowers plugin.
5. Session resume: read the artifact directory first; continue from the breakpoint.

## Artifact Directory Convention

```
docs/pipeline/<refactor-slug>/
├── 00-source.md         # original change request archive (file input only)
├── 01-impact.md         # impact analysis
├── 02-proposal.md       # change proposal + ADRs
├── 03-plan.md           # implementation plan
└── 04-regression/       # regression & acceptance reports
```

---

## Phase 1 — Impact Analysis & Behavior Pinning

1. **Read code before asking questions.** Use a subagent to explore the current state: affected files and modules, all callers and callees, involved API endpoints and data structures, existing test coverage. Compress long findings into conclusions in `01-impact.md`.
2. If the change request is ambiguous (how far to go, which behaviors may change, performance/compat constraints), invoke `superpowers:brainstorming` to disambiguate with the user. The output must explicitly separate **behaviors allowed to change** (the requirement delta) from **behaviors that must be preserved**.
3. **Establish the behavioral baseline**: write characterization tests for every must-preserve behavior — record current input/output faithfully without judging whether it is "correct". For externally consumed APIs, baseline against the existing `02-contract/` (if present) or observed behavior. Run the full suite and confirm a green baseline.
4. Present: impact summary, behavior classification (preserve vs. allowed-to-change), baseline test status.

**🚦 Gate 1: STOP. Await user confirmation of impact and behavior classification.**

---

## Phase 2 — Change Proposal

1. Write the proposal to `02-proposal.md`, containing:
   - Target structure vs. current structure comparison;
   - An ADR (Context / Options / Decision / Trade-offs) for every significant choice — especially "why refactor now" and "why this scope rather than a smaller one". The most common refactoring failure is scope creep; the proposal must argue it is the **minimal change satisfying the goal** (KISS);
   - Migration strategy: big-bang vs. incremental (strangler pattern, feature flags, dual-write); any data migration requires a rollback plan;
   - If the API contract must change: invoke `trivium:api-contract` to produce the new contract version plus a diff, marking breaking changes.
2. Present the proposal summary and key ADRs.

**🚦 Gate 2: STOP. Await user approval of the proposal.**

---

## Phase 3 — Task Plan

1. Invoke `superpowers:writing-plans` to generate the plan into `03-plan.md`.
2. Pipeline additions:
   - Every task's verification **must include "baseline tests stay green"** (tests covering approved behavior changes are updated in their own explicit tasks — never edited in passing);
   - In incremental migrations, the system must be in a **releasable state after every task**; prolonged broken intermediate states are forbidden;
   - If frontend and backend are both touched, apply the /feature rules: `[FE]`/`[BE]` tags, `[INTEGRATION]` task at the end.
3. Present the task list.

**🚦 Gate 3: STOP. Await user approval of the plan.**

---

## Phase 4 — Implementation

1. Invoke `superpowers:using-git-worktrees` for an isolated workspace.
2. Invoke `superpowers:subagent-driven-development` to execute task by task (TDD and two-stage review built in).
3. **If any task turns a baseline test red outside the approved behavior deltas: stop that task immediately** and invoke `superpowers:systematic-debugging` to find out why — implementation error means fix it; proposal flaw means return to Gate 2.
4. Brief the user after each task.

---

## Phase 5 — Regression & Wrap-up

1. Full regression: all baseline tests plus updated tests green.
2. If the project has Playwright acceptance specs (`docs/pipeline/*/04-acceptance/`), invoke `trivium:acceptance` to re-run the relevant end-to-end specs and confirm user-visible behavior.
3. Write the report to `04-regression/`: preserved-behavior verification, approved-delta verification, performance comparison (if the proposal set performance goals).
4. Invoke `superpowers:finishing-a-development-branch` (if available) for wrap-up.

**🚦 Gate 4: STOP. Await final user sign-off.**
