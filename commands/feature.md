---
description: "Full feature pipeline: brownfield archaeology → disambiguation → PRD → architecture & migration-aware API contract → plan → subagent implementation → regression + Playwright acceptance"
argument-hint: "<requirement text and/or file path(s): .md .txt .docx .doc .pdf>"
---

# /feature — New Feature Pipeline

Process the following input:

$ARGUMENTS

## Phase 0 — Input Parsing (automatic, no gate)

`$ARGUMENTS` may be **requirement text**, **one or more file paths** (.md / .txt / .docx / .doc / .pdf), a **backlog reference** (`backlog#<ID>`, e.g. `backlog#F-012`), or a **mix of the above plus supplementary notes**.

### 0.a Backlog reference (`backlog#<ID>`)

1. Read the entry from `docs/project/backlog.md`. If the file or the ID does not exist, stop and tell the user.
2. **Route re-check**: verify the entry still belongs on this road using the relationship-to-defined-behavior rule (new observable behavior → feature; behavior preserved, structure changed → refactor; deviation from defined behavior → bugfix). Triage-time labels decay as the codebase evolves — if the entry's substance no longer matches `route: feature` (e.g. the capability already partially exists), stop and recommend the correct command instead of forcing this pipeline.
3. Use the entry's Description + Dependencies as the raw requirement; mark the entry 🔧 in-progress. When Gate 4 passes, **write the entry's status back to ✅** in backlog.md.

### 0.b File and text input

1. Classify: any token that looks like a path and verifiably exists on disk is an input file; remaining text is supplementary notes. **Filenames with spaces**: when a path-like token does not resolve on its own, greedily test whether it joined with the following token(s) by single spaces forms a path that *does* exist — if so, treat that whole span as one file before classifying the leftover tokens as notes.
2. Extract content by extension:
   - `.md` / `.txt`: read directly;
   - `.docx`: `pandoc <file> -t gfm -o -`; if pandoc is unavailable, use python-docx (`pip install python-docx`) walking paragraphs and tables;
   - `.doc` (legacy): convert via `libreoffice --headless --convert-to docx`, then treat as docx; if libreoffice is unavailable, ask the user to re-save as docx or md;
   - `.pdf`: `pdftotext -layout <file> -`; fall back to pdfplumber (`pip install pdfplumber`). If extraction yields empty/garbled output (scanned or encrypted PDF), stop and ask the user for a text version;
   - Other extensions: ask the user how to proceed.
3. Archive the extracted content **verbatim** to `docs/pipeline/<feature-slug>/00-source.md`, with a header noting source filename, extraction method, and timestamp. All later phases treat this archive as the authoritative requirement source.
4. Merge extracted content with supplementary notes as the "raw requirement" for Phase 1. On conflict, supplementary notes win — but the conflict must be explicitly confirmed with the user during Phase 1 disambiguation.
5. Pure-text input skips extraction and goes straight to Phase 1.

## Global Rules (take precedence over all default behavior)

1. This command is an **orchestrator**. Methodology lives in the referenced skills; do not improvise process.
2. At every Gate, **stop output completely and wait for explicit user approval** ("approved", "go", "OK"). Without approval, entering the next phase is forbidden. If the user requests changes, revise and re-submit the same gate.
3. All intermediate artifacts must be written to disk in the designated directory — never exist only in conversation. Every artifact carries YAML frontmatter `status: draft | approved`; update to `approved` when its gate passes.
4. Follow the engineering invariants in the project's CLAUDE.md. KISS / YAGNI / DRY are enforced by the superpowers skills themselves; do not restate them.
5. If any referenced `superpowers:*` skill is unavailable, stop and instruct the user to install the superpowers plugin first.
6. Session resume: if this is a resumed run, read existing artifacts under `docs/pipeline/<feature-slug>/`, determine the current phase from each file's `status`, and continue from the breakpoint. Never redo approved phases.

## Artifact Directory Convention

