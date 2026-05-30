# F08 · S02 — Recipe Processing with Input Buffers

**Feature:** F08 — Macro Manufacturing
**Scope:** macro · **MVP:** yes · **Depends on:** S01

## Intent
Run the recipe at macro scale: accumulate routed inputs, craft over time, emit output —
the coarse counterpart to a micro assembler (no per-item belts inside).

## Behavior
1. `MacroManufactureSystem` (schedule step 3) pulls inbound resources from connected
   roads into `inputs` buffers (per resource). When all recipe inputs meet their
   counts, it advances `progress` by `Δt` until `craftTime`.
2. On completion it deducts input counts, adds the recipe output count to `output`, and
   resets `progress` (carrying overrun deterministically). `output` accumulates into the
   cell's `MacroOutput.buffer` for outbound roads, back-pressuring at capacity (I2).
3. Material is conserved exactly per the recipe over any run. Starved or output-blocked
   manufacturers hold progress without consuming inputs.

## Data touched
`MacroManufacturer` (inputs/output/progress), `MacroOutput.buffer`, inbound roads,
catalog `RecipeDef`.

## Acceptance criteria
- **AC-1** *Given* sufficient routed inputs, *then* output is produced at
  `1 craft / craftTime` within ε, consuming exactly recipe-input counts per craft.
- **AC-2** *Given* missing inputs or full output, *then* progress holds and inputs are
  not consumed; no material is lost.
- **AC-3** *Over* a long run, inputs consumed and outputs produced satisfy the recipe
  ratio exactly (conservation, F08-A2).

## Out of scope
Road flow (F07); placement (S01); multi-stage chains (S04).
