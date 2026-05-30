# F04 · S03 — Mouse Selection & Placement → Commands

**Feature:** F04 — Game States, Camera & Input
**Scope:** both · **MVP:** yes · **Depends on:** S01

## Intent
Translate pointer actions into validated module commands: select a cell/tile, place or
remove a building/road/manufacturer.

## Behavior
1. The mouse selects the cell under the pointer (`MapView`) or the tile under the
   pointer (`CellView`), accounting for camera pan/zoom.
2. Placement actions emit the appropriate command: micro `place/remove/configure`
   (F05/F06), macro `layRoad/clearRoad` (F07), `placeManufacturer/removeManufacturer`
   (F08), or `paste` (F09).
3. Every emitted command is validated by its owning module; invalid placements are
   rejected and surfaced to the player, leaving state unchanged (F04-A1).
4. The interaction layer never bypasses module validation or mutates state directly.

## Data touched
Issues module commands; reads world/micro state to present selection.

## Acceptance criteria
- **AC-1** *Given* a pointer over a tile/cell, *then* selection resolves to the correct
  coord under the current camera transform.
- **AC-2** *Given* a valid placement, *then* exactly one well-formed command is emitted
  and applied; *given* an invalid one, *then* it is rejected and state is unchanged.
- **AC-3** *Then* no placement path mutates simulation state except through a module
  command.

## Out of scope
The command semantics themselves (owning features); camera math (S02).
