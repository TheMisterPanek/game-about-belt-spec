# F04 — Game States, Camera & Input

**Module:** `interaction` · **Scope:** both · **MVP:** yes

## Intent

The player-facing shell: the top-level state machine that switches between
**Map View** (macro) and **Cell View** (micro), the `WASD`-driven camera, and the
mapping from input to module **commands**. This layer never holds simulation state —
it only translates intent into commands and reads state to present it
(`04-module-boundaries.md` §2).

## Scope

In: game-state machine (`MapView ↔ CellView` + transitions), camera movement/zoom
within the current scope, selection and placement input, input→command mapping,
enter/leave-cell triggering LOD promotion/demotion. Out: rendering pixels (the spec
describes *what must be perceivable/interactable*, not how to draw), and the
simulation itself (F01).

## Interaction model

- **Movement:** `WASD` pans the camera in the current scope. The mouse selects a cell
  (macro) or a tile (micro) and places/removes buildings/roads.
- **Enter cell:** a player action on a developed (or developable) cell switches to
  `CellView`, which **pins that cell active** (F01 promotion). Leaving returns to
  `MapView` and unpins (demotion).
- Camera and zoom are `Real`-valued and presentation-only; they never feed
  simulation (CONSTITUTION Art. IV).

## Stories

| ID  | Title | MVP |
|-----|-------|-----|
| S01 | Game-state machine: MapView ↔ CellView | ✅ |
| S02 | WASD camera pan & zoom (per scope) | ✅ |
| S03 | Mouse selection & placement → commands | ✅ |
| S04 | Enter/leave cell drives LOD promotion/demotion | ✅ |

## Feature-level acceptance

- **F04-A1** Every player action results in a well-formed module command; invalid
  placements are rejected and surfaced, leaving state unchanged.
- **F04-A2** Entering a cell makes exactly that cell active (plus any pins); leaving
  demotes it. The active set matches the view state at all times.
- **F04-A3** Camera state changes never alter `tickNo`-indexed logical state
  (moving the camera does not affect the simulation).
