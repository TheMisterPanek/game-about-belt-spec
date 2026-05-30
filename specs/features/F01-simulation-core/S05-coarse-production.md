# F01 · S05 — Coarse Production from Profiles

**Feature:** F01 — Simulation Core & ECS
**Scope:** macro · **MVP:** yes · **Depends on:** S03, S04

## Intent
Advance inactive cells cheaply: integrate each cell's `ProductionProfile` by
`rate × Δt` instead of simulating its interior. This is the O(1)-per-cell path that
the whole LOD design rests on.

## Behavior
1. `CoarseProductionSystem` runs over every inactive cell whose `profile.valid` is
   true. For each `outputs` `RateEntry`, it accumulates `rate × Δt` (`Fixed`) and
   moves `floor(...)` discrete units into the matching `MacroOutput.buffer`, carrying
   the remainder (S03).
2. For each `demands` `RateEntry`, it pulls `floor(demand × Δt)` units from inbound
   roads / available inputs. If demand cannot be fully met, achievable output scales
   down proportionally and deterministically (a starved cell produces less).
3. Back-pressure: if a `MacroOutput.buffer` would exceed `capacity`, production for
   that resource stalls (no overflow, no loss; Invariant I2).
4. Cells with `profile.valid == false` are not coarse-produced; the scheduler keeps
   such cells active until their profile stabilizes (S04, §8).

## Data touched
`ProductionProfile` (read), `MacroOutput.buffer` (write), inbound `RoadSegment`
buffers (read/consume), `Fixed` remainder accumulators.

## Acceptance criteria
- **AC-1** *Given* a valid profile with output rate r and ample buffer space, *when*
  run for T seconds, *then* exported units = `floor(r·T)` within carry, matching a
  fine-grained reference within ε (`03-simulation-lod.md` §7).
- **AC-2** *Given* a full output buffer, *then* coarse production stalls and no units
  are lost; it resumes when space frees.
- **AC-3** *Given* a profile demanding more input than supplied, *then* output scales
  down proportionally and deterministically; no negative buffers occur.

## Out of scope
Computing the profile (F05 S04); promotion/demotion (S06).
