# Trivium

> **Three roads, gated by you.** Spec-driven development pipelines for Claude Code.

*Trivium* (Latin: "where three roads meet") gives Claude Code three disciplined entry points for the three situations all software work falls into — building something new, changing something that exists, and fixing something broken — each guarded by hard human-approval gates.

[中文文档 / Chinese README](./README.zh-CN.md)

## The three roads

| Command | For | Pipeline |
|---|---|---|
| `/trivium:feature` | New requirements | brownfield codebase archaeology → disambiguation → PRD → architecture & frozen API contract (reuse-first ADRs + migration/rollback plan) → UI design (design-system screens, visually signed off) → task plan → subagent implementation (TDD + two-stage review) → full regression + Playwright acceptance |
| `/trivium:refactor` | Changes & refactoring | impact analysis → behavior-pinning tests → proposal with ADRs → plan → implementation with a green baseline invariant → regression |
| `/trivium:bugfix` | Defects | failing reproduction test **first** → systematic root-cause analysis → minimal fix → red-to-green + full regression. Accepts an Excel/CSV bug list for batch triage mode. |

## The map: `/trivium:inception`

For a **new project**, one `/feature` run cannot swallow the whole vision — and pre-generating hundreds of tasks is planning that decays before it runs. `/trivium:inception` draws the map instead (run once per project):

1. **Vision constitution** (`docs/project/00-vision.md`): goals, users, success metrics, project-level non-goals. 🚦
2. **System architecture, conventions & design system** (`01-architecture.md`, `02-conventions.md` → feeds CLAUDE.md; initializes the project-level API contract *and* the project-level **visual design system** in `docs/project/design/` — the business-agnostic color/type/spacing/component/motion language every feature composes from). 🚦
3. **Backlog** (`backlog.md`): epic → feature, **stopping at feature grain** — tasks are generated just-in-time inside each `/feature` run. Every entry carries a `route:` label (feature / refactor / bugfix, judged by its relationship to defined behavior) and a copy-pasteable `Run:` command. The first entry is always a **walking skeleton**: the thinnest slice through every architectural layer, validating the architecture before anything else is built. 🚦

**Adopting an existing codebase?** `/trivium:inception` also runs in **brownfield mode**: architecture and conventions are *extracted from the code* (not invented), the master contract is reverse-generated from existing endpoints, the walking skeleton is skipped, and the backlog holds only future work — you still confirm the vision and sign the same three gates.

The long-running loop is then simply: pick the next backlog entry → paste its `Run:` line → sign gates → status writes back → repeat. Routes are re-checked at execution time (labels decay as the codebase evolves), features whose plans exceed ~20 tasks get split instead of swallowed, and each feature's contract merges as a delta into the project master contract.

## The gates

Your role collapses to signing off at gates. Between gates, the pipeline runs itself.

| | Gate 1 | Gate 2 | Gate 3 | Gate 4 | Gate 5 |
|---|---|---|---|---|---|
| inception | approve vision | approve architecture + conventions + design system | approve backlog + first milestone | — | — |
| feature | approve PRD | approve architecture + contract | **approve UI** | approve plan | final acceptance |
| refactor | confirm impact + behavior classification | approve proposal | approve plan | final regression | — |
| bugfix (single) | confirm reproduction | — | — | final sign-off | — |
| bugfix (batch) | triage: scope & order | per-bug reproduction | — | per-bug or batch sign-off | — |

The **UI gate** (feature Gate 3) is where the design MCP (e.g. Stitch) renders the actual screens from the project design system and you sign off on the look *before any page is implemented* — so screens across features stay one product instead of drifting apart.

## What Trivium adds on top of superpowers

