## Old Wikis
The active wiki no longer advertises removed or deprecated features. Wikis for old versions of the mod are hosted as branches on [a separate repository](https://github.com/Estecka/mc-Variants-CIT-wiki).
(Links across pages will be broken on this repo.)

## Removed Features / Breaking changes
### Variants-CIT v3
- Textures and models must now be located in `textures/item/` and `models/item/`. Assets outside of the `item/` subdirectory will no longer be loaded.
- Removed custom [model override predicates](https://github.com/Estecka/mc-Variants-CIT-wiki/blob/v2.9/Override%20Predicates.md), no longer supported by Minecraft.
- **Java API:** CIT modules now provide models as plain `Identifier` instead of `ModelIdentifier`.
- **Java API:** `ICitModule::getItemVariant` was moved to `ISimpleCitModule::getItemVariant`.
- **Java API:** Removed `IVariantManager::GetModelVariantForItem`.
- **Java API:** Removed module factories (`SpecialCitModuleFactory`, `ParameterizedCitModuleFactory`, `ComplexCitModuleFactory`).
### Variants-CIT v4
- Removed [item state extensions](https://github.com/Estecka/mc-Variants-CIT-wiki/blob/v3.6/Item%20State%20extensions.md).

## Deprecated Features
These features still work, but should no longer be used. Removal is not necessarily planned, but they are no longer maintained. They could change behaviour unexpectedly at the whims of minecraft internals.

### Miscellaneous
- Modules located in `variant-cits/` (mispelled) should be moved to `variants-cit/` (mod id).
- Modules should explicitely specify an `items` field, instead of naming the module itself after the target item type.
- `modelPrefix` should never be empty, or containing only `"item/"`.

### Module Types
- `stored_enchantments` (plural) should be renamed to `stored_enchantment` (singular)
- Use `component_data` instead of `custom_data`, `entity_data`, `bucket_entity_data` or `block_entity_data`.

###	Parameters of `*_data` modules
- `"caseSensitive":false` should be replaced with `"transform":"lowercase"`.
- `nbtKey` should be replaced with `nbtPath`
- `nbtPath` should no longer start directly with a key name. Old pathes should be prefixed with a dot. (`"a.b.c"` -> `".a.b.c"`)
- `validate` in regex transforms should be replaced with `optional`.

### Minecraft versions
- Variants-CIT v2 (MC 1.21 to 1.21.3) is nearing end of life. The internals of minecraft have greatly changed in MC 1.21.4, and newer mod features are becoming difficult or impossible to backport to this version.
