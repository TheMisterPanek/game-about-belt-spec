# F04 · S04 — Enter/Leave Cell Drives LOD Promotion/Demotion

**Feature:** F04 — Game States, Camera & Input
**Scope:** both · **MVP:** yes · **Depends on:** S01, F01 S04

## Intent
Couple the view to the simulation's LOD: entering a cell makes it fine-grained;
leaving demotes it. This is the user-facing reason a cell becomes active.

## Behavior
1. Entering `CellView` for cell C issues `pinActive(C.coord)` so the LOD scheduler
   promotes C (F01 S06). Leaving issues `unpinActive(C.coord)`, demoting C.
2. The active set always reflects the current view: the entered cell (plus any
   explicit pins) is fine-grained; everything else is coarse (F04-A2).
3. Promotion/demotion correctness is owned by F01; this story only guarantees the
   view issues the right pin commands at the right transitions.

## Data touched
Issues `pinActive`/`unpinActive`; reads active set to present.

## Acceptance criteria
- **AC-1** *When* the player enters cell C, *then* C is in the active set within one
  tick; *when* they leave, *then* C is demoted within one tick.
- **AC-2** *Given* the player is in `MapView` with no pins, *then* no cell is
  fine-grained.
- **AC-3** *Then* the entered cell's running factory is visible immediately on entry
  (promotion seeding, F01 S06), with no cold-start beyond ε.

## Out of scope
Promotion/demotion math (F01 S06); active-set selection policy (F01 S04).
