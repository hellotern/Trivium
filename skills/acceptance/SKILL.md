---
name: acceptance
description: Generate and run Playwright end-to-end acceptance tests traced one-to-one to PRD acceptance criteria, producing a traceability matrix and acceptance report. Use whenever the trivium /feature pipeline reaches its acceptance phase, whenever the user asks for acceptance testing, E2E tests, Playwright verification, or wants proof that delivered features match the PRD — including regression acceptance after /refactor.
---

# Acceptance

Turn PRD acceptance criteria into executable Playwright end-to-end specs, judging delivery by **evidence rather than claims**. Core principle: **one-to-one traceability between specs and the PRD** — no P0 criterion without a spec, and no spec that cannot name its criterion.

## Deliverables

Written to `docs/pipeline/<slug>/04-acceptance/`:

```
04-acceptance/
├── traceability.md      # matrix: PRD criterion ↔ spec ↔ result
├── report.md            # acceptance report (with failure analysis)
└── (test code lives in the project's test tree, e.g. e2e/ or tests/e2e/, per project convention)
```

## Process

### 1. Build the traceability matrix (before writing any code)

Read `01-prd.md` and register every Given/When/Then criterion:

```markdown
| PRD criterion | Priority | Level | Spec | Status |
|---|---|---|---|---|
| F1-AC1: Given logged out, When visiting /dashboard, Then redirected to login | P0 | E2E | e2e/auth.spec.ts::redirects-guest | ⬜ |
| F2-AC1: Given ..., Then returns 422 | P0 | API | covered by BE contract tests | ✅ref |
```

**Level judgment**: not every criterion deserves E2E. Pure-backend criteria already covered by unit/contract tests are referenced in the matrix instead — avoiding duplicated, brittle E2E. Reserve E2E for **user-visible critical paths**: P0 main flows, full cross-stack journeys, and user-perceivable parts of the PRD's non-functional requirements. P0 criteria require full coverage; P1 covers happy paths; P2 is out of this cycle's matrix.

### 2. Write the specs

- Use `@playwright/test` (the test framework), not a Playwright MCP — MCPs suit exploratory manual verification; acceptance needs **repeatable, evidence-preserving** test code.
- Structure mirrors the PRD: `test.describe` per feature (F1/F2); test titles cite the criterion id (e.g. `'F1-AC1: redirects guest to login'`) so reports trace backwards.
- Selector priority: `getByRole` / `getByLabel` / `getByTestId` > CSS selectors; never depend on volatile style classes.
- Self-managed test data: each spec owns its preconditions (API seeding or fixtures); ordering dependencies between specs are forbidden.
- Assert what the PRD says is observable, not implementation details (assert "the page shows the order number", not "the store contains an order object").

### 3. Execute & report

1. Run the full set, preserving Playwright traces/screenshots as evidence (every failure must have a trace).
2. Update matrix statuses: ✅ pass / ❌ fail / ⬜ uncovered (an uncovered P0 = acceptance FAILED).
3. Produce `report.md`:

```markdown
# Acceptance Report: <feature-slug>
Date / commit / environment

## Verdict: PASS | FAIL

## Matrix summary
P0: x/x passed; P1: x/x passed; referenced existing tests: n

## Failure analysis (if any)
| Spec | Symptom | Attribution | Disposition |
|---|---|---|---|
| F1-AC2 | ... | implementation defect / contract breach / PRD ambiguity / test defect | back to Task-N / back to gate for clarification |

## Open items & recommendations
```

### 4. Failure-handling principles

- **Failures inside an active pipeline** (/feature Phase 6, /refactor Phase 5): return to the owning task's context, fix, re-run. This is rework, not /bugfix.
- Failures attributed to **PRD ambiguity**: silently bending the test or the implementation to your own interpretation is forbidden — return to the gate, clarify with the user, amend the PRD, then update the matrix.
- **Flaky specs**: in an acceptance report, flaky = failed. Fix determinism first (explicit wait conditions, controlled data); masking with retries is forbidden.
- The bar for passing is **a fully green run in one execution**, not "the failures passed when re-run individually".
