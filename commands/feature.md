---
description: "Full feature pipeline: disambiguation → PRD → architecture & API contract → plan → subagent implementation → Playwright acceptance"
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

1. Classify: any token that looks like a path and verifiably exists on disk is an input file; remaining text is supplementary notes.
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
├── 01-prd.md            # Phase 1
├── 02-design.md         # Phase 2 (architecture + ADRs)
├── 02-contract/         # Phase 2 (openapi.yaml + types.ts + mock notes)
├── 03-plan.md           # Phase 3
└── 04-acceptance/       # Phase 5 (Playwright specs + traceability + report)
```

`<feature-slug>` is a short kebab-case identifier derived from the requirement, e.g. `user-auth-jwt`.

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
2. Split frontend vs. backend responsibilities: which logic lives where, and where the boundary sits.
3. Invoke the `trivium:api-contract` skill to produce the OpenAPI spec + TypeScript types + mock plan into `02-contract/`. **If a project-level contract exists** (`docs/project/contract/`, created by inception): this feature's `02-contract/` contains the **delta** (new/changed endpoints only, reusing the project's shared components — error body, pagination, auth); on Gate 2 approval, merge the delta into the project master contract, regenerate the master `types.ts`, and record the merge in the master `CHANGELOG.md`. Without a project contract, `02-contract/` stands alone as before. Once approved at Gate 2, the contract is **frozen**.
4. Present: architecture summary, key ADR decisions, endpoint list.

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
2. Invoke the `superpowers:subagent-driven-development` skill to execute `03-plan.md` task by task. That skill has TDD (red-green-refactor) and two-stage review (spec compliance → code quality) built in. **Never nest this command or the other entry commands inside a task.**
3. Issues found in review are rework within the current task: fix in the same task context and re-review. Do not invoke /bugfix.
4. If implementation reveals the API contract must change: **stop implementation**, explain the reason and blast radius, return to Gate 2 for contract re-approval, then update affected tasks.
5. After each task, give the user a brief report: task id, result, test status. This is visibility, not a gate — do not wait for approval.

---

## Phase 5 — Acceptance

1. Confirm the `[INTEGRATION]` task is complete and mocks are removed.
2. Invoke the `trivium:acceptance` skill: generate and run Playwright end-to-end specs from the acceptance criteria in `01-prd.md`, producing the traceability matrix and report into `04-acceptance/`.
3. Failed items go back to Phase 4 and are fixed in the corresponding task context (still in-development defects — not /bugfix), then re-run acceptance.
4. When everything passes, invoke `superpowers:finishing-a-development-branch` (if available) for branch wrap-up, and present a delivery report: feature completion status, test coverage, acceptance results, open items.

**🚦 Gate 4: STOP. Await final user sign-off.** Then merge / open PR / keep branch per the user's choice.