```
docs/pipeline/<feature-slug>/
├── 00-source.md         # Phase 0 (only when file input was given)
├── 01-context.md        # Phase 1.0 (brownfield codebase archaeology)
├── 01-prd.md            # Phase 1
├── 02-design.md         # Phase 2 (architecture + ADRs)
├── 02-contract/         # Phase 2 (openapi.yaml + types.ts + mock notes)
├── 03-plan.md           # Phase 3
└── 04-acceptance/       # Phase 5 (Playwright specs + traceability + report)
```

`<feature-slug>` is a short kebab-case identifier derived from the requirement, e.g. `user-auth-jwt`.

---

## Phase 1.0 — Codebase Archaeology (brownfield only, automatic, no gate)

**Skip entirely for a greenfield project** (empty or near-empty repo). Otherwise, before asking a single disambiguation question, dispatch a subagent to survey the ground the feature will land on. Building a feature into an existing system is the norm, not the exception — and a new feature almost always interacts with existing tables, auth, and modules. Questions asked before reading the code are asked in a vacuum, and Phase 2 is far likelier to invent a pattern that clashes with what already exists.

1. Explore and record conclusions into `01-context.md`: the modules and files the feature will touch or sit beside; the **existing data models** relevant to it (tables/entities and their ownership); **reusable components, services, and utilities** already present; the established **conventions** (naming, error handling, auth, validation) it must conform to; and existing tests around the affected area.
2. Compress findings into conclusions, not code dumps — each conclusion names the concrete artifact (file / table / component) it refers to.
3. Phase 1 disambiguation and the PRD build **on top of** `01-context.md`: questions probe the gap between what exists and what is wanted, and the later design prefers conforming to discovered patterns over inventing parallel ones.

No gate here; the output feeds Phase 1.

---

## Phase 1 — Disambiguation & PRD

0. **Project-context awareness**: if `docs/project/` exists (created by `/trivium:inception`), read `00-vision.md`, `01-architecture.md`, and `02-conventions.md` first. Disambiguation then asks **feature-level questions only** — project-level questions (users, non-goals, stack, conventions) are already answered there and must not be re-asked. The PRD **references** project documents instead of restating them, and must not contradict the vision's non-goals; a genuine conflict goes to the user as a vision question, not a silent PRD decision.
1. Invoke the `superpowers:brainstorming` skill for Socratic disambiguation of the raw requirement. Questions must cover: target users and scenarios, functional boundaries (explicitly what NOT to build), relationship to the existing system, non-functional constraints (performance / security / compatibility), and observable acceptance criteria.
2. When disambiguation completes, invoke the `trivium:prd-writing` skill and generate the PRD per its template into `01-prd.md`.
3. Present the user a **summary** (feature list + priorities + out-of-scope), not the full document.

**🚦 Gate 1: STOP. Await user approval of the PRD.**

---

## Phase 2 — Architecture & API Contract

1. Design the technical architecture from the approved PRD. Record every significant decision (framework, data model, state management, auth, etc.) as an ADR inside `02-design.md`: each ADR contains Context / Options / Decision / Trade-offs.
2. **Reuse before invention (brownfield)**: with `01-context.md` in hand, every "build something new" decision — new table, new module, new service, new component — must be an ADR that explicitly answers *"why the existing X cannot be extended instead"*. Adding a column to an existing table beats a new table; extending an existing module beats standing up a new one. Novelty without that justification is rejected at Gate 2.
3. **Data evolution, not data design (brownfield)**: when the feature changes the schema of a system that already holds data, the deliverable is a **migration script**, not just a schema diagram. It must state, for each change: the default value or backfill strategy for existing rows, whether a nullable transition is needed, index/lock-table risk on large tables (flag it explicitly), and a **rollback plan**. Greenfield tables are designed directly; evolving a populated table without a migration + rollback is forbidden.
4. Split frontend vs. backend responsibilities: which logic lives where, and where the boundary sits.
5. Invoke the `trivium:api-contract` skill to produce the OpenAPI spec + TypeScript types + mock plan into `02-contract/`. **If a project-level contract exists** (`docs/project/contract/`, created by inception): this feature's `02-contract/` contains the **delta** (new/changed endpoints only, reusing the project's shared components — error body, pagination, auth); on Gate 2 approval, merge the delta into the project master contract, regenerate the master `types.ts`, and record the merge in the master `CHANGELOG.md`. Without a project contract, `02-contract/` stands alone as before. Once approved at Gate 2, the contract is **frozen**.
6. Present: architecture summary, key ADR decisions, endpoint list — and, for brownfield, the reuse-vs-new decisions and the migration/rollback plan.

