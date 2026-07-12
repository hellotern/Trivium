---
name: design-system
description: Establish and freeze a project-wide visual design system (color, typography, spacing, radius, component appearance, motion personality) so every feature composes screens from one shared visual language instead of inventing its own. Use whenever design language, visual consistency, UI look-and-feel, a design.md, a Stitch/Figma design system, or per-screen UI generation comes up, whenever the trivium /inception or /feature pipeline reaches its design phase, and whenever screens will be built by separate subagents that must look like one product.
---

# Design System

Produce one machine-appliable **visual** design system so screens built by different features (and different subagents) look like a single product. Core idea, mirroring the API contract: **the design system is the visual truth** — every screen composes from it, no screen redefines it, and consistency is structural, not a matter of taste or luck.

## The Iron Boundary — visual only, business-agnostic

The design system describes **how things look and move**, never **what the product does**. It is the vocabulary; feature screens are the sentences.

**Litmus test for every line of `design.md`:** *Could this rule be copied verbatim into a completely unrelated product (a bank app, a game, a CMS) and still make sense?* If it names a business noun — user, order, invoice, checkout, dashboard-for-X, a specific screen or flow — it does **not** belong in the design system. It belongs in a feature's `02-screens/`.

| Belongs in the design system (visual) | Belongs in a feature's screens (business) |
|---|---|
| Color palette + semantic roles (primary / danger / success / muted) | *Which* button on the checkout page is primary |
| Type families, scale, weights, line-heights | The heading copy on the orders page |
| Spacing & sizing scale, radius, elevation, stroke | This screen's specific layout of cards and forms |
| What a Button / Input / Card / Modal / Toast **looks like** in every state (default / hover / focus / active / disabled / loading / error) | Which screens use a Modal and what it says |
| Motion personality: duration scale, easing curves, character (snappy vs. gentle), enter/exit patterns | The navigation flow between this feature's screens |
| Iconography & imagery style, grid, breakpoints, light/dark theming, focus-ring & contrast rules | The data each screen displays |

Semantic role names (`primary`, `danger`) are visual and allowed. Business role names (`buy-now-button`, `invoice-card`) are not.

## Deliverables

**Project level** (created once by `/inception`, at `docs/project/design/`):

```
docs/project/design/
├── design.md          # the visual language — business-agnostic, the sole authority
├── design-system.json # Stitch/Figma design-system id + reference (when a design MCP is used)
└── CHANGELOG.md        # visual-language change history (needed once frozen)
```

**Feature level** (created per feature by `/feature` Phase 3, at `docs/pipeline/<slug>/02-screens/`):

```
02-screens/
├── screens.md         # screen inventory + flow; references design.md tokens/components by name
└── <screen>.prompt.md # one Stitch prompt per screen: composition + copy, NOT new visual rules
```

## `design.md` sections (all visual)

1. **Foundations** — color palette + semantic roles, typography (families / scale / weight / line-height), spacing & sizing scale, radius, elevation/shadow, stroke/border, z-index scale.
2. **Component appearance** — for each primitive (Button, Input/Select, Card, Modal/Dialog, Toast, Table, Tabs, Nav, …): what it looks like in every state. Appearance only — no data, no business labels.
3. **Motion personality** — duration scale, easing curves, and the character they express; standard enter/exit/hover/press patterns.
4. **Iconography & imagery** — icon style/weight/grid, illustration/photo treatment.
5. **Layout system** — column grid, breakpoints, container widths, density.
6. **Theming** — light/dark tokens; how roles remap.
7. **Accessibility (visual)** — contrast minimums, focus-ring spec, hit-target sizes, motion-reduced fallback.

Start with the `frontend-design` skill to set an intentional aesthetic direction — avoid templated defaults — *before* filling these sections.

## Stitch / design-MCP mechanics

- **Project (inception):** turn `design.md` into a registered design system — `create_design_system_from_design_md` (or `upload_design_md` → `create_design_system`). Store the returned id in `design-system.json`. This is the object every feature will apply.
- **Feature (per screen):** `generate_screen_from_text` with `<screen>.prompt.md`, then **`apply_design_system`** with the project's id so the screen inherits the frozen visual language. A screen prompt describes composition and copy; it must not restate or override tokens.
- No design MCP available? `design.md` is still the authority; screens are built to it by hand, and `apply_design_system` becomes a manual review against `design.md`.

## Freeze & change process (mirrors the API contract)

- `design.md` **freezes** when its gate passes (inception Gate 2, or a feature's Gate 3 for a delta). After the freeze, features may only **compose** from it.
- A feature that finds it genuinely needs a **new visual primitive** (a new component appearance, a new token) does **not** invent it locally. It raises a change request against `design.md` — reason, the proposed addition, why an existing primitive can't be restyled to fit — and on approval the delta is **merged into the master `design.md`**, the design system is updated, and the change is recorded in `CHANGELOG.md`. Breaking visual changes (recoloring a role, resizing the scale) are marked `[BREAKING]`.
- **Brownfield:** do not invent a system. **Reverse-extract** the de-facto tokens and component styles from the existing CSS/components into `design.md`, exactly as the contract is reverse-generated from existing endpoints; seed genuine inconsistencies as `route: refactor` cleanups rather than blessing them.

## Red flags — you are leaking business into the design system

- Writing a screen name, a data field, or a user flow into `design.md`
- A component entry that says what it *contains* rather than what it *looks like*
- A token named after a business concept (`order-total-color`) instead of a role (`price-emphasis` → still borderline; prefer `text-strong`)
- A feature's screen prompt that sets a hex value, font size, or radius instead of naming a design-system token
- A feature adding a new color/component without a change request against `design.md`

**All of these mean: move it. Visual rules live in `design.md`; business composition lives in the feature's `02-screens/`.**
