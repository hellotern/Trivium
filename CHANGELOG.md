# Changelog

## 1.0.0 — 2026-07-09

Initial release.

- **Four commands**: `/trivium:inception` (project map: vision → architecture & conventions → epic/feature backlog with route labels and a mandatory walking-skeleton first entry), plus the three roads `/trivium:feature`, `/trivium:refactor`, `/trivium:bugfix`
- Hard human-approval gates with stop-and-wait semantics
- Cross-session resume via on-disk artifacts (`docs/pipeline/<slug>/`, `docs/project/`, `status` frontmatter)
- Backlog integration: `backlog#<ID>` references with execution-time route re-check and automatic status write-back
- Project-context awareness: features inherit vision/architecture/conventions; per-feature contract deltas merge into the project master contract; ~20-task size guard triggers feature splitting
- File input: .md / .txt / .docx / .doc / .pdf, plus .xlsx / .xls / .csv batch triage mode for /bugfix
- Skills: `prd-writing`, `api-contract`, `acceptance`
- Orchestrates superpowers: brainstorming, writing-plans, using-git-worktrees, subagent-driven-development, systematic-debugging, requesting-code-review, finishing-a-development-branch