**🚦 Gate 2: STOP. Await user approval of architecture and contract.**

---

## Phase 3 — Task Plan

1. Invoke the `superpowers:writing-plans` skill against `01-prd.md` and `02-design.md`, writing the plan to `03-plan.md`.
2. On top of the superpowers plan template, enforce these pipeline additions:
   - Tag every task `[FE]` / `[BE]` / `[SHARED]`;
   - `[FE]` tasks develop exclusively against the mocks from `02-contract/` — never against a live backend;
   - `[BE]` task verification must include contract tests (implementation matches openapi.yaml);
   - The plan **must end with an `[INTEGRATION]` task**: remove mocks, wire FE to the real backend, prepare for Phase 5 acceptance. A plan without an integration task is incomplete and must not be submitted to Gate 3.
   - **Size guard**: if the plan exceeds ~20 tasks, the feature is cut too coarse. Stop, propose a split into two (or more) independently deliverable features (updating backlog.md if this came from a backlog entry), and let the user choose at Gate 3 — do not swallow an oversized plan.
3. Present the task list (id, tag, one-line goal, dependencies).

**🚦 Gate 3: STOP. Await user approval of the plan.**

---

## Phase 4 — Implementation

1. Invoke the `superpowers:using-git-worktrees` skill to create an isolated workspace with a clean test baseline.
2. **Establish the green baseline**: run the full existing test suite and confirm it passes *before* writing any code. This green baseline is the invariant the whole phase must preserve — the top brownfield risk is a new feature quietly breaking delivered behavior. A red or unknown baseline is resolved with the user first (fix it, or explicitly record which pre-existing failures are out of scope) — never built on top of silently.
3. Invoke the `superpowers:subagent-driven-development` skill to execute `03-plan.md` task by task. That skill has TDD (red-green-refactor) and two-stage review (spec compliance → code quality) built in. **Never nest this command or the other entry commands inside a task.**
4. Issues found in review are rework within the current task: fix in the same task context and re-review. Do not invoke /bugfix.
5. If implementation reveals the API contract must change: **stop implementation**, explain the reason and blast radius, return to Gate 2 for contract re-approval, then update affected tasks.
6. After each task, give the user a brief report: task id, result, test status. This is visibility, not a gate — do not wait for approval.

---

## Phase 5 — Acceptance

1. Confirm the `[INTEGRATION]` task is complete and mocks are removed.
2. **Full regression first**: run the entire existing test suite and confirm the Phase 4 green baseline still holds. Acceptance is **two-sided** — the feature must meet its PRD *and* must not have broken delivered behavior. Any pre-existing test that is now red is a regression: back to Phase 4, fixed in the offending task's context.
3. Invoke the `trivium:acceptance` skill: generate and run Playwright end-to-end specs from the acceptance criteria in `01-prd.md`, producing the traceability matrix and report into `04-acceptance/`.
4. Failed items go back to Phase 4 and are fixed in the corresponding task context (still in-development defects — not /bugfix), then re-run acceptance.
5. When everything passes (full regression green **and** all acceptance criteria met), invoke `superpowers:finishing-a-development-branch` (if available) for branch wrap-up, and present a delivery report: feature completion status, test coverage, regression + acceptance results, open items.

**🚦 Gate 4: STOP. Await final user sign-off.** Then merge / open PR / keep branch per the user's choice.
