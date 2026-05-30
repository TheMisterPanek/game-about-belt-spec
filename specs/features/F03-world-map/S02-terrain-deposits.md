# F03 · S02 — Cell Terrain, Ore Deposits & Depletion

**Feature:** F03 — World Map & Grid
**Scope:** macro · **MVP:** yes · **Depends on:** S01

## Intent
Model what a cell is made of and how much ore it holds, including optional depletion,
so mining has long-term consequences when configured.

## Behavior
1. Each resource cell holds an `OreDeposit { resource, remaining }`. `Empty` cells
   have no deposit.
2. When `WorldConfig.oreFinite == true`, extraction (F06) decrements `remaining`; at
   `remaining == 0` the deposit is exhausted and miners over it produce nothing.
3. When `oreFinite == false`, `remaining` is ignored and extraction never depletes.
4. Depletion is reflected in the cell's `ProductionProfile` (a depleted cell's output
   rate drops to 0), keeping coarse simulation consistent (F01 S05).

## Data touched
`OreDeposit`, `Terrain`, `ProductionProfile` (consequence).

## Acceptance criteria
- **AC-1** *Given* `oreFinite = true`, *when* a deposit is mined to 0, *then* further
  extraction yields nothing and the cell's profile output for that resource is 0.
- **AC-2** *Given* `oreFinite = false`, *then* extraction never changes `remaining`
  and never stops for depletion.
- **AC-3** *Then* total ore extracted over the world's life never exceeds the initial
  deposit sum when finite (conservation).

## Out of scope
The extraction machine (F06 S01); generation (S01).
