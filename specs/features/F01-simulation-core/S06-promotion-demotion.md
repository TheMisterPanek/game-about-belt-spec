# F01 · S06 — Promotion & Demotion (Lossless at Steady State)

**Feature:** F01 — Simulation Core & ECS
**Scope:** both · **MVP:** yes · **Depends on:** S04, S05

## Intent
Move a cell between coarse and fine tiers without losing or inventing material and
without a visible production discontinuity — the correctness crux of the LOD design
(property **P-LOD**, `03-simulation-lod.md` §5–7).

## Behavior
1. **Promotion (coarse → fine):** reconstruct the cell's micro entities from its
   stored `MicroGrid` layout, then **seed** transient state (belts pre-filled,
   assembler progress/buffers primed) so aggregate output over the next second equals
   the profile's rate within ε. Profile authority yields to the live sim.
2. **Demotion (fine → coarse):** ensure a `valid` profile reflects current
   steady-state throughput (S04/F05 S04), **fold in-flight inventory** (items on
   belts / in buffers) into the cell's output buffer or a carried amount so material
   is conserved exactly, then delete transient entities.
3. Material conservation across either transition is **exact** (not within ε); only
   export *timing* may differ within ε.
4. Promotion/demotion happen during `LODSchedulerSystem` (S04), before downstream
   systems read the cell's tier this tick.

## Data touched
Micro entities (create/delete), `ProductionProfile` (read/write), `MacroOutput.buffer`
/ carried inventory (write), `ActiveTag`, `Dirty`.

## Acceptance criteria
- **AC-1** *Given* a steady cell exported X units over T while always fine, *when* the
  same cell alternates fine/coarse during T, *then* total exported equals X within ε
  (P-LOD).
- **AC-2** *When* a cell is demoted, *then* total material (buffered + in-flight folded
  + exported) equals the pre-demotion total exactly — nothing lost.
- **AC-3** *When* a cell is promoted, *then* no units are created; the seeded floor's
  first-second output matches the profile rate within ε with no cold-start dip beyond
  ε.
- **AC-4** *Repeated* promote/demote cycles do not drift totals over many iterations
  (no accumulating error).

## Out of scope
Selecting which cells flip (S04); coarse math (S05); profile computation (F05 S04).
