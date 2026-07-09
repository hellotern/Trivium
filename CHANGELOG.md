# Changelog

## 1.0.0 — 2026-07-09

Initial release.

- Three entry commands: `/trivium:feature`, `/trivium:refactor`, `/trivium:bugfix`
- Hard human-approval gates with stop-and-wait semantics
- Cross-session resume via on-disk artifacts (`docs/pipeline/<slug>/`, `status` frontmatter)
- File input support: .md / .txt / .docx / .doc / .pdf, plus .xlsx / .xls / .csv batch triage mode for /bugfix
- Skills: `prd-writing`, `api-contract`, `acceptance`
- Orchestrates superpowers: brainstorming, writing-plans, using-git-worktrees, subagent-driven-development, systematic-debugging, requesting-code-review, finishing-a-development-branch
