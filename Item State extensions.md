# Item State Extensions

Item States (resources under the `items/` folder) are the vanilla way of changing an item's model. Those can be used to provide a second layer of variance to an item, on top of the modules from this mod.

> [!IMPORTANT]
>
> This page is only applicable to MC 1.21.4 and onward.

## Numeric Properties
Properties to be used inside the vanilla `dispatch_range` selector

### `variants-cit:stored_enchantment_level`

Returns the level of the first enchantment in `stored_enchantment`.
Behaviour is undefined on books with multiple enchantments.

> [!TIP]
>
> The `stored_enchantment` module now has built-in support for enchantment levels.

### `variants-cit:custom_data`, `entity_data`, `bucket_entity_data`, `block_entity_data`
Returns the value of the a number stored in the NBT component with matching name.

#### Parameters:
- `nbtPath`: *Mandatory, String.* The path to the number, with dot '`.`' separated elements.


## Model selectors
### `minecraft:range_dispatch`
> [!CAUTION]
>
> This feature is experimental. It could be removed depending what direction Mojang takes item models in the future.

The vanilla `range_dispatch` has been extended so that its entries may be automatically populated based on available models, similarly to how variant-Cit's modules work.

The selector takes a new optional field `variants-cit:baseId`, which is a namespaced identifier.
If set, the mod will look for models whose ID are the given value value followed by a number. Those models will be added to the selector's entries, using the found number as threshold.

The vanilla `entries` field is still strictly required.

## Making modded assets vanilla-compatible 
> [!CAUTION]
>
> This feature is experimental. It could be removed depending on what direction Mojang takes item models in the future.

Item states that use modded properties may become completely unusable in vanilla. Variants-Cit provides a mechanism to provide a vanilla fallback to these.

Every model selector now take a new optional field `variants-cit:override`, its value is another model selector. If Variants-Cit is installed, the selector specified there will completely override its parent. In vanilla minecraft, this field will be ignored.

```json
{
	"model": {
		"//": "Uses this model in vanilla minecraft",
		"type": "model",
		"model": "...",

		"//": "Uses this model instead when variants-CIT is installed"
		"variants-cit:override": {
			"type": "range_dispatch",
			"property": "variants-cit:stored_enchantment_level",
			"//": "..."
		}
	}
}
```
