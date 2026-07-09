---
name: prd-writing
description: Write structured PRDs (Product Requirements Documents) with priorities, testable Given/When/Then acceptance criteria, and a mandatory out-of-scope list. Use whenever the user asks to write, rewrite, or review a PRD or requirements document, whenever the trivium /feature pipeline reaches its PRD phase, or whenever raw requirements need converting into an implementable specification — even if the user only says "organize these requirements" or "turn this into a spec".
---

# PRD Writing

Convert disambiguated requirements into a PRD that is both **implementable and acceptance-testable**. A PRD has two audiences: the human who approves it, and the agents that implement it in later phases. It must let a human judge correctness at a glance AND let a machine execute it without ambiguity.

## Precondition

This skill owns the **output format**, not disambiguation. Requirement clarification (typically via superpowers:brainstorming) must be complete before use. If the input still contains unresolved ambiguity, go back to asking questions — do not fill gaps with assumptions. **Every silent assumption you make on the user's behalf is a future bug.** Genuinely irreducible uncertainty goes explicitly into the Open Questions section; never pretend it does not exist.

## Hard Rules

1. **Every P0/P1 feature must carry testable acceptance criteria** in Given/When/Then form. A feature you cannot write criteria for is a feature you have not understood — return to disambiguation.
2. **The Out-of-Scope section is mandatory and must be non-empty.** Inability to name a single exclusion usually means the scope never actually converged.
3. Acceptance criteria describe **observable behavior only** (what the user sees, what the system returns, how data changes) — never implementation ("uses Redis caching" belongs in architecture, not here).
4. Precise language: unmeasurable adjectives ("fast", "friendly", "stable") may not stand alone — quantify ("P95 < 300ms") or give a decidable definition.
5. Once approved, the PRD is the **single source of truth** for all downstream phases. If implementation reveals a PRD error, the PRD must be amended and re-gated; code silently diverging from the document is forbidden.

## PRD Template

```markdown
---
status: draft          # draft | approved
version: 1.0
date: YYYY-MM-DD
slug: <feature-slug>
---

# PRD: <Feature Name>

## 1. Background & Goals
<Why this exists; whose problem it solves. 1–2 paragraphs.>
**Success metric:** <How we will know post-launch that this worked; quantify where possible.>

## 2. Users & Scenarios
<Who uses this, in what context. List each user class separately if multiple.>

## 3. Functional Requirements

### F1: <Feature name>  [P0]
**Description:** <What it does.>
**Acceptance criteria:**
- Given <precondition>, When <user action / trigger>, Then <observable result>
- Given ..., When ..., Then ...

### F2: <Feature name>  [P1]
...

<Priority definitions: P0 = cannot ship without it; P1 = should ship this cycle, may be cut in extremis; P2 = explicit future enhancement, NOT built this cycle, listed only so architecture can leave room.>

## 4. Non-Functional Requirements
<Only real constraints; write "none" rather than boilerplate:>
- Performance: <e.g. P95 latency, concurrency>
- Security: <e.g. auth requirements, data sensitivity>
- Compatibility: <e.g. browser matrix, mobile, legacy data>

## 5. Out of Scope (explicitly not in this cycle)
- <Exclusion 1, with why / when it might happen>
- <Exclusion 2>

## 6. Dependencies & Assumptions
- Dependencies: <external services, other teams, prerequisite features>
- Assumptions: <preconditions that must hold for this PRD to be implementable as written>

## 7. Open Questions
- <Undecided items needing resolution in architecture or with the user. Write "none" if empty.>
```

## Pre-Gate Self-Check

Verify before submitting to the gate:

- [ ] Every P0/P1 feature has at least one Given/When/Then criterion
- [ ] All criteria are observable behavior; zero implementation leakage
- [ ] Out-of-Scope is non-empty
- [ ] No unmeasurable adjectives standing alone
- [ ] Open Questions hides no P0-level ambiguity that belonged in disambiguation
- [ ] The user-facing summary contains: feature list + priorities, out-of-scope, open questions — the three things an approver most needs to see
