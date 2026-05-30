# F06 · S01 — Miner Extraction & Ore Depletion

**Feature:** F06 — Micro Logistics: Miners, Belts, Power
**Scope:** micro · **MVP:** yes · **Depends on:** F05 S02, F03 S02

## Intent
The source of all raw resources: a powered `Miner` over its cell's ore emits the
matching resource at a rate, depleting the deposit when finite.

## Behavior
1. A `Miner` extracts the owning cell's `Terrain` resource at `MinerState.rate`
   (`Fixed`, units/second) when `powered` (F06 S05). Unpowered miners do nothing.
2. `MinerSystem` (schedule step 6) accumulates `rate × Δt`, emits `floor(...)` items
   into the adjacent belt/buffer in the miner's facing direction (carrying remainder,
   F01 S03). If the downstream is full, the miner back-pressures (its internal buffer
   fills, then it stalls; no loss).
3. When `oreFinite` (F03 S02), each emitted unit decrements `OreDeposit.remaining`; at
   0 the miner produces nothing.

## Data touched
`MinerState`, `OreDeposit.remaining`, adjacent `BeltState`/buffer, `PowerState.powered`.

## Acceptance criteria
- **AC-1** *Given* a powered miner over ore with a free downstream, *then* it emits at
  `rate` within ε at steady state (verified by unit count over time).
- **AC-2** *When* power is cut, *then* the miner halts the same tick.
- **AC-3** *Given* a full downstream, *then* the miner stalls without losing units.
- **AC-4** *Given* finite ore mined to 0, *then* extraction stops and never goes
  negative (conservation).

## Out of scope
Belt transport (S02); power network (S05); profile effect (F05 S04).
