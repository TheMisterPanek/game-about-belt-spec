# F10 · S03 — Replay-Based Save/Load Round-Trip

**Feature:** F10 — Persistence & Save Format
**Scope:** both · **MVP:** no · **Depends on:** F01 S07

## Intent
An alternative save strategy: store the inputs, not the state. Determinism guarantees
re-simulation reproduces the world exactly.

## Behavior
1. A replay save serializes `WorldConfig` plus the ordered `(tickNo, command)` log
   (F01 S07).
2. `load` re-runs `generate(config)` and replays the command stream, ticking to the
   saved `tickNo`, yielding a byte-equivalent logical state to the original (reuse of
   determinism, F01-A1).
3. Replay and snapshot strategies are interchangeable from the player's view; an
   implementation may offer either or both.

## Data touched
`WorldConfig`, command log; re-derives all state by simulation.

## Acceptance criteria
- **AC-1** *Given* a replay save, *when* loaded and advanced to the saved `tickNo`,
  *then* logical state is byte-equivalent to the original (F10-A2).
- **AC-2** *Then* no transient or derived state is stored in the replay save (only
  config + commands).
- **AC-3** *Then* a replay save and a snapshot save of the same world load to equal
  logical state.

## Out of scope
Snapshot strategy (S02); the headless seam itself (F01 S07).
