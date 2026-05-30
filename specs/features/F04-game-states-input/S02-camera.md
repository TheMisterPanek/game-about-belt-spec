# F04 · S02 — WASD Camera Pan & Zoom (per scope)

**Feature:** F04 — Game States, Camera & Input
**Scope:** both · **MVP:** yes · **Depends on:** S01

## Intent
Let the player move around the current scope with `WASD` and zoom, without ever
touching simulation state.

## Behavior
1. `W/A/S/D` pan the camera up/left/down/right within the current scope (the macro
   grid in `MapView`, the micro grid in `CellView`).
2. Zoom adjusts the visible extent within documented min/max bounds.
3. The camera is clamped to the current scope's bounds (finite map / finite micro
   grid); it cannot pan into nonexistent space.
4. Camera position/zoom are `Real`-valued and presentation-only; they are never read
   by any simulation system (CONSTITUTION Art. IV, F04-A3).

## Data touched
Camera state (presentation only).

## Acceptance criteria
- **AC-1** *Given* `WASD` input, *then* the camera pans in the corresponding direction
  within the current scope.
- **AC-2** *Given* the camera at a scope boundary, *then* further pan in that direction
  is clamped, not wrapped.
- **AC-3** *Given* any camera movement/zoom, *then* `tickNo`-indexed logical state is
  unchanged (isolation).

## Out of scope
Selection/placement (S03); state switching (S01).
