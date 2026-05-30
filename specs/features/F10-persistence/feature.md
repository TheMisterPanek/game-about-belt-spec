# F10 — Persistence & Save Format

**Module:** `persist` · **Scope:** both · **MVP:** no (Post-MVP)

## Intent

Save a world to a durable form and load it back, exactly. Because the simulation is
deterministic and transient micro entities are reconstructable via promotion, the
save format stores **persistent** state only (layouts, occupancy, buffers, profiles,
`tickNo`, command log if replay-based) and rebuilds the rest on load.

## Scope

In: serialization of all persistent module state, a versioned save format, and
load-time reconstruction (including re-promotion of the active cell). Out: cloud
sync, compression specifics, or migration tooling beyond a version field.

## Two valid strategies (implementor's choice, both must round-trip)

1. **Snapshot:** serialize the full persistent state at a `tickNo`.
2. **Replay:** serialize `WorldConfig` + the ordered `(tickNo, command)` log and
   re-simulate on load. Determinism (F01) guarantees an identical result.

A conformant save must reload to a world that is **logically equal** at the saved
`tickNo` (same as the determinism criterion, F01-A1).

## Rules of note

- Transient micro entities are **not** serialized; they are reconstructed by
  promotion from the stored `MicroGrid` layout (`03-simulation-lod.md` §5).
- The format carries a version integer; loaders reject unknown future versions
  cleanly rather than corrupting state.
- Conservation: total material in the loaded world equals the saved world.

## Stories

| ID  | Title | MVP |
|-----|-------|-----|
| S01 | Versioned save format (persistent state only) | no |
| S02 | Snapshot save/load round-trip | no |
| S03 | Replay-based save/load round-trip | no |
| S04 | Load-time reconstruction of transient state | no |

## Feature-level acceptance

- **F10-A1** Save→load yields a world logically equal at the saved `tickNo`
  (snapshot strategy) — same logical state, material conserved.
- **F10-A2** A replay save reloaded and advanced to the same `tickNo` is
  byte-equivalent in logical state to the original (determinism reuse).
- **F10-A3** Loading does not serialize transient micro entities; entering the
  previously-active cell reproduces its running factory via promotion.
