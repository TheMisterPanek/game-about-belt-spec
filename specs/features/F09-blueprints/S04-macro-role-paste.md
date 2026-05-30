# F09 · S04 — Replicate Macro Role (Road / Manufacturer)

**Feature:** F09 — Blueprints: Copy / Paste
**Scope:** macro · **MVP:** yes · **Depends on:** S03, F07 S01, F08 S01

## Intent
Extend copy/paste to macro cells so a player can replicate a road segment or a
recipe-bound manufacturer, enabling fast construction of repeated chain stages.

## Behavior
1. A blueprint of a macro cell captures its `MacroRole` (the `Occupancy` plus the road
   resource/direction or the manufacturer's `RecipeId`).
2. Pasting onto a `Wild`, `Empty` cell issues the corresponding macro command
   (`layRoad` / `placeManufacturer`) with the captured parameters, reusing F07/F08
   validation, atomically (S03 semantics).
3. Pasting a macro role onto an incompatible cell (non-empty, occupied) is rejected
   without mutation.

## Data touched
`MacroRole`; issues `layRoad`/`placeManufacturer`; `Cell.occupancy`.

## Acceptance criteria
- **AC-1** *Given* a captured manufacturer role, *then* pasting onto an empty cell
  creates a manufacturer bound to the same recipe.
- **AC-2** *Given* a captured road role, *then* pasting recreates the road with the same
  resource and direction.
- **AC-3** *Given* an incompatible target, *then* the paste is rejected without
  mutation.

## Out of scope
Micro building paste (S03); chain auto-wiring (player lays connecting roads).
