# Trivium

> **Three roads, four gates.** Spec-driven development pipelines for Claude Code.

*Trivium* (Latin: "where three roads meet") gives Claude Code three disciplined entry points for the three situations all software work falls into — building something new, changing something that exists, and fixing something broken — each guarded by hard human-approval gates.

[中文文档 / Chinese README](./README.zh-CN.md)

## The three roads

| Command | For | Pipeline |
|---|---|---|
| `/trivium:feature` | New requirements | disambiguation → PRD → architecture & frozen API contract → task plan → subagent implementation (TDD + two-stage review) → Playwright acceptance |
| `/trivium:refactor` | Changes & refactoring | impact analysis → behavior-pinning tests → proposal with ADRs → plan → implementation with a green baseline invariant → regression |
| `/trivium:bugfix` | Defects | failing reproduction test **first** → systematic root-cause analysis → minimal fix → red-to-green + full regression. Accepts an Excel/CSV bug list for batch triage mode. |

## The map: `/trivium:inception`

For a **new project**, one `/feature` run cannot swallow the whole vision — and pre-generating hundreds of tasks is planning that decays before it runs. `/trivium:inception` draws the map instead (run once per project):

1. **Vision constitution** (`docs/project/00-vision.md`): goals, users, success metrics, project-level non-goals. 🚦
2. **System architecture & conventions** (`01-architecture.md`, `02-conventions.md` → feeds CLAUDE.md; initializes the project-level API contract). 🚦
3. **Backlog** (`backlog.md`): epic → feature, **stopping at feature grain** — tasks are generated just-in-time inside each `/feature` run. Every entry carries a `route:` label (feature / refactor / bugfix, judged by its relationship to defined behavior) and a copy-pasteable `Run:` command. The first entry is always a **walking skeleton**: the thinnest slice through every architectural layer, validating the architecture before anything else is built. 🚦

The long-running loop is then simply: pick the next backlog entry → paste its `Run:` line → sign gates → status writes back → repeat. Routes are re-checked at execution time (labels decay as the codebase evolves), features whose plans exceed ~20 tasks get split instead of swallowed, and each feature's contract merges as a delta into the project master contract.

## The four gates

Your role collapses to signing off at gates. Between gates, the pipeline runs itself.

| | Gate 1 | Gate 2 | Gate 3 | Gate 4 |
|---|---|---|---|---|
| inception | approve vision | approve architecture + conventions | approve backlog + first milestone | — |
| feature | approve PRD | approve architecture + contract | approve plan | final acceptance |
| refactor | confirm impact + behavior classification | approve proposal | approve plan | final regression |
| bugfix (single) | confirm reproduction | — | — | final sign-off |
| bugfix (batch) | triage: scope & order | per-bug reproduction | — | per-bug or batch sign-off |

## What Trivium adds on top of superpowers

Trivium **orchestrates** [superpowers](https://github.com/obra/superpowers) rather than reimplementing it. Superpowers provides the methodology (brainstorming, writing-plans, subagent-driven development, TDD, systematic debugging, code review); Trivium adds what a spec-driven pipeline needs and superpowers doesn't have:

- **`prd-writing`** — a PRD format with priorities, testable Given/When/Then acceptance criteria, and a mandatory out-of-scope list; the single source of truth for everything downstream.
- **`api-contract`** — frozen frontend/backend contracts (OpenAPI + generated TypeScript types + mocks) so FE and BE tasks can be built in parallel by subagents that never talk to each other, with a formal change process when the contract must move.
- **`acceptance`** — Playwright end-to-end specs traced one-to-one to PRD criteria, with a traceability matrix and an evidence-based acceptance report.

Plus the pipeline mechanics: hard stop-and-wait gates, all artifacts written to `docs/pipeline/<slug>/` with `status` frontmatter for **cross-session resume**, a contract-change loop that returns to the gate instead of silently diverging, and a strict boundary between in-development rework and post-delivery bugfixing.

## Requirements

- **[superpowers](https://github.com/obra/superpowers)** installed (Trivium invokes its brainstorming / writing-plans / using-git-worktrees / subagent-driven-development / systematic-debugging / requesting-code-review / finishing-a-development-branch skills)
- `@playwright/test` runnable in the target project (acceptance phase)
- Recommended: `openapi-typescript` (type generation); MSW or `@stoplight/prism-cli` (mocks)
- For file input: `pandoc` (docx), `poppler-utils` (pdf), `libreoffice` (legacy .doc); Python fallbacks: `pip install python-docx pdfplumber pandas openpyxl`

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
