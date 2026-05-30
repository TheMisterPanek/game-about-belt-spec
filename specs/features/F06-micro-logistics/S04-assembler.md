# F06 · S04 — Assembler Crafting from Recipe

**Feature:** F06 — Micro Logistics: Miners, Belts, Power
**Scope:** micro · **MVP:** yes · **Depends on:** S02, F02 S02

## Intent
Transform resources on the micro floor: an `Assembler` consumes recipe inputs from
adjacent belts and produces the recipe output, the micro-scale crafting step.

## Behavior
1. An `Assembler` is bound to a `RecipeId` (F02). It pulls input items from adjacent
   belts into `inputs` buffers. When all inputs meet the recipe counts and the machine
   is `powered`, it advances `progress` by `Δt` until `craftTime`.
2. On completion it deducts the input counts, increments `output`, and resets
   `progress` (carrying any overrun deterministically). Output items leave onto an
   adjacent belt in its facing direction, back-pressuring if blocked.
3. Material is conserved exactly: inputs consumed and outputs produced match the recipe
   over any run; nothing is created or lost.
4. Unpowered or input-starved assemblers hold progress without consuming inputs.

## Data touched
`AssemblerState` (recipe, inputs, output, progress), adjacent `BeltState`,
`PowerState.powered`, catalog `RecipeDef`.

## Acceptance criteria
- **AC-1** *Given* sufficient inputs and power, *then* the assembler produces output at
  `1 craft / craftTime` within ε, consuming exactly recipe-input counts per craft.
- **AC-2** *Given* missing inputs or no power, *then* progress holds and no inputs are
  consumed.
- **AC-3** *Given* a blocked output belt, *then* finished output buffers and crafting
  stalls; no item is lost.
- **AC-4** *Over* a long run, total inputs consumed and outputs produced satisfy the
  recipe ratio exactly (conservation).

## Out of scope
Recipe data (F02); macro manufacturers (F08); power generation (S05).
