# F05 · S01 — Micro Grid Creation & Bounded Sizing

**Feature:** F05 — Cell View: Finite Factory Floor
**Scope:** micro · **MVP:** yes · **Depends on:** F03 S03

## Intent
Create the bounded `N×N` interior of a developed cell — the finite factory floor that
makes this a bounded, finite factory floor — not an infinite world.

## Behavior
1. Developing a cell (`Wild → Mine`, F03 S03) creates a `MicroGrid` with
   `size == WorldConfig.cellSize` (default 50), fixed for the world's lifetime.
2. The grid is initially empty except for its boundary `OutputPort`s (S03), which are
   placed to match the cell's fixed `MacroOutput`s (F03 S04).
3. Coords are `0..size-1` on each axis; the grid is finite and non-wrapping.
4. The micro grid exists only while the cell is `Mine` (Invariant I3) and its live
   entities exist only while active (Invariant I4).

## Data touched
`MicroGrid`, `OutputPort`, `Cell.micro`.

## Acceptance criteria
- **AC-1** *Then* a developed cell's grid is exactly `cellSize × cellSize`, with
  boundary outputs matching the cell's macro outputs.
- **AC-2** *Given* `oreFinite` or terrain, *then* grid size is independent of them
  (always `cellSize`).
- **AC-3** *When* the cell stops being `Mine`, *then* its micro grid is removed.

## Out of scope
Placement rules (S02); export (S03); machines (F06).
