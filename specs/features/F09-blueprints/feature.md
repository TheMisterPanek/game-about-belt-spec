# F09 — Blueprints: Copy / Paste

**Module:** `blueprint` · **Scope:** both · **MVP:** yes

## Intent

Capture an existing cell's layout as a position-independent `Blueprint` and paste it
onto another cell, so the player can replicate a working factory and build big chains
without re-placing every building. Supports the design goal: "copy-paste existing
manufacturing on a cell wholly and paste to another cell to build big chains."

## Scope

In: capturing a developed cell's micro layout (and/or its macro role) into a
`Blueprint`, validating a paste target, and applying a paste as a batch of placement
commands. Out: the placement rules themselves (F05/F06/F07/F08 — paste reuses their
validation), persistence of blueprints to disk (F10).

## Data

`Blueprint`, `BuildingData`, `MacroRole` (`02-data-model.md` §7). Commands:
`capture(cell) → Blueprint`, `paste(Blueprint, cell)`, query `canPaste(...)`.

## Rules of note

- A blueprint stores buildings by **relative** coord (origin-anchored), so it is
  position-independent and reusable across cells.
- **Paste validity:** target `MicroGrid.size` must equal `Blueprint.size`; every
  building must land on a legal, empty tile per F05/F06 validation; required
  terrain/outputs must be compatible (e.g. a miner only works over ore — paste onto a
  non-ore cell still places it but it produces nothing, or is rejected per S03's rule).
- **Atomicity:** a paste either fully applies or fully rejects; it never leaves a cell
  half-built. It reuses each module's command validation; if any sub-placement is
  invalid, the whole paste fails with a reason.
- After a successful paste, the target cell is marked `Dirty` so its profile
  recomputes (F01/F05).

## Stories

| ID  | Title | MVP |
|-----|-------|-----|
| S01 | Capture a cell layout into a blueprint | ✅ |
| S02 | Validate paste target (size, terrain, occupancy) | ✅ |
| S03 | Atomic paste as batched commands | ✅ |
| S04 | Replicate macro role (road/manufacturer) | ✅ |

## Feature-level acceptance

- **F09-A1** Capturing then pasting onto an empty, compatible cell reproduces the
  source layout exactly (same buildings at same relative coords, same configs).
- **F09-A2** A paste that violates any placement rule applies **none** of its
  buildings and reports why (atomicity).
- **F09-A3** A pasted-and-identical micro layout, once steady, yields the same
  `ProductionProfile` as its source (replication is behavior-preserving).
