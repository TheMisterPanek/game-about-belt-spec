# F02 · S02 — Recipe Definitions & Validation

**Feature:** F02 — Resource & Recipe Catalog
**Scope:** both · **MVP:** yes · **Depends on:** S01

## Intent
Define every transformation (inputs → output, with a craft time) used by micro
assemblers and macro manufacturers, validated for integrity at load.

## Behavior
1. The catalog holds `RecipeDef { id, inputs[], output, craftTime }`. `inputs` is a
   list of `(resource, count)`; `output` is one `(resource, count)`; `craftTime` is
   `Fixed` seconds.
2. Validation at load: every referenced resource exists; counts are positive;
   `craftTime > 0`; no recipe outputs a tier-0 resource (raw ores are mined, not
   crafted).
3. Recipe ids are unique and dense. The catalog is immutable at runtime.

## Data touched
`RecipeDef`, `Catalog.recipes`.

## Acceptance criteria
- **AC-1** *Given* the loaded catalog, *then* every recipe input/output resource
  exists and no recipe produces a tier-0 resource.
- **AC-2** *Given* a malformed catalog (dup id, missing resource, non-positive count
  or craft time), *then* loading fails with a clear reason; no partial catalog is
  exposed.
- **AC-3** *Given* the MVP recipes table (`feature.md`), *then* all load and validate.

## Out of scope
Crafting execution (F06 S04, F08 S02); queries (S03).
