# F10 · S02 — Snapshot Save/Load Round-Trip

**Feature:** F10 — Persistence & Save Format
**Scope:** both · **MVP:** no · **Depends on:** S01

## Intent
Save the full persistent state at a tick and reload it to a logically identical world.

## Behavior
1. `save(World)` serializes the snapshot (S01); `load(Bytes)` rebuilds the world at the
   saved `tickNo`.
2. The loaded world is **logically equal** to the saved one: same grid, occupancy,
   buffers, profiles, and `tickNo`; material conserved exactly.
3. Equality is checked via the headless state dump (F01 S07), which excludes
   transient/presentation data.

## Data touched
Full persistent state (read on save, written on load).

## Acceptance criteria
- **AC-1** *Given* a world at `tickNo = t`, *when* saved then loaded, *then* the state
  dump matches the original at `t` (F10-A1).
- **AC-2** *Then* total material in the loaded world equals the saved world.
- **AC-3** *Then* continuing to tick the loaded world matches continuing the original
  (determinism preserved).

## Out of scope
Replay strategy (S03); transient reconstruction details (S04).
