# F01 — Simulation Core & ECS

**Module:** `sim` · **Scope:** both · **MVP:** yes

## Intent

The substrate every other feature runs on: an abstract ECS world, a deterministic
fixed-timestep tick loop, and the **two-tier LOD scheduler** that makes a large
world affordable by simulating only active cells fine-grained and everything else as
production profiles. This feature owns the schedule, determinism guarantees, and the
promotion/demotion machinery. See `specs/architecture/01-ecs-model.md` and
`03-simulation-lod.md`.

## Why it matters

LOD correctness (property **P-LOD**) is the project's signature challenge and the
most discriminating benchmark axis. Everything downstream assumes the tick is
deterministic and that profiles are a faithful reduction of micro layouts.

## Scope

In: ECS abstractions, tick loop, fixed-point clock, system schedule, LOD scheduler,
coarse production, promotion/demotion, active-set management, command/replay seam.
Out: the actual micro/macro behaviors (those are F05–F08); this feature only
sequences and schedules them.

## Stories

| ID  | Title | MVP |
|-----|-------|-----|
| S01 | Deterministic fixed-timestep tick loop | ✅ |
| S02 | ECS world, components, and ordered system schedule | ✅ |
| S03 | Fixed-point clock & rounding contract | ✅ |
| S04 | LOD scheduler & active-set selection | ✅ |
| S05 | Coarse production from profiles | ✅ |
| S06 | Promotion & demotion (lossless at steady state) | ✅ |
| S07 | Command & replay seam (headless) | ✅ |

## Feature-level acceptance

- **F01-A1** A headless run of identical `(config, command-stream)` produces
  identical logical state at every `tickNo` across repeated runs (determinism).
- **F01-A2** Property **P-LOD** holds for all reference scenarios in
  `benchmark/scenarios.md` (lossless LOD within tolerance ε; material exactly
  conserved).
- **F01-A3** With M active cells out of a world of K cells, per-tick cost scales with
  M (active) + K (O(1) coarse), not with total micro entity count of all cells.
