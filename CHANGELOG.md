# Changelog

## 1.0.0 — 2026-07-09

Initial release.

- **Four commands**: `/trivium:inception` (project map: vision → architecture & conventions → epic/feature backlog with route labels and a mandatory walking-skeleton first entry), plus the three roads `/trivium:feature`, `/trivium:refactor`, `/trivium:bugfix`
- Hard human-approval gates with stop-and-wait semantics
- Cross-session resume via on-disk artifacts (`docs/pipeline/<slug>/`, `docs/project/`, `status` frontmatter)
- **Brownfield discipline** — building into an existing codebase is treated as the norm, not an edge case:
  - `/trivium:feature` **Phase 0.5 Context Loading**: reads `docs/project/` (if present) then does incremental, feature-scoped codebase archaeology into `01-context.md`; archaeology highlights (modules, reusable components, related tables) surface at Gate 1 for a sanity check
  - **Reuse-first ADRs**: every "build something new" decision (table / module / service / component) must justify why the existing one cannot be extended
  - **Migration discipline**: schema changes to a populated system ship as migration scripts with default/backfill strategy, nullable-transition and index/lock-table risk flagged, and a rollback plan — not just a schema diagram
  - **Green-baseline invariant + two-sided acceptance**: the full suite must be green before implementation, and Phase 5 runs full regression *and* PRD-traced Playwright acceptance (a feature must meet its PRD without breaking delivered behavior)
  - `/trivium:inception` **brownfield mode**: architecture and conventions are extracted from the code rather than designed, the master contract is reverse-generated from existing endpoints (scoped to shared components + core endpoints, the long tail backfilled on demand), and the walking skeleton is skipped
- Backlog integration: `backlog#<ID>` references with execution-time route re-check and automatic status write-back
- Project-context awareness: features inherit vision/architecture/conventions; per-feature contract deltas merge into the project master contract; ~20-task size guard triggers feature splitting
- File input: .md / .txt / .docx / .doc / .pdf, plus .xlsx / .xls / .csv batch triage mode for /bugfix; filenames with spaces are tolerated during argument parsing
- Skills: `prd-writing`, `api-contract`, `acceptance`
- Orchestrates superpowers: brainstorming, writing-plans, using-git-worktrees, subagent-driven-development, systematic-debugging, requesting-code-review, finishing-a-development-branch
