# F06 · S05 — Power Network: Generation, Distribution, Gating

**Feature:** F06 — Micro Logistics: Miners, Belts, Power
**Scope:** micro · **MVP:** yes · **Depends on:** F05 S02

## Intent
Gate machine activity on power: `PowerSource`s generate, `PowerPole`s distribute, and
consumers (miners, assemblers) run only when powered. Power runs first each tick so
downstream systems read a settled state.

## Behavior
1. `PowerSystem` (schedule step 5) computes, per connected power network, total
   `produced` and total `consumed` (sum of consumer demands), then `available`.
2. If `available ≥ consumed`, all consumers in the network are `powered`. If supply is
   insufficient, consumers are powered down by a **deterministic** rule (e.g. by
   ascending tile coord until demand fits) so results are reproducible — partial
   brownout, not random flicker.
3. Connectivity is defined by adjacency of `PowerSource`/`PowerPole`/consumer tiles
   (documented adjacency, e.g. orthogonal + pole reach). Belts do not require power
   (they are passive) unless stated.
4. `powered` flags are set before miners/assemblers run this tick.

## Data touched
`PowerState` (produced/consumed/available/powered); network connectivity.

## Acceptance criteria
- **AC-1** *Given* supply ≥ demand, *then* every consumer in the network is `powered`.
- **AC-2** *Given* supply < demand, *then* consumers are shed in the documented
  deterministic order until supply suffices; the same set is shed across runs.
- **AC-3** *Then* `PowerSystem` runs before `MinerSystem`/`AssemblerSystem` so they see
  the settled `powered` flag this tick.
- **AC-4** *Given* a power source removed, *then* dependent consumers lose power the
  same tick.

## Out of scope
Miner/assembler behavior when powered (S01/S04); macro power (out of scope entirely —
macro manufacturers are abstracted, F08).
