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

2. **Steady-state detection:** ProfileUpdateSystem maintains a rolling history of the 
   cell's observed output rates over the last **K = 20 ticks**. When a cell is marked 
   `Dirty`, the history is cleared.
   - Each tick, the system computes the cell's current output rate per resource 
     (units per second, `Fixed`).
   - The profile is considered **stable** if: the max rate change across any 
     `RateEntry` (input or output) is **≤ 1 unit/s** (absolute difference between 
     max and min observed over the 20-tick window).
   - If a cell is `Dirty` or rates are not stable, `profile.valid` is set to `false` 
     and the LOD scheduler keeps the cell active (F01 S04).

3. Once stable (both conditions below hold), the system locks the profile and clears 
   `Dirty`:
   - `profile.valid = true` for 2 consecutive ticks.
   - `Dirty == false`.

4. The profile must be accurate enough that coarse simulation matches fine simulation
   within tolerance ε (`03-simulation-lod.md` §7).

### Dirty flag lifecycle
- **Set:** A cell is marked `Dirty` when any of these commands succeed: `place` 
  (any building), `remove` (any building), `configureAssembler` (any recipe change).
- **Preserved on promotion:** When a cell is promoted from coarse to fine (F01 S06), 
  the `Dirty` flag is preserved; the cell continues warm-up.
- **Cleared on stabilization:** When the profile becomes stable and `valid = true` 
  for 2 consecutive ticks, `Dirty` is cleared in that tick.
- **Preserved on demotion:** When a cell is demoted from fine to coarse, the `Dirty` 
  flag is *not* cleared; it carries forward so that if the cell is promoted again, 
  it is aware the profile may be stale.

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
