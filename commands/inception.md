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

## Greenfield or brownfield

Detect the starting point before Phase 1: an empty or near-empty repo is **greenfield** — design forward, exactly as the phases below describe. A repo that already holds a real codebase is **brownfield**, and inception switches to **archaeology mode** so an existing project can adopt Trivium and still gain project-context awareness and an accumulating contract. Only the *source of truth* changes; the four Gates do not:

- **Vision (Phase 1)** is still elicited from you and confirmed at Gate 1 — direction is a human decision, never reverse-engineered from code.
- **Architecture & conventions (Phase 2)** are **extracted from the code, not invented**: derive the real module boundaries, data ownership, stack, and de-facto conventions, and record them as the starting `01-architecture.md` / `02-conventions.md`. Prefer a code-intelligence MCP or LSP for module boundaries and call graphs when one is available and freshly indexed; fall back to subagent exploration otherwise. Where the code diverges from what it *should* be, note the gap and seed it as a `route: refactor` backlog entry rather than silently blessing it.
- **Contract** is **reverse-generated** from the existing endpoints into `docs/project/contract/` as the project master contract, instead of starting from an empty skeleton. **Scoped, not exhaustive** — reverse-engineering hundreds of endpoints up front is wasted work, since most are never touched by a new feature: capture only the **shared components** (error body, pagination, auth scheme) plus the **core high-frequency endpoints**, and let the long tail be backfilled on demand — each remaining endpoint is added to the master contract the first time a feature touches it, in that feature's Phase 2.
- **Design system (Phase 3)** is **reverse-extracted** from the existing UI into `docs/project/design/` — the de-facto visual language (color, typography, spacing, radius, component appearance, motion) — instead of invented. Genuine visual inconsistencies are seeded as `route: refactor` entries, not blessed.
- **Backlog (Phase 4)** holds only **future** work; the mandatory walking-skeleton entry is **skipped** when a running system already exists (the skeleton it would validate is already in production).

## Phase 0 — Input Parsing (automatic, no gate)

Same rules as the other trivium commands: `$ARGUMENTS` may be vision text, file paths (.md / .txt / .docx / .doc / .pdf — extract via pandoc / python-docx / libreoffice / pdftotext / pdfplumber; stop on failed extraction and request a text version), or both. When matching file paths, **tolerate spaces in filenames**: if a path-like token does not resolve on its own, greedily test whether it joined with the following token(s) by single spaces forms a path that does exist, and treat that whole span as one file before classifying the rest as vision text / notes. Archive extracted content verbatim to `docs/project/00-source.md`. Supplementary notes win on conflict, confirmed during disambiguation.

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
├── design/              # project-level visual design system (accumulates across features)
│   ├── design.md
│   ├── design-system.json
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
3. Initialize `docs/project/contract/` with a skeleton `openapi.yaml` (shared components: error body, pagination, auth scheme — no endpoints yet; features add those as deltas). **Brownfield**: reverse-generate the master contract from the existing endpoints instead of starting empty (see *Greenfield or brownfield*).
4. Present: architecture summary, key ADRs, conventions digest, contract skeleton (shared components).

**🚦 Gate 2: STOP. Await user approval of architecture and conventions.**

---

## Phase 3 — Design System

**Only when the project has a UI surface** (web / mobile / desktop front-end). For a headless API, CLI, or library there is nothing to design visually: **state explicitly that this phase is skipped and why**, then go straight to Phase 4 — do not invent a design system for a product that has no screens. Do not silently pass over this phase; a UI project that reaches Phase 4 without a signed-off `design.md` has skipped a required gate.

1. Invoke the `trivium:design-system` skill to establish the project's **visual** design system into `docs/project/design/`: the business-agnostic visual language (color palette + semantic roles, typography, spacing & sizing scale, radius, elevation, component appearance in every state, motion personality). This is the visual counterpart of the contract — **skeleton, not screens**: it defines *how things look*, never *what any screen does*. Per-feature screens are composed later in each feature's own UI gate.
2. **Greenfield**: design the visual language forward — start from the `frontend-design` skill to set an intentional aesthetic direction *before* filling `design.md`, so the system is deliberate rather than a templated default. **Brownfield**: **reverse-extract** the de-facto design language (color, typography, spacing, radius, component styles, motion) from the existing UI into `design.md` instead of inventing one; seed genuine visual inconsistencies as `route: refactor` backlog entries rather than blessing them (see *Greenfield or brownfield*).
3. When a design MCP (Stitch / Figma) is available, turn `design.md` into a **registered design system** and store its id in `docs/project/design/design-system.json` — this is the object every feature will `apply_design_system` against. With no design MCP, `design.md` is still the sole visual authority; screens are later built to it by hand.
4. Present the **design-language digest** for sign-off: the palette + roles, type scale, spacing/radius, the component inventory and their states, and the motion character — enough for the user to see the product's look before any screen exists.

**🚦 Gate 3: STOP. Await user approval of the design system.** For a non-UI project, record the skip and its reason here instead, then continue.

---

## Phase 4 — Backlog

1. Decompose the vision into **epic → feature**, two levels, no further. A feature is a slice that can be independently delivered and accepted through one `/trivium:feature` run. If a feature looks like it would need more than ~20 tasks, split it now.
2. Route every entry. Use the relationship-to-defined-behavior rule:
   - introduces new observable behavior → `route: feature`
   - behavior preserved, structure changed → `route: refactor`
   - delivered code deviates from defined behavior → `route: bugfix`
   (A fresh project's backlog is naturally almost all `feature`; the other routes accumulate as the project lives.)
3. **The first entry is mandatory (greenfield): a walking skeleton** — the thinnest slice that cuts through every architectural layer (one trivial endpoint + one trivial page *rendered with the design system, when the project has UI* + database + CI/CD to a deployable environment). Near-zero business value, maximum architecture- (and, for UI projects, design-system-) validation value. No other feature starts before the skeleton is accepted. **Brownfield**: skip the skeleton — a running system has already proven its layers wire together — and fill the backlog with future work only.
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

**🚦 Gate 4: STOP. Await user approval of the backlog and first milestone.**

---

## After inception — the long-running loop

```
pick next ⬜ entry from backlog (re-triage priorities as you go)
  → run its Run: command  (/trivium:feature | refactor | bugfix)
  → entry status written back on completion
  → repeat
```

Route labels are **routing suggestions frozen at triage time, re-checked at execution time**: the entry commands verify the route still matches reality (a "feature" may have partially materialized since triage and now be a refactor) and redirect instead of forcing the wrong pipeline. The backlog is a living document — completed features feed new entries (defect classes from /bugfix hardening, acceptance-report leftovers, refactoring ideas), all with route labels, keeping the list directly executable.
