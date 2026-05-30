# F05 · S02 — Tile Placement & Removal Validation

**Feature:** F05 — Cell View: Finite Factory Floor
**Scope:** micro · **MVP:** yes · **Depends on:** S01

## Intent
The placement rulebook for the micro floor: where a building may go, enforced
uniformly so every feature (including blueprint paste) reuses it.

## Behavior
1. `place(cell, tile, BuildingKind, dir)` succeeds only if: the tile is in-bounds
   (`0..size-1`), currently empty, and not an `OutputPort` tile; and the kind is legal
   for the cell (e.g. a `Miner` is meaningful only over ore terrain — placement on a
   non-ore cell is rejected for `Miner`).
2. `remove(cell, tile)` clears a building only if no in-flight item would be orphaned
   in a way that violates conservation (items are dropped to the cell's carried
   inventory, not deleted).
3. A tile holds at most one building.
4. Any successful placement/removal marks the cell `Dirty` so its profile recomputes
   (F01/F05 S04). Invalid commands are rejected with a reason, state unchanged
   (F05-A1).

## Data touched
`MicroGrid.tiles`, `BuildingKind`, `Dirty`; carried inventory on remove.

## Acceptance criteria
- **AC-1** *Given* an out-of-bounds, occupied, or output tile, *then* placement is
  rejected and state is unchanged.
- **AC-2** *Given* a `Miner` on a non-ore cell, *then* placement is rejected.
- **AC-3** *When* a building is removed with items inside, *then* those items move to
  carried inventory (conserved), not deleted.
- **AC-4** *When* placement/removal succeeds, *then* the cell becomes `Dirty`.

## Out of scope
Machine behavior (F06); export (S03); atomic multi-tile paste (F09).