Trivium **orchestrates** [superpowers](https://github.com/obra/superpowers) rather than reimplementing it. Superpowers provides the methodology (brainstorming, writing-plans, subagent-driven development, TDD, systematic debugging, code review); Trivium adds what a spec-driven pipeline needs and superpowers doesn't have:

- **`prd-writing`** — a PRD format with priorities, testable Given/When/Then acceptance criteria, and a mandatory out-of-scope list; the single source of truth for everything downstream.
- **`api-contract`** — frozen frontend/backend contracts (OpenAPI + generated TypeScript types + mocks) so FE and BE tasks can be built in parallel by subagents that never talk to each other, with a formal change process when the contract must move.
- **`design-system`** — a project-wide **visual** language (color, typography, spacing, radius, component appearance, motion personality) frozen at inception and *composed* — never reinvented — by each feature's screens, so UI built by separate subagents looks like one product. Strictly business-agnostic: the look lives here, the screens live in each feature. The visual counterpart of `api-contract`.
- **`acceptance`** — Playwright end-to-end specs traced one-to-one to PRD criteria, with a traceability matrix and an evidence-based acceptance report.

Plus the pipeline mechanics: hard stop-and-wait gates, all artifacts written to `docs/pipeline/<slug>/` with `status` frontmatter for **cross-session resume**, a contract-change loop that returns to the gate instead of silently diverging, and a strict boundary between in-development rework and post-delivery bugfixing.

## Requirements

- **[superpowers](https://github.com/obra/superpowers)** installed (Trivium invokes its brainstorming / writing-plans / using-git-worktrees / subagent-driven-development / systematic-debugging / requesting-code-review / finishing-a-development-branch skills)
- `@playwright/test` runnable in the target project (acceptance phase)
- Recommended: `openapi-typescript` (type generation); MSW or `@stoplight/prism-cli` (mocks)
- For file input: `pandoc` (docx), `poppler-utils` (pdf), `libreoffice` (legacy .doc); Python fallbacks: `pip install python-docx pdfplumber pandas openpyxl`

## Works great with

Trivium's structural steps — codebase archaeology, impact / blast-radius analysis, root-cause tracing — automatically use a **code-intelligence MCP or LSP** when one is installed, preferring graph queries and call tracing over text search for "who calls this?" and "what breaks if I change it?". This is capability-detected and **entirely optional**: with no such tool present, Trivium falls back to subagent exploration. (Only [superpowers](https://github.com/obra/superpowers) is a hard dependency — it is methodology, not a tool.)

- **GitNexus** — a code-graph MCP with impact / blast-radius queries, plus a PostToolUse hook that rebuilds its index after each commit so blast-radius answers stay fresh. Its open-source edition is licensed **PolyForm Noncommercial**; commercial use requires an enterprise license — confirm this fits before relying on it in a for-profit setting.

Two rules whenever you lean on a code-graph tool: confirm the **index is fresh** before trusting a query (a stale graph is worse than none), and verify the tool's **license** covers your use.

## Install

From the official plugin directory (once listed):

```
/plugin install trivium
```

Or directly from this repository:

```
/plugin marketplace add hellotern/Trivium
/plugin install trivium@trivium
```

## Usage

Text and file inputs, mixable:

```
/trivium:inception docs/vision.md            # new project: draw the map first
/trivium:feature backlog#F-001               # then consume the backlog entry by entry
/trivium:feature Users can register and log in with email, with remember-me
/trivium:feature docs/specs/membership.docx  Note: email only this cycle, no phone signup
/trivium:refactor Consolidate the order module's scattered useState into a state machine
/trivium:refactor backlog#T-004
/trivium:bugfix Form submission intermittently returns 500; logs show duplicate key
/trivium:bugfix qa/uat-defect-list.xlsx
```

File contents are extracted and archived verbatim to `docs/pipeline/<slug>/00-source.md` as the authoritative requirement source. A spreadsheet triggers `/trivium:bugfix`'s **batch mode**: parse → root-cause grouping → triage gate (you pick scope and order) → per-bug reproduce-diagnose-fix cycles → status written back to the inventory → batch report.

## When NOT to use Trivium

These pipelines are token-intensive by design. Skip them for one-file tweaks, quick prototypes, and throwaway scripts — just talk to Claude. Reserve the roads for work that earns its process tax: features you'll maintain, cross-stack requirements, refactors with regression risk, and defects in delivered code.

## License

MIT
