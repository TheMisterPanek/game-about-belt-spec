# F01 · S01 — Deterministic Fixed-Timestep Tick Loop

**Feature:** F01 — Simulation Core & ECS
**Scope:** both · **MVP:** yes · **Depends on:** —

## Intent
The clock. A single tick function advances the whole world by one fixed `Δt`,
running the system schedule once, so the simulation is deterministic and replayable
independent of frame rate or platform.

## Behavior
1. The simulation advances only via `tick(Δt)`. The canonical `Δt = 1/30` simulated
   second (`TICKS_PER_SECOND = 30`).
2. One `tick` runs the system schedule (`01-ecs-model.md` §5) exactly once, in order,
   then increments `World.tickNo` by 1.
3. Rendering/frame rate is decoupled: the host may call `tick` zero or more times per
   rendered frame (e.g. accumulate real elapsed time and step in fixed `Δt`
   increments). Interpolation for display is presentation-only.
4. No system reads wall-clock time or any non-seeded source; all time comes from `Δt`
   and `tickNo`.

## Data touched
`World.tickNo`; invokes all scheduled systems. No new data.

## Acceptance criteria
- **AC-1** *Given* a world and a command script, *when* it is ticked N times in two
  separate runs, *then* logical state at each `tickNo` is identical between runs.
- **AC-2** *Given* the same total simulated time reached via different host frame
  rates (e.g. step-per-frame vs batched), *then* the logical state at equal `tickNo`
  is identical.
- **AC-3** *When* a tick completes, *then* `tickNo` increased by exactly 1 and every
  scheduled system ran exactly once in declared order.

## Out of scope
LOD active-set selection (S04); variable timestep (Post-MVP, must reduce to fixed).
