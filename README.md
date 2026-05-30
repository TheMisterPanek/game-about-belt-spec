# belty

A **macro + micro factory game**, defined entirely as specification — no engine,
no programming language, no source code. The repository is the design; an
implementation is something you (or an AI model) produce *from* it.

> **Macro:** a big finite grid of cells. You route resources between cells over
> roads that behave like giant single-resource belts, and you build cell-scale
> manufacturers.
> **Micro:** step *into* a cell and you get a bounded `N×N` factory
> floor (miners, belts, power, assemblers) constrained by a fixed set of outputs.
> What you build inside a cell *is* what that cell produces on the macro map.

## Why this exists

1. **A real game design**, captured rigorously enough to build.
2. **An AI benchmark.** Hand the specs to a model, ask it to implement the game in
   any language/engine, and score the result against `benchmark/`. Because the
   specs are deliberately language- and engine-neutral (see the
   [CONSTITUTION](./CONSTITUTION.md)), the same spec set can benchmark a Rust
   implementation, a TypeScript one, a Python one, and so on, comparably.

## How to read this repo

Read in this order:

1. **[CONSTITUTION.md](./CONSTITUTION.md)** — the non-negotiable rules. Read first.
2. **[GLOSSARY.md](./GLOSSARY.md)** — shared vocabulary. Every other doc uses these
   exact terms.
3. **[specs/architecture/](./specs/architecture/)** — the big picture: ECS model,
   neutral data model, the two-tier (LOD) simulation, module boundaries.
4. **[specs/features/](./specs/features/)** — ten features (`F01`–`F10`), each a
   folder with a `feature.md` and numbered stories (`Sxx-*.md`).
5. **[benchmark/](./benchmark/)** — how an implementation is validated and scored.

To *build* from these specs (in any language): follow **[ROADMAP.md](./ROADMAP.md)** —
ordered vertical-slice milestones (M0–M8), each independently demonstrable and
benchmark-scorable. Track status in a per-run copy of **[PROGRESS.md](./PROGRESS.md)**.
"Done" vs "Verified" is defined in [specs/README.md](./specs/README.md#definition-of-done).

## Feature map

| ID  | Feature | Scope | MVP |
|-----|---------|-------|-----|
| F01 | Simulation Core & ECS | both | ✅ |
| F02 | Resource & Recipe Catalog | both | ✅ |
| F03 | World Map & Grid | macro | ✅ |
| F04 | Game States, Camera & Input | both | ✅ |
| F05 | Cell View — Finite Factory Floor | micro | ✅ |
| F06 | Micro Logistics — Miners, Belts, Power | micro | ✅ |
| F07 | Macro Routing — Roads & Big Belts | macro | ✅ |
| F08 | Macro Manufacturing | macro | ✅ |
| F09 | Blueprints — Copy / Paste | both | ✅ |
| F10 | Persistence & Save Format | both | ⏳ Post-MVP |

## Status

Specification phase. See `CONSTITUTION.md` Article II — no implementation lives
here by design.

## Contributing

Contributions are **spec changes, not code** (this repo holds no implementation by
design). See **[CONTRIBUTING.md](./CONTRIBUTING.md)** for the house rules and the
PR checklist, and **[CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)**.

## License

Released under the [MIT License](./LICENSE). © 2026 TheMisterPanek.
