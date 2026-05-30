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

### Promotion seeding strategy

The goal is that the promoted cell's first-tick output matches the profile's rate 
within ±20% (a goal ceiling; tighter is better).

**Belt seeding:** For each belt carrying resource R with a profile rate component:
- Pre-fill belt slots with items such that the belt will emit approximately 
  `floor(profile_rate × Δt × 0.5)` units in the next tick under normal flow.
- If the belt has 10 slots and carries 1 item per slot, that's 10 items across 
  `belt_speed` distance per tick; adjust the count to match the expected rate.

**Assembler seeding:** For each assembler with recipe producing resource R:
- Set input buffers to approximately half the recipe's input requirements.
- Set progress to `craftTime / 2`.
- This way, the assembler is "mid-craft" and will produce output soon after promotion.

**Verification:** After seeding, simulate one tick and observe the cell's output. 
If output is within ±20% of profile rate, seeding is complete. If not, adjust 
(more items on belts, higher assembler progress, etc.) and try again. This 
verification is *deterministic* and must yield the same result across runs given 
identical profiles.

**Why:** Cold-start dips (a cell producing nothing for the first few ticks, then 
ramping up) would violate P-LOD tolerance if large. Seeding pre-positions items so 
the cell is already at steady-state output. The ±20% tolerance allows reasonable 
engineering (exact per-item seeding is complex); tighter seeding improves P-LOD.

### Demotion inventory folding

When a cell is demoted, all in-flight items and partial production must be conserved 
exactly. Items and units are **1:1 equivalent**.

1. **Belt inventory:** For each `BeltState` on the micro floor, count all non-empty 
   slots. Each item counts as 1 unit of its resource.
2. **Assembler inventory:** For each `AssemblerState`, add its `inputs` map values 
   (total input units accumulated) to the in-transit total. Assemblers mid-recipe 
   have partial input but no output yet; count the inputs as conserved material.
3. **Output in flight:** Items already at `OutputPort`s (ready to export) are already 
   in `MacroOutput.buffer`; do not double-count.
4. **Accumulation:** Sum all in-flight units by resource. Add this to the appropriate 
   `MacroOutput.buffer` (matching the resource). If the cell has no matching output 
   (impossible in a well-formed layout but handle for robustness), carry the units in 
   a temporary buffer; if the cell is promoted again, prepopulate that buffer into 
   micro entities.

**Why:** Items on belts and in assembler inputs are material mid-production. Losing 
them violates conservation; double-counting would create material. Exact 1:1 folding 
ensures `total_produced` is identical before and after demotion.

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
