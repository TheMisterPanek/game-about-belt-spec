# F09 · S03 — Atomic Paste as Batched Commands

**Feature:** F09 — Blueprints: Copy / Paste
**Scope:** both · **MVP:** yes · **Depends on:** S02

## Intent
Apply a blueprint as a single all-or-nothing operation, reusing each module's placement
validation, so a cell is never left half-built.

## Behavior
1. `paste(Blueprint, cell)` first runs `canPaste` (S02). If it fails, **nothing** is
   applied (atomicity, F09-A2).
2. If valid, it issues the underlying placement commands (F05/F06 for micro) for every
   building, translating relative coords to absolute on the target. Each reuses its
   module's validation.
3. On success the target cell is marked `Dirty` so its profile recomputes
   (F01/F05 S04). On any unexpected sub-failure, already-applied placements are rolled
   back so the cell returns to its pre-paste state.

## Data touched
Issues `place`/`configure` commands; `MicroGrid.tiles`; `Dirty`.

## Acceptance criteria
- **AC-1** *Given* a valid paste, *then* the target reproduces the blueprint exactly
  (same buildings, relative coords, and configs — F09-A1).
- **AC-2** *Given* any invalid sub-placement, *then* none of the paste is applied and a
  reason is reported (atomicity).
- **AC-3** *When* a paste succeeds, *then* the cell becomes `Dirty` and its profile
  recomputes to match the source's at steady state (F09-A3).

## Out of scope
Validation rules (S02); macro role paste (S04).
