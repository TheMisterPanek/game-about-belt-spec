# F02 — Resource & Recipe Catalog

**Module:** `catalog` · **Scope:** both · **MVP:** yes

## Intent

The single source of truth for every `Resource Type` (with its `Tier`) and every
`Recipe`. No other feature hard-codes a resource list or a crafting formula; they all
read the catalog. This is what makes "another level of resources" a data change, not
a code change, and what keeps the MVP terrain set (`Empty/Stone/Iron/Copper`)
configurable per CONSTITUTION Art. VIII.

## Scope

In: resource definitions, tiers, recipe definitions (inputs → output + craft time),
catalog queries, validation of catalog integrity. Out: how resources move or are
produced (logistics/manufacturing live in F06/F07/F08).

## Data

`ResourceDef`, `RecipeDef`, `Catalog` (`02-data-model.md` §6). The catalog is authored
data, loaded once, immutable at runtime.

## MVP catalog contents

Raw ores (tier 0): `Stone`, `Iron`, `Copper`. At least these first-tier products to
prove micro and macro manufacturing:

| Recipe | Inputs | Output | craftTime |
|--------|--------|--------|-----------|
| Smelt Iron Plate | 1 Iron | 1 `IronPlate` (tier 1) | 1.0 s |
| Smelt Copper Plate | 1 Copper | 1 `CopperPlate` (tier 1) | 1.0 s |
| Brick | 2 Stone | 1 `StoneBrick` (tier 1) | 1.0 s |
| Copper Wire | 1 CopperPlate | 2 `CopperWire` (tier 2) | 0.5 s |

(Exact balance numbers may be tuned in spec amendments; structure is binding.)

## Stories

| ID  | Title | MVP |
|-----|-------|-----|
| S01 | Resource type & tier definitions | ✅ |
| S02 | Recipe definitions & validation | ✅ |
| S03 | Catalog queries (by id, by tier, producers-of) | ✅ |

## Feature-level acceptance

- **F02-A1** Every `ResourceId`/`RecipeId` referenced anywhere in a running world
  resolves to a catalog entry (Invariant I5).
- **F02-A2** Catalog loads with no duplicate ids, no recipe referencing an undefined
  resource, and no tier-0 resource produced by any recipe (raw ores are mined, not
  crafted).
- **F02-A3** Adding a new resource + recipe to the catalog requires no change to any
  other feature's spec.
