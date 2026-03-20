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
### Variants-CIT v5
- Removed the Java API
- Removed module types: `custom_data`, `entity_data`, `bucket_entity_data` and `block_entity_data`.  
Use `component_data` instead.
- Removed module parameters: `caseSensitive` and `nbtKey`.  
  Use `nbtPath` and `"transform":"lowercase"` instead.
- Removed support for the old syntax of `nbtPath`.  
  Most old pathes can be fixed by adding a dot at their beginning: `a.b.c` => `.a.b.c`
- `expect` in `item_component` now defaults to accepting every data types.
- Removed `expect` values: `auto`, `primitive`.
- `sanitize`'s behaviour was changed to be the same as `sanitize_auto`.
- `custom_name` module's flavour of sanitize was changed to `sanitize_auto`

## Deprecated Features
These features still work, but should no longer be used. Removal is not necessarily planned, but they are no longer maintained. They could change behaviour unexpectedly at the whims of minecraft's internals.

### Modules
- Modules located in `variant-cits/item/` (mispelled) and `variants-cit/item/` should be moved to `variants-cit/modules/`.
- Modules should explicitely specify an `items` field, instead of naming the module itself after the target item type.
- `modelPrefix` should never be empty, or contain only `"item/"`.
- Module type `stored_enchantments` (plural) should be renamed to `stored_enchantment` (singular)

###	Transforms & Item Properties:
- `validate` in regex transforms should be replaced with `optional`.
- `expect` should be replaced with its new `transform` counterparts.

### Minecraft versions
- Variants-CIT v2 (MC 1.21 to 1.21.3) has reached end of life. It may still receive bugfixes, but no new major features. Newer versions of VCIT are unlikely to be backported to this version of Minecraft.

- Variants-CIT v3 and v4 (MC 1.21.4+) are discontinued. Variants-CIT v5 replaces them for the corresponding versions of Minecraft.
