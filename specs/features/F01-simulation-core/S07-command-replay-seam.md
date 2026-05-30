# F01 · S07 — Command & Replay Seam (Headless)

**Feature:** F01 — Simulation Core & ECS
**Scope:** both · **MVP:** yes · **Depends on:** S01, S02

## Intent
Expose a headless way to drive the whole simulation with no rendering: generate a
world, apply an ordered command stream, tick N times, and dump logical state. The
benchmark and persistence (F10 replay) depend on this seam.

## Behavior
1. A session is fully described by `WorldConfig` plus an ordered stream of
   `(tickNo, command)` events, where commands are the documented module commands
   (`04-module-boundaries.md`).
2. Commands are applied **between** ticks, at their scheduled `tickNo`, after
   validation; an invalid command is rejected without mutating state and is recorded
   as rejected.
3. A headless host can: `generate(config)`, feed the command stream while ticking, and
   emit a deterministic textual/structured dump of logical state at any `tickNo`.
4. The dump excludes presentation-only and transient-reconstructable data (it may be
   re-derived); it includes everything needed to compare two runs for equality.

## Data touched
`World` (via commands), `tickNo`; produces a state dump (read-only view).

## Acceptance criteria
- **AC-1** *Given* the same `(config, command-stream)`, *then* the state dump at every
  `tickNo` is identical across runs and across machines (determinism, reuse of F01-A1).
- **AC-2** *Given* a command scheduled for `tickNo = t`, *then* its effect is visible
  from tick `t` onward and not before.
- **AC-3** *Given* an invalid command, *then* the dump shows it rejected and the world
  state unchanged at that tick.
- **AC-4** *A* headless run requires no camera, input device, or renderer.

## Out of scope
The on-disk format (F10); specific command semantics (owned by each module).
