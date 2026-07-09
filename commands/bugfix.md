---
description: "Defect-fix pipeline: failing reproduction test first → systematic root-cause analysis → minimal fix → red-to-green + full regression. Accepts single bug text/doc or an Excel/CSV bug list (batch mode)"
argument-hint: "<bug description, error output, or file path: .md .txt .docx .pdf .xlsx .csv>"
---

# /bugfix — Defect Fix Pipeline

Process the following input:

$ARGUMENTS

## Phase 0 — Input Parsing & Mode Selection

`$ARGUMENTS` may be: **text describing a single bug**; a **file path** — either a bug description document (.md / .txt / .docx / .doc / .pdf) or a **bug-list spreadsheet (.xlsx / .xls / .csv)**; a **backlog reference** (`backlog#<ID>`); or a file plus supplementary notes.

### 0.0 Backlog reference

Read the entry from `docs/project/backlog.md` (stop if missing). **Route re-check**: confirm this is genuinely a deviation from defined behavior in delivered code — if it introduces new behavior, redirect to `/trivium:feature`; if it is behavior-preserving restructuring, redirect to `/trivium:refactor`. Use the entry's Description as the bug description; mark 🔧, write back ✅ on final sign-off. Then proceed in single-bug mode.

### 0.1 Document files (.md/.txt/.docx/.doc/.pdf) → single-bug mode

Extract by extension (`.docx` via pandoc or python-docx; `.doc` via libreoffice conversion; `.pdf` via pdftotext or pdfplumber; on failed extraction, stop and request a text version). The content becomes the bug description and proceeds to Scope Check and Phase 1. If one document describes multiple bugs, switch to batch mode below (manually compile the inventory).

### 0.2 Spreadsheet lists (.xlsx/.xls/.csv) → batch mode

1. **Parse** with Python (`pip install pandas openpyxl`; legacy .xls also needs `xlrd`). First print column names and the first 3 rows to understand the structure, then map column semantics (title / description / repro steps / severity / status / environment / reporter). When headers are non-standard, infer from content **and show the user the mapping for confirmation**. Rows already marked fixed/closed are skipped by default (listed in the inventory as ⏭).
2. **Generate the triage inventory** at `docs/pipeline/bugs-<YYYYMMDD>/inventory.md`, archiving the source spreadsheet alongside it. One entry per bug:

```markdown
### BUG-001: <title>  [severity] [status: ⬜]
Source row: N | Env: ...
Description: <excerpt>
Initial assessment: <reproducibility / suspected shared root cause with other bugs / actually a requirement change that should go to /refactor>
```

3. **Root-cause grouping**: bugs with different symptoms but a suspected common root cause form a group (e.g. `GROUP-A: BUG-003, BUG-007`). One group goes through one fix cycle — avoiding fixing the same root cause multiple times in conflicting ways.
4. **🚦 Triage Gate: STOP.** Present the triage summary (totals, severity distribution, groups, proposed order, items recommended for skipping or /refactor). Await user confirmation of: which to fix, in what order, and the sign-off style — **per-bug sign-off** or **batch sign-off** (one review after all fixes).
5. **Execute item by item**: for each bug (or root-cause group) run Phases 1–4 below. **The per-bug Reproduction Gate is never skippable** — it is the foundation of this pipeline. The final sign-off follows the user's triage choice. Update inventory status immediately after each item: ⬜ pending / 🔧 in-progress / ✅ fixed / ⏭ skipped / ➡ moved-to-refactor / ❓ cannot-reproduce (record attempted reproduction strategies; awaiting more info).
6. **Batch report**: after all items, summarize — fixes (one-line root cause each), non-reproducible items, moved items, defense hardening; also produce a results column mappable back to the source spreadsheet (bug id → status → commit).
7. **Resume**: after interruption, re-run this command with the same inventory.md path; continue from unfinished items, never redoing ✅ entries.

### 0.3 Plain text → single-bug mode, proceed directly.

## Core Invariants

1. **No failing reproduction test, no production-code changes.** If you cannot prove the bug exists, you cannot prove it is fixed.
2. **No root cause understood, no fix attempted.** Root-cause analysis is enforced by `superpowers:systematic-debugging`, which explicitly forbids fixing what you have not understood.
3. **Minimal fix.** Fix the root cause only — no drive-by refactoring, no opportunistic optimization, no scope expansion. Other problems discovered along the way are recorded as new entries with route labels in `docs/project/backlog.md` (or `docs/pipeline/backlog.md` if no project backlog exists); anything needing restructuring gets `route: refactor`.

If any referenced `superpowers:*` skill is unavailable, stop and instruct the user to install the superpowers plugin.

## Scope Check (do this first)

This command is for defects in **delivered / merged code**. If the bug was found by review or acceptance during an active /feature or /refactor run, it is rework belonging to that task's context — return there; **do not use this command**.

---

## Phase 1 — Reproduction

1. Locate the observable manifestation. If information is insufficient, ask the user: expected vs. actual behavior, trigger conditions, environment, frequency (deterministic / intermittent).
2. **Write a failing automated test that reproduces the bug**, named to express the defect (e.g. `test_login_fails_when_email_contains_plus_sign`). Choose the lowest level that reproduces it: prefer a unit test over an integration test; use Playwright only for UI-level defects.
3. Run it and **confirm it fails in a way consistent with the bug** (the failure must match the defect — not an environment error or a mis-written assertion).
4. Intermittent bugs: make them deterministic first (fixed random seeds, controlled concurrency timing, injected clocks). If truly non-deterministic, explain to the user and agree on a verification strategy before continuing.
5. Present: the reproduction test, its failing output, and your explanation of how the failure maps to the bug.

**🚦 Gate: STOP. Await user confirmation that "this failure reproduces the bug I reported."**

---

## Phase 2 — Root Cause

1. Invoke `superpowers:systematic-debugging` and follow its four-phase process.
2. Produce the root-cause conclusion: where the defect was introduced (which change / which broken assumption), its propagation path, and why existing tests did not catch it.
3. Brief the user (one paragraph; not a gate — unless the root cause reveals a fix scope far beyond expectations, in which case stop and consult; it may belong in /refactor).

## Phase 3 — Minimal Fix

1. Implement the minimal fix targeting the root cause.
2. Run the reproduction test: **red turns green**.
3. Run the **full test suite**: no regressions. If the fix breaks other tests, return to Phase 2 (the root-cause understanding is likely incomplete). Making the suite green by editing unrelated tests is forbidden.
4. Harden the defense: if the analysis shows the same defect class may exist elsewhere (copy-pasted error patterns, the same broken assumption used in multiple places), check each; fix confirmed instances with their own tests in this run, or record them to backlog with user consent.

## Phase 4 — Review & Wrap-up

1. Invoke `superpowers:requesting-code-review` on the fix diff, focusing on: does the fix target the root cause, is the scope minimal, are the tests sufficient.
2. If the bug touches a user-visible flow and Playwright acceptance specs exist, re-run the relevant specs.
3. Present the fix report: root cause, change, reproduction test (red→green evidence), regression results, hardening done.

**🚦 Gate: STOP. Await final user sign-off, then commit / merge per their choice.**
