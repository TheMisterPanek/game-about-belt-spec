# F10 · S04 — Load-Time Reconstruction of Transient State

**Feature:** F10 — Persistence & Save Format
**Scope:** both · **MVP:** no · **Depends on:** S02, F01 S06

## Intent
Rebuild the things a save deliberately omitted — the live micro entities of the
previously-active cell — using the same promotion machinery the simulation already has.

## Behavior
1. On load, all cells start coarse (represented by their stored profiles). No transient
   micro entities are deserialized.
2. When the player enters a cell (or a pin is restored), the LOD scheduler promotes it
   (F01 S06), reconstructing and seeding its micro entities from the stored layout so
   the running factory reappears with no cold start beyond ε.
3. Material is conserved across load + first promotion (nothing created or lost).

## Data touched
Stored `MicroGrid` layouts and profiles; promotion creates transient entities.

## Acceptance criteria
- **AC-1** *Given* a loaded world, *then* no transient micro entities exist until a cell
  is promoted.
- **AC-2** *When* the previously-active cell is entered, *then* its running factory is
  reconstructed via promotion, matching its pre-save throughput within ε (F10-A3).
- **AC-3** *Then* total material immediately after load + promotion equals the saved
  world's (conservation).

## Out of scope
Save format (S01); promotion math (F01 S06).
