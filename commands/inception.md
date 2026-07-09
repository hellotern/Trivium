---
description: "Project inception: vision constitution → system architecture & conventions → epic/feature backlog with route labels and a walking-skeleton first entry. Run once per project; the three roads then consume the backlog"
argument-hint: "<project vision text and/or file path(s): .md .txt .docx .doc .pdf>"
---

# /inception — Project Inception Pipeline

Process the following input:

$ARGUMENTS

## What this command is — and is not

This command draws the **map** for a new (or newly adopted) project: the vision, the system-level architecture, the global conventions, and a feature-grained backlog. It does **not** implement anything, and it deliberately stops decomposition at the feature level — task-level planning happens just-in-time inside each `/trivium:feature` run, because plans decay with distance from implementation. Producing hundreds of upfront tasks is forbidden.

Run it once per project. Re-run only for major direction changes, and then only regenerate the affected artifacts (existing `approved` artifacts are inputs, not things to overwrite silently).

## Phase 0 — Input Parsing (automatic, no gate)

Same rules as the other trivium commands: `$ARGUMENTS` may be vision text, file paths (.md / .txt / .docx / .doc / .pdf — extract via pandoc / python-docx / libreoffice / pdftotext / pdfplumber; stop on failed extraction and request a text version), or both. Archive extracted content verbatim to `docs/project/00-source.md`. Supplementary notes win on conflict, confirmed during disambiguation.

## Global Rules

1. At every Gate, **stop completely and await explicit user approval**.
2. All artifacts go to `docs/project/` with `status: draft | approved` frontmatter. Session resume: read the directory, continue from the breakpoint, never redo approved artifacts.
3. If `superpowers:brainstorming` is unavailable, stop and instruct the user to install the superpowers plugin.

## Artifact Directory Convention

```
docs/project/
├── 00-source.md         # original vision archive (file input only)
├── 00-vision.md         # project constitution
├── 01-architecture.md   # system-level architecture + ADRs
├── 02-conventions.md    # global conventions (also feeds CLAUDE.md)
├── contract/            # project-level API contract (accumulates across features)
│   ├── openapi.yaml
│   ├── types.ts
│   └── CHANGELOG.md
└── backlog.md           # epic → feature backlog (living document)
```

---

## Phase 1 — Vision Constitution

1. Invoke `superpowers:brainstorming` for project-level disambiguation: what problem, for whom, why now; how success will be measured; what this project is explicitly NOT (project-level non-goals); hard constraints (budget, timeline, platform, compliance).
2. Write `00-vision.md`: Goals / Users / Success metrics / **Non-goals** / Constraints. This is the project's constitution — every future feature PRD must be answerable to it.
3. Present a summary.

**🚦 Gate 1: STOP. Await user approval of the vision.**

---

## Phase 2 — System Architecture & Conventions

1. Design the **system-level skeleton** in `01-architecture.md`: tech-stack selection as ADRs (Context / Options / Decision / Trade-offs), module boundaries and responsibilities, data ownership per module, deployment shape. Skeleton, not detail — how each module works inside is decided per-feature in that feature's Phase 2.
2. Write `02-conventions.md`: API style, the project-wide error-body shape, naming conventions, testing conventions, engineering invariants. Then **create or update the project's CLAUDE.md** so every future session inherits these conventions automatically (keep CLAUDE.md to invariants only; point to `docs/project/` for the rest).
3. Initialize `docs/project/contract/` with a skeleton `openapi.yaml` (shared components: error body, pagination, auth scheme — no endpoints yet; features add those as deltas).
4. Present: architecture summary, key ADRs, conventions digest.

**🚦 Gate 2: STOP. Await user approval of architecture and conventions.**

---

## Phase 3 — Backlog

1. Decompose the vision into **epic → feature**, two levels, no further. A feature is a slice that can be independently delivered and accepted through one `/trivium:feature` run. If a feature looks like it would need more than ~20 tasks, split it now.
2. Route every entry. Use the relationship-to-defined-behavior rule:
   - introduces new observable behavior → `route: feature`
   - behavior preserved, structure changed → `route: refactor`
   - delivered code deviates from defined behavior → `route: bugfix`
   (A fresh project's backlog is naturally almost all `feature`; the other routes accumulate as the project lives.)
3. **The first entry is mandatory: a walking skeleton** — the thinnest slice that cuts through every architectural layer (one trivial endpoint + one trivial page + database + CI/CD to a deployable environment). Near-zero business value, maximum architecture-validation value. No other feature starts before the skeleton is accepted.
4. Entry template:

```markdown
### F-001: Walking skeleton  [P0] [route: feature] [status: ⬜]
Description: One trivial end-to-end slice through all layers, deployed.
Dependencies: none
Run: /trivium:feature backlog#F-001
---
### F-002: <feature name>  [P1] [route: feature] [status: ⬜]
Description: <one paragraph, user-observable outcome>
Dependencies: F-001
Run: /trivium:feature backlog#F-002
```

   Status legend: ⬜ pending / 🔧 in-progress / ✅ done / ⏸ deferred. The `Run:` line is a complete, copy-pasteable command.
5. Order by priority and dependency; mark the first milestone's scope.
6. Present: epic overview, feature count per epic, first-milestone contents, the walking-skeleton definition.

**🚦 Gate 3: STOP. Await user approval of the backlog and first milestone.**

---

## After inception — the long-running loop

```
pick next ⬜ entry from backlog (re-triage priorities as you go)
  → run its Run: command  (/trivium:feature | refactor | bugfix)
  → entry status written back on completion
  → repeat
```

Route labels are **routing suggestions frozen at triage time, re-checked at execution time**: the entry commands verify the route still matches reality (a "feature" may have partially materialized since triage and now be a refactor) and redirect instead of forcing the wrong pipeline. The backlog is a living document — completed features feed new entries (defect classes from /bugfix hardening, acceptance-report leftovers, refactoring ideas), all with route labels, keeping the list directly executable.
