---
name: api-contract
description: Design and freeze frontend-backend API contracts as an OpenAPI spec + generated TypeScript types + mock setup, enabling parallel FE/BE development by non-communicating subagents. Use whenever API design, interface definition, frontend/backend split, parallel FE/BE work, or contract changes come up, whenever the trivium /feature or /refactor pipeline reaches its architecture/contract phase, and whenever frontend and backend tasks will be built by separate subagents.
---

# API Contract

Produce machine-verifiable API contracts so frontend and backend tasks can be developed in parallel by subagents that never talk to each other. Core idea: **the contract is the truth** — the frontend trusts the mocks, the backend trusts the contract tests, and neither trusts verbal agreements.

## Deliverables (all three, no exceptions)

Written to `docs/pipeline/<slug>/02-contract/`:

```
02-contract/
├── openapi.yaml     # the single source of truth
├── types.ts         # TS types GENERATED from openapi.yaml, shared by FE and BE
├── mock.md          # mock plan (tooling, start command, fixture locations)
└── CHANGELOG.md     # contract change history (needed only after freezing)
```

## Design Rules

1. **openapi.yaml is the sole authority.** types.ts must be generated from it (e.g. `openapi-typescript`) — hand-writing types and "keeping them in sync" is forbidden; manual sync always decays. Reuse the project's existing type-generation chain if one exists.
2. Every endpoint is fully specified: request/response schemas (field types, required-ness, value constraints), **every error response that can actually occur** (each applicable one of 400/401/403/404/409/422/500, with a uniform error body), and explicit pagination/sorting/filtering conventions.
3. One error-body shape project-wide, e.g. `{ code: string, message: string, details?: object }`. `code` is a stable machine-readable enum; frontend logic branches on `code`, never on `message`.
4. Naming and style follow the project's existing API conventions (read existing code before designing). Greenfield defaults: REST, plural resource names, snake_case or camelCase — pick one, apply globally.
5. **KISS check**: every endpoint and every field must answer "which PRD feature needs this?" Anything without an answer gets deleted — fields reserved for imagined future needs are a YAGNI violation.
6. Give every non-trivial field a `description` and `example` — these directly feed mock data and are how subagents learn field semantics.

## Mock Plan

Selection order (simplest that works):

1. The project already has mock infrastructure → reuse it.
2. Frontend project → prefer [MSW](https://mswjs.io/) (interception-layer mocks; no business-code changes; removed wholesale at integration).
3. A standalone mock server is needed → Prism (`@stoplight/prism-cli`) serves mocks straight from openapi.yaml with zero hand-written code.

mock.md must state: the start command, which endpoints are mocked, fixture-quality requirements (no "string"/"test" placeholders — use the OpenAPI `example` values, semantically plausible), and **the exact steps the integration task takes to remove the mocks**.

## Freeze & Change Process

- The contract **freezes** when its gate (architecture approval) passes. After the freeze, FE/BE tasks are dispatched. If either side finds the contract unimplementable or defective:
  1. **Stop the affected task immediately** — unilaterally diverging from the contract "just to keep moving" is forbidden;
  2. Raise a change request: reason, diff, affected-task list, breaking or not;
  3. On user approval: update openapi.yaml → regenerate types.ts → record in CHANGELOG.md → update affected task descriptions → resume.
- Breaking changes (removing fields, changing types, changing required-ness, changing semantics) must be marked `[BREAKING]` in the CHANGELOG.

## Verification Duties on Both Sides

- **Backend tasks**: the definition of done includes contract tests — schema-validation tooling (e.g. jest + an OpenAPI validator, or the project's existing setup) verifying real responses match openapi.yaml field by field. Correct types with wrong semantics (e.g. pagination starting at 1 vs. 0) is still a breach; semantic conventions go into `description` fields and get tested.
- **Frontend tasks**: may only import types from `types.ts`; declaring duplicate interface types locally is forbidden. All requests go through a single client layer so integration can switch mock → real endpoint in one place.
- **Integration task**: remove mocks, point at the real backend, run the end-to-end happy paths. If the contract was done well, this step should be **boring to the point of disappointment** — if integration produces surprises, trace each to either a contract defect or a breaching implementation, and record the lesson in the CHANGELOG.
