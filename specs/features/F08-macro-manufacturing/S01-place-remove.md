# F08 · S01 — Place & Remove Macro Manufacturer

**Feature:** F08 — Macro Manufacturing
**Scope:** macro · **MVP:** yes · **Depends on:** F03 S03, F02 S02

## Intent
Put a recipe-bound manufacturer on an empty cell so the macro layer can transform
resources, and remove it cleanly.

## Behavior
1. `placeManufacturer(coord, RecipeId)` succeeds only on a `Wild`, `Empty` cell with a
   valid `RecipeId`; it sets `Occupancy = Manufacturer` and creates a
   `MacroManufacturer { recipe, inputs={}, output=0, progress=0 }`. The cell's
   `MacroOutput` for the recipe's output resource is established for outbound roads.
2. `removeManufacturer(coord)` removes it only when its buffers/in-flight are drained
   (else rejected), returning the cell to `Wild/Empty`.
3. Invalid commands (non-empty terrain, bad recipe, draining manufacturer) are rejected
   with a reason, state unchanged.
4. A manufacturer has no micro grid (Invariant I3).

## Data touched
`Cell.occupancy`, `MacroManufacturer`, `Cell.outputs`.

## Acceptance criteria
- **AC-1** *Given* a `Wild` `Empty` cell + valid recipe, *then* placement sets
  `Manufacturer` and binds the recipe; *given* any other cell or bad recipe, *then*
  rejected.
- **AC-2** *Given* buffered material, *then* removal is rejected until drained; once
  empty, removal returns the cell to `Wild/Empty`.
- **AC-3** *Then* a manufacturer cell never has a micro grid (Invariant I3).

## Out of scope
Recipe processing (S02); routing integration (S03); chains (S04).
