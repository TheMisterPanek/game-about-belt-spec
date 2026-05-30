# F05 · S04 — Profile Computation from Live Layout

**Feature:** F05 — Cell View: Finite Factory Floor
**Scope:** both · **MVP:** yes · **Depends on:** S03, F01 S03

## Intent
Reduce a live, fine-grained micro layout to a `ProductionProfile` (output rates +
input demands) so the cell can be simulated coarsely when inactive. This is the
demotion-enabling bridge and a linchpin of P-LOD.

## Behavior
1. `ProfileUpdateSystem` (schedule step 10) observes an active cell's steady-state
   export rate per resource (at its `OutputPort`s) and its steady-state input demand
   (resources consumed from inbound roads), writing them as
   `ProductionProfile.outputs` / `demands` (`RateEntry` vectors, `Fixed`).
2. "Steady state" is measured over a window; while a cell is `Dirty` (recently edited)
   or its rates vary beyond a threshold, the profile is marked `valid = false` and the
   scheduler keeps the cell active (F01 S04, §8).
3. Once rates stabilize, the system sets `valid = true` and clears `Dirty`.
4. The profile must be accurate enough that coarse simulation matches fine simulation
   within tolerance ε (`03-simulation-lod.md` §7).

## Data touched
`ProductionProfile` (write), `Dirty` (clear), reads `OutputPort`/belt/assembler state.

## Acceptance criteria
- **AC-1** *Given* a steady mining cell with miner rate r → one output, *then*
  `profile.outputs` contains `(ore, r)` within ε and `valid = true`.
- **AC-2** *Given* a cell importing X and exporting Y, *then* `demands` and `outputs`
  reflect both within ε.
- **AC-3** *Given* a just-edited (`Dirty`) cell, *then* `valid = false` until rates
  stabilize, after which `valid = true` and `Dirty` clears.
- **AC-4** *Then* coarse production using the profile matches the fine reference within
  ε (feeds F01 S05/S06).

## Out of scope
Coarse integration (F01 S05); promotion/demotion (F01 S06).
