# F01 · S04 — LOD Scheduler & Active-Set Selection

**Feature:** F01 — Simulation Core & ECS
**Scope:** both · **MVP:** yes · **Depends on:** S02

## Intent
Decide, each tick, which cells are simulated fine-grained. Keeping this set small and
bounded is what makes a large world affordable (CONSTITUTION Art. VI).

## Behavior
1. `LODSchedulerSystem` runs first in the schedule. It computes the **active set**
   from: the cell currently entered in `CellView` (F04), plus any pinned cells
   (`pinActive`/`unpinActive` commands), plus recently-`Dirty` cells still warming up
   (`03-simulation-lod.md` §8).
2. Cells entering the active set are **promoted** (S06); cells leaving are **demoted**
   (S06). `ActiveTag` is added/removed accordingly.
3. The active set has a bounded size; the scheduler never promotes the whole map.
   Inactive cells are handled by `CoarseProductionSystem` (S05).
4. Selection is deterministic given the same inputs (view state, pins, dirty set).

### Warm-up termination (Dirty cell stabilization)
A `Dirty` cell remains active until its `ProductionProfile` is valid (see `F05 S04`). 
The profile becomes `valid = true` when both of these conditions hold for **K = 10 
consecutive ticks**:
- The profile has not changed (output rates and demands identical to the prior tick).
- The maximum rate change across any `RateEntry` (input or output) is ≤ **1 unit/s** 
  (absolute tolerance).

If either condition fails, the counter resets. Once both hold for K ticks, the 
`Dirty` flag clears in that tick and the cell may demote on the next scheduler cycle.

## Data touched
`ActiveTag` (add/remove); reads camera/view state and pin set; `Dirty`.

## Acceptance criteria
- **AC-1** *Given* the player is inside cell C, *then* C has `ActiveTag` and is the
  only fine-grained cell unless others are pinned.
- **AC-2** *When* a cell leaves the active set, *then* it is demoted to a coarse
  profile that same tick and loses `ActiveTag`.
- **AC-3** *Given* a world of K cells with M active, *then* per-tick fine-grained work
  is proportional to M's micro entities, not K's (F01-A3).
- **AC-4** *Given* identical view/pin/dirty inputs, *then* the active set is identical
  across runs.

## Out of scope
Coarse integration math (S05); promotion/demotion correctness (S06).
