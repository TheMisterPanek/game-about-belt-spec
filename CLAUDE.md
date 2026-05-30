# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A **specification-only** design for a macro+micro factory game, written to double as a
**language- and engine-neutral AI benchmark**. There is no source code here and — by
rule — there must not be. The specs are the product.

Read in order: `CONSTITUTION.md` → `GLOSSARY.md` → `specs/architecture/00..04` →
`specs/features/Fxx/` → `benchmark/`. To build *from* the specs, follow `ROADMAP.md`
(vertical-slice milestones M0–M8) and track status in a per-run copy of `PROGRESS.md`;
"Done" vs "Verified" is defined in `specs/README.md` → Definition of Done.

## Hard rules (from CONSTITUTION.md — do not violate)

- **No implementation code.** Not in any language, not "just an example." If a task
  seems to call for code, it is actually a request to *amend a spec* (Art. II).
- **No language/engine/library/rendering-API references** in any spec. Express data in
  the neutral pseudo-notation of `specs/architecture/02-data-model.md` (Art. III).
  Forbidden: concrete types, engines (Unity/Godot/Bevy), language idioms.
- **Determinism & fixed-point.** Anything that affects simulation state is integer or
  `Fixed` (numerator over the global scale **1/1024**); never `Real`. Reals are allowed
  only for presentation (camera/interpolation) and must never feed a system (Art. IV).
- **Single-resource logistics (Art. VII).** A belt or road carries exactly one resource
  type at a time. This is invariant **I1**, not a tunable.
- **MVP scope (Art. VIII).** MVP terrain is exactly `Empty/Stone/Iron/Copper`; new
  resources go through the catalog (F02), never hard-coded elsewhere. F10 is Post-MVP.

## The one concept to understand first

**Two-tier LOD simulation** (`specs/architecture/03-simulation-lod.md`). The active
cell is simulated fine-grained (per belt/item/machine); every other cell is reduced to
a `ProductionProfile` and advanced by `rate × Δt`. Entering/leaving a cell
**promotes/demotes** it, and that transition must be **lossless at steady state**
(property **P-LOD**). This is the project's signature challenge and the highest-weighted
benchmark axis — most specs ultimately serve it. The micro→macro bridge is the profile
(`F05 S04`); the LOD machinery is `F01 S04–S06`.

## Structure & ownership

- `specs/architecture/` — cross-cutting: ECS model (abstract), neutral data model, the
  LOD/simulation contract, module boundaries, and the testing strategy
  (`05-testing-strategy.md` — step-the-headless-core + assert-invariants-per-tick,
  property & golden-master/differential testing, the P-LOD recipe). Changes here ripple
  everywhere.
- `specs/features/Fxx-<slug>/` — one feature per folder: a `feature.md` (intent, scope,
  story index, feature-level acceptance `Fxx-Ay`) and stories `Syy-*.md`.
- Modules map 1:1 to features (`specs/architecture/04-module-boundaries.md`): `catalog`
  F02, `world` F03, `microfactory` F05/F06, `routing` F07, `macromfg` F08, `blueprint`
  F09, `sim` F01, `interaction` F04, `persist` F10.
- `benchmark/` — the rubric: harness protocol, scoring axes/weights, reference scenarios.

## Conventions when editing specs

- **Vocabulary is fixed.** Use only `GLOSSARY.md` terms with their exact meanings; to
  introduce a term, define it in the glossary first.
- **IDs are immutable.** Feature ids (`Fxx`), story ids (`Syy`), acceptance ids
  (`Fxx-Ay`, `AC-n`) are referenced by the benchmark — never renumber or reuse;
  deprecate instead.
- **Stories follow the template** in `specs/README.md`: Intent → Behavior → Data touched
  → Acceptance criteria → Out of scope. Every acceptance criterion must be checkable
  from observable state/dump output, not from reading source (Art. IX). Split a story if
  it exceeds ~7 ACs or spans two modules.
- **Command vs system discipline** (`04-module-boundaries.md` §3): player actions are
  *commands* applied between ticks with validation; tick-time behavior lives in *systems*
  running in the fixed schedule (`01-ecs-model.md` §5). Keep this split intact — it is
  what makes the sim deterministic and replayable.
- **Cross-reference** with spec ids (e.g. "see `F01 S06`", "Invariant I1") rather than
  restating, to keep a single source of truth.

## When adding/changing a feature

1. Update the owning `feature.md` story index and feature-level acceptance.
2. Keep `README.md`'s feature map and the data model / glossary in sync if new types or
   terms appear.
3. If behavior affects simulation, state its determinism and fixed-point implications
   and, where relevant, add or extend a scenario in `benchmark/scenarios.md`.
4. Material conservation is **exact** and LOD losslessness is **within ε** — every new
   producer/consumer/transport spec must say how it upholds both.
