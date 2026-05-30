# F04 · S01 — Game-State Machine: MapView ↔ CellView

**Feature:** F04 — Game States, Camera & Input
**Scope:** both · **MVP:** yes · **Depends on:** —

## Intent
The top-level mode switch that decides what the player sees and which scope input
targets.

## Behavior
1. The game is always in exactly one `Game State`: `MapView` (macro) or `CellView`
   (micro), with transitional states allowed during enter/leave animations.
2. From `MapView`, an "enter cell" action on a selected cell transitions to `CellView`
   bound to that cell. From `CellView`, a "leave" action returns to `MapView`.
3. Input mapping and rendering are determined by the current state; commands issued in
   one state target that state's scope only.
4. State changes are presentation-layer events; they may trigger commands
   (`pinActive`/`unpinActive`, S04) but never directly mutate simulation state.

## Data touched
Interaction state (presentation); issues `sim`/module commands.

## Acceptance criteria
- **AC-1** *Then* the game is in exactly one of `MapView`/`CellView` at any time.
- **AC-2** *Given* `CellView` bound to cell C, *then* placement input targets C's
  micro grid; *given* `MapView`, *then* it targets the macro grid.
- **AC-3** *When* states change, *then* logical simulation state changes only via
  issued commands, not by the transition itself.

## Out of scope
Camera (S02); LOD coupling (S04); rendering specifics.
