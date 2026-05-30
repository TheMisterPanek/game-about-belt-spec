# F06 — Micro Logistics: Miners, Belts, Power

**Module:** `microfactory` (machines) · **Scope:** micro · **MVP:** yes

## Intent

The machines that make a micro factory work: `Miner`s that extract the cell's ore,
single-resource `Belt`s that transport items, `Assembler`s that craft per a `Recipe`,
and a `Power` network that gates them. The project's signature twist: **one resource
per belt** (CONSTITUTION Art. VII), never two interleaved lanes.

## Scope

In: miner extraction, belt transport & item hand-off, assembler crafting, power
generation/distribution/gating, and the deterministic micro system schedule.
Out: the grid & outputs (F05), profiles' macro use (F01), macro manufacturing (F08).

## Data

`MinerState`, `BeltState`, `AssemblerState`, `PowerState`, `Direction`
(`02-data-model.md` §4). Systems (schedule order): `PowerSystem`, `MinerSystem`,
`BeltSystem`, `AssemblerSystem` (`01-ecs-model.md` §4–5).

## Rules of note

- **Single-resource belts (I1):** a belt's `resource` may change only while it is
  empty. Trying to feed a second resource type onto an occupied belt is back-pressured
  at the source, not merged.
- **Power gates everything:** an unpowered consumer does no work this tick;
  `PowerSystem` runs first so downstream systems read a settled `powered` flag.
- **Determinism:** belts advance items in a fixed per-belt order; junction hand-off
  resolves by a documented priority (e.g. clockwise from belt's facing) so two
  implementations agree item-for-item.
- All rates/progress are `Fixed`; discrete items are produced by flooring accumulated
  `Fixed` and carrying the remainder (`03-simulation-lod.md` §2).

## Stories

| ID  | Title | MVP |
|-----|-------|-----|
| S01 | Miner extraction & ore depletion | ✅ |
| S02 | Single-resource belt transport | ✅ |
| S03 | Belt junctions & deterministic hand-off | ✅ |
| S04 | Assembler crafting from recipe | ✅ |
| S05 | Power network: generation, distribution, gating | ✅ |

## Feature-level acceptance

- **F06-A1** A belt never simultaneously holds two distinct resource types
  (Invariant I1), under any placement/feed sequence.
- **F06-A2** A powered miner over its ore raises the cell's matching output buffer at
  its `rate` (within ε at steady state); cutting power halts it the same tick.
- **F06-A3** An assembler consumes exactly its recipe inputs and emits exactly its
  recipe output, conserving material across long runs (no creation/loss).
- **F06-A4** Given the same layout and inputs, two runs advance items along belts
  identically tick-by-tick (determinism).
