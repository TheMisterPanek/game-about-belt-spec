# Architecture — Simulation & Level-of-Detail

This is the heart of the project and the most discriminating part of the benchmark.
It specifies the tick model, the fixed-point contract, and the two-tier (LOD)
simulation with lossless promotion/demotion.

## 1. Tick model

- The simulation advances in **ticks**. Each tick applies a delta-time `Δt`.
- The canonical mode is a **fixed timestep**: `Δt = 1/30` simulated second per tick
  (`TICKS_PER_SECOND = 30`). Rendering may run at any rate and interpolate; it must
  not change `Δt`.
- A tick runs the system schedule from `01-ecs-model.md` §5 exactly once.
- `World.tickNo` increments by 1 per tick. Determinism is defined per `tickNo`:
  two conformant implementations at the same `tickNo`, fed the same inputs, hold
  equal logical state.

> Variable timestep is **Post-MVP** and, if added, must reduce to identical results
> as the fixed step under the same total elapsed time (it is an optimization, not a
> behavior change).

## 2. Fixed-point contract

- Simulation-affecting non-integer quantities (rates, progress, times) use
  **fixed-point** with a **global scale of 1/1024** (i.e. values are stored as
  `Int` numerators over denominator `1024`). Call this `Fixed`.
- Rounding rule: when converting an accumulated `Fixed` amount into discrete `Int`
  units (e.g. items produced), use **floor**, and carry the remainder forward in the
  source's `Fixed` accumulator. No amount is ever lost or created by rounding.
- Multiplication/division order is specified where it matters (e.g. `rate × Δt`
  computed as `Fixed`), so cross-language results match bit-for-bit at the chosen
  scale.

## 3. The two tiers

Every cell is in exactly one tier each tick:

| Tier | When | How it advances | Cost |
|------|------|-----------------|------|
| **Fine** | cell has `ActiveTag` | full micro system schedule over its entities | high |
| **Coarse** | otherwise | `CoarseProductionSystem`: integrate `profile × Δt` | O(1) |

The **active set** is small and bounded (typically: the cell the player is inside,
plus any explicitly pinned cells). `LODSchedulerSystem` recomputes it each tick from
camera state + pins.

## 4. Production Profile — the bridge

A cell's `ProductionProfile` summarizes what its micro layout does at steady state:
output rates and input demands (see `02-data-model.md` §3).

- **Source of truth while Fine:** the live micro simulation. The
  `ProfileUpdateSystem` observes the cell's steady-state output/consumption and
  writes the profile, marking `valid = true`. A `Dirty` flag (set whenever the micro
  layout is edited) forces recomputation.
- **Source of truth while Coarse:** the profile itself; `CoarseProductionSystem`
  moves `floor(rate × Δt)` units (carrying remainders, §2) from "production" into the
  cell's `MacroOutput.buffer`, and pulls `floor(demand × Δt)` from inbound roads.
- Back-pressure: coarse production stalls when `MacroOutput.buffer` would exceed
  `capacity` (Invariant I2); demand that cannot be met scales the achievable output
  proportionally (a starved cell produces less, deterministically).

## 5. Promotion (coarse → fine)

When a cell becomes active:

1. Reconstruct its micro entities from the stored `MicroGrid` layout (the layout is
   persisted; only the live transient entities were absent).
2. **Seed transient state** so the floor is already at the steady state the profile
   implied — belts pre-filled, assembler buffers/progress primed — so there is no
   visible "cold start" pop. The seeding target is the profile's rates.
3. Remove the profile's authority: from now until demotion, the live sim governs.

The promotion seeding need not be exact per-item, but the cell's *aggregate output
over the next second* must equal the profile's rate within **tolerance ε**.

## 6. Demotion (fine → coarse)

When a cell stops being active:

1. Ensure `ProfileUpdateSystem` has produced a `valid` profile reflecting current
   steady-state throughput.
2. **Conserve in-flight inventory:** items currently on belts / in buffers are not
   destroyed — they are folded into the cell's output buffer or a carried-inventory
   amount, so total material is conserved across the transition.
3. Delete the transient micro entities. The cell is now coarse.

## 7. Losslessness requirement & tolerance

The central correctness property:

> **P-LOD.** For a cell at steady state, the total resource exported over a long
> interval `T` must be equal — within tolerance — whether the cell was Fine, Coarse,
> or alternated between them during `T`.

- Tolerance `ε`: the cumulative export over `T = 60` simulated seconds must match
  the all-Fine reference to within **±1 unit per resource, plus ≤1%** of the
  reference total, whichever is larger.
- Material conservation is **exact** (not within ε): promotion/demotion may not
  create or destroy units (§5.2, §6.2). Only the *timing* of exports may differ
  within ε.

The benchmark (`benchmark/scenarios.md`) checks P-LOD directly by running scenarios
both all-Fine and with forced promotion/demotion cycles and comparing totals.

## 8. Non-steady-state behavior

Profiles model steady state. During transients (a layout just changed, ore just
depleted, a road just connected) a cell may be Fine for a warm-up period before its
profile is trusted. `LODSchedulerSystem` may keep a recently-edited (`Dirty`) cell
Fine until its profile stabilizes (output rate variance below a threshold over a
window). This avoids demoting a cell whose profile is not yet representative.

## 9. Ordering & concurrency

- Systems run in the fixed schedule order (`01-ecs-model.md` §5).
- Cells may be simulated in parallel **only** if cross-cell interactions
  (routing/manufacturing reads of `MacroOutput.buffer`) are resolved in the
  serialized macro systems (steps 2–4), which run before/after the per-cell micro
  work, never interleaved with it in a result-affecting way.
