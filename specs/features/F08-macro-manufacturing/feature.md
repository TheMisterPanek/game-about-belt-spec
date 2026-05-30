# F08 — Macro Manufacturing

**Module:** `macromfg` · **Scope:** macro · **MVP:** yes

## Intent

Let the macro layer *transform* resources, not just move them. A `Manufacturer`
placed on an `Empty` cell consumes resources routed in by roads (F07) and produces a
higher-`Tier` resource per a `Recipe`, which roads then carry onward. This is how the
player builds **cross-cell production chains** — the macro counterpart to a micro
assembler — turning the map itself into a factory graph.

## Scope

In: placing/removing macro manufacturers, their input buffers and recipe processing,
the `MacroManufactureSystem`, and their participation in routing (as a demand sink and
a production source). Out: micro assemblers (F06), the roads feeding them (F07), the
recipe data (F02).

## Data

`MacroManufacturer`, plus the cell's `Occupancy == Manufacturer` and its
`MacroOutput` for the produced resource (`02-data-model.md` §2,§5). System:
`MacroManufactureSystem` (`01-ecs-model.md` §4, schedule step 3).

## Rules of note

- A manufacturer is bound to one `RecipeId`; its input demands and output resource
  come from that recipe (F02), keeping resources data-driven.
- It pulls inputs from inbound roads into `inputs` buffers; when all inputs meet the
  recipe counts, it advances `progress` by `Δt` until `craftTime`, then converts
  inputs→output deterministically.
- Output accumulates in the cell's `MacroOutput.buffer` and is carried away by
  outbound roads; at capacity it back-pressures (stalls), conserving material (I2).
- Always coarse/macro: manufacturers run every tick regardless of LOD tier; no micro
  grid is involved (Invariant I3: only Mines have micro grids).

## Stories

| ID  | Title | MVP |
|-----|-------|-----|
| S01 | Place & remove macro manufacturer | ✅ |
| S02 | Recipe processing with input buffers | ✅ |
| S03 | Integration with routing (demand & supply) | ✅ |
| S04 | Multi-stage cross-cell production chains | ✅ |

## Feature-level acceptance

- **F08-A1** A manufacturer may be placed only on an `Empty/Wild` cell, bound to a
  valid `RecipeId`; removal returns the cell to `Empty/Wild`.
- **F08-A2** It consumes exactly recipe-input counts and emits exactly recipe-output
  counts per craft, conserving material over long runs.
- **F08-A3** A chain `mine → road → manufacturer → road → manufacturer` reaches a
  deterministic steady-state throughput governed by the slowest stage (bottleneck),
  identical across runs.
