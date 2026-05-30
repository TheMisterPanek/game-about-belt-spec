# F09 · S01 — Capture a Cell Layout into a Blueprint

**Feature:** F09 — Blueprints: Copy / Paste
**Scope:** both · **MVP:** yes · **Depends on:** F05 S02

## Intent
Snapshot a developed cell's contents into a position-independent `Blueprint` so it can
be replicated elsewhere.

## Behavior
1. `capture(cell)` records each building as `{ rel: Coord, kind, data }` where `rel` is
   relative to a chosen origin (so the blueprint is position-independent), plus the
   source `size` and any macro `role` (Road/Manufacturer) if capturing a macro cell.
2. `data` carries the building's configuration (e.g. belt `dir`, assembler `recipe`)
   with absolute positions stripped.
3. Capture is read-only — it does not modify the source cell.
4. `OutputPort`s are **not** captured (they belong to the target cell's fixed outputs);
   only player-placed buildings are.

## Data touched
`Blueprint` (created); reads `MicroGrid`/buildings (no mutation).

## Acceptance criteria
- **AC-1** *Given* a developed cell, *then* `capture` returns a blueprint listing every
  player-placed building at correct relative coords with its config.
- **AC-2** *Then* capture leaves the source cell unchanged.
- **AC-3** *Then* output ports are excluded from the blueprint.

## Out of scope
Paste validation (S02); applying a paste (S03); macro role replication (S04).
