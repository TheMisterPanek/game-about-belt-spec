# F10 · S01 — Versioned Save Format (Persistent State Only)

**Feature:** F10 — Persistence & Save Format
**Scope:** both · **MVP:** no · **Depends on:** F01 S07

## Intent
Define a stable, versioned representation of all persistent world state — and only the
persistent state — so saves are forward-checkable and compact.

## Behavior
1. The save carries a version integer and all persistent module state: `WorldConfig`,
   grid terrain/deposits/occupancy, micro layouts, macro outputs/road/manufacturer
   buffers, valid profiles, and `tickNo`.
2. Transient micro entities are **excluded** (reconstructed via promotion on load,
   `03-simulation-lod.md` §5). Presentation state (camera) is excluded.
3. Loaders reject unknown future versions cleanly rather than corrupting state.

## Data touched
Serialized view of all persistent state.

## Acceptance criteria
- **AC-1** *Then* the format includes everything needed to reconstruct logical state at
  `tickNo` and excludes transient/presentation data.
- **AC-2** *Given* an unknown future version, *then* load fails cleanly with a version
  error.

## Out of scope
The round-trip behavior (S02/S03); reconstruction (S04).
