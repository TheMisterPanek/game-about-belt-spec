# CONSTITUTION

The supreme law of this project. Every spec, story, architecture note, and future
implementation **must** conform to the articles below. When any other document
conflicts with this one, this document wins. Amendments require an explicit,
dated entry in §X.

---

## Article I — Purpose

1. This repository defines a **macro + micro factory-building game** entirely as
   specification. It is a *spec-first*, *implementation-free* project.
2. Its second purpose is to serve as a **cross-language, cross-engine benchmark**
   for AI coding models: a model is given these specs and asked to implement the
   game in a target language/engine. The quality of the result is scored against
   the acceptance criteria defined in `benchmark/`.
3. Because of (2), the specs are the *product*. They must be precise enough that
   two independent implementors, in two different languages, produce games that
   behave identically on the reference scenarios.

## Article II — Spec-First

1. **No production code lives in this repository.** Not in any language. The repo
   contains only Markdown specifications, diagrams-as-text, and pseudo-notation.
2. A feature may not be considered "ready to implement" until it has: a
   `feature.md`, at least one story with acceptance criteria, and a mapping to the
   data/ECS model.
3. Changes flow **spec → review → (downstream) implementation**, never the reverse.
   If an implementation reveals a spec gap, the fix is a spec amendment, not an
   undocumented code decision.

## Article III — Language & Engine Neutrality

1. Specs **must not** assume any programming language, runtime, game engine,
   rendering API, or platform. Forbidden: references to specific types
   (`std::vec`, `ArrayList`), engines (Unity, Godot, Bevy), or language idioms.
2. All data structures are expressed in the **neutral pseudo-notation** defined in
   `specs/architecture/02-data-model.md`.
3. Where a concept *could* be implemented many ways (memory layout, dispatch,
   storage), the spec describes **observable behavior and invariants**, not the
   mechanism. The mechanism is the implementor's freedom.
4. UI/rendering specs describe **what must be perceivable and interactable**, never
   how to draw it.

## Article IV — Determinism & Reproducibility

1. The simulation is **deterministic**: given the same `seed`, the same initial
   configuration, and the same ordered input stream, every conformant
   implementation must reach byte-equivalent *logical* state at every tick.
2. All randomness derives from the world `seed` through a specified, documented
   procedure. No implementation may use a non-seeded or wall-clock source for
   simulation-affecting decisions.
3. Floating point is a known hazard for cross-language determinism. Therefore:
   simulation state that must match across implementations is defined in
   **fixed-point / integer quantities** (see `specs/architecture/03-simulation-lod.md`).
   Rendering and camera may use real numbers freely; the simulation may not.

## Article V — The Two Worlds

1. The game has exactly two interactive scopes that share one simulation:
   - **Macro (Map View):** a large finite grid of *cells*. Strategy, routing, and
     cell-to-cell logistics happen here.
   - **Micro (Cell View):** the interior of a single cell, a finite `N×N` factory
     floor bounded by a fixed set of **outputs**.
2. The two scopes are not separate games; the micro layout of a cell **produces**
   the macro behavior (throughput) of that cell. The macro layer consumes those
   products and routes them.

## Article VI — Level-of-Detail Simulation

1. The world may be very large. Full fine-grained simulation of every cell every
   tick is **not permitted as a requirement**; it is an explicit non-goal.
2. The canonical model is **two-tier**:
   - The cell the player currently occupies (and any explicitly "active" cells) is
     simulated **fine-grained** (belts, items, machines tick individually).
   - All other cells are simulated **coarse-grained**: each carries a *production
     profile* (a steady-state rate vector) and advances by `rate × Δt`.
3. Entering a cell promotes it to fine-grained; leaving it demotes it back to a
   profile. The demotion/promotion **must be lossless at steady state** within the
   tolerance defined in the simulation spec.

## Article VII — Single-Resource Logistics

1. A conveyor/belt — at both micro and macro scale — carries **exactly one resource
   type at a time**, never two interleaved lanes. This is a hard rule, not a tuning
   choice.

## Article VIII — Scope Discipline (MVP)

1. The first milestone (MVP) terrain set is exactly: `Empty`, `Stone`, `Iron`,
   `Copper`. New resource/terrain types are added only via the catalog
   (`F02`) and never hard-coded into other features.
2. Features marked *Post-MVP* in their `feature.md` are planned but out of scope
   for the first benchmark run. They must not be prerequisites of MVP stories.

## Article IX — Every Story Is Testable

1. Every story states **acceptance criteria** as observable, checkable statements
   (Given/When/Then or equivalent).
2. Criteria reference the **glossary** terms and the **data model**; they do not
   introduce new vocabulary silently.
3. If a criterion cannot be checked without seeing source code, it is wrong and
   must be rewritten in terms of behavior, state, or output.

## Article X — Amendments

Amendments are appended here with a date and a one-line rationale.

- _2026-05-30_ — Constitution ratified. Initial articles I–X.
