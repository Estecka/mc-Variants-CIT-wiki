# Module Types

This page describes the behaviour of every built-in types to use in [module configurations](Module-Configuration). See [model prefixes](Module-Configuration#modelPrefix) in particular, to understand how variants are associated with textures and models.

Some modules are designed around specific item types, but can still be used on any item, so long as they provides the required component.

## `axolotl_variant`
Derives the item variant from the `Variant` element in the `bucket_entity_data` component. The component is used by multiple types of bucket-of-mob, but this CIT module only ever returns the ID of axolotl variants.

Axolotl variants are not inherently identifier-based, but this modules converts them to a namespaced identifier. It will ignore variants that are not stored in number format.

As of writting, available variant IDs are, in order: `lucy`, `wild`, `gold`, `cyan`, `blue`. All those ids will use `minecraft:` as their namespace.

## `custom_data`, `entity_data`, `bucket_entity_data`, `block_entity_data`
> [!TIP]
>
> These item components also have more specialised modules: `axolotl_variant` and `painting_variant`.

Copies the variant from either a **string** or a **number** stored somewhere in the nbt component with matching name.
For numbers, the namespace will always be "`minecraft:`".


### Parameters:
- `nbtPath`: *Mandatory, String.* The path to the variant ID, with dot '`.`' separated elements.
- `caseSensitive`: *Optional, boolean, default to True.* If set to false, will convert the strings to lower-case. This should be reserved for when you have no control over the data; if you are the data designer, consider limiting your identifiers to the character set `[a-z0-9_-.]`, like for vanilla identifier.

Example:
```json
{
	"type": "custom_data",
	"modelPefix": "item/cutomItemType/",
	"parameters": {
		"nbtPath": "path.to.variant"
	}
}
```

## `custom_name`
Derives the variant from the `custom_name` component.

The name itself is used as the variant, with illegal characters being either removed or converted:
Uppercases are converted to lowercases, accents are stripped off, spaces '` `' are converted to underscores '`_`', and all other invalid characters are completely removed.

E.g: "Épée de l'End" -> `minecraft:epee_de_lend`

Special formatting, such as colour and boldness, are ignored.

### Parameters:
- `debug`: *Optional, defaults to false*. Prints name-to-variants conversion into the log, whenever a new one is computed.
- `specialNames`: *Optional, Maps Strings to Identifiers.* Lets you hardcode some associations between names and variant ID, instead of letting the module compute them automatically like above. (This parameter is a relica from older versions, and will not be required in most cases.)

## `enchantment`
Copies the variant from a single enchantment in the item component `enchantments`, used by enchanted tools and armours.
This will prioritize enchantments that, in order: have available models, have the largest exclusive set, or have the highest level.

While one module can only work with a single enchantment, it's still possible to create CITs based on multiple enchantments, by using multiple modules with different priorities and parameters.

### Parameters:
- `requiredEnchantments`: *Optional, maps Identifiers to Integers.* The module will only be applied to items that have enough levels in the specified enchantments. Enchantments listed here are completely excluded Copies being as the item variant.

### Example:
This module will manage CITs that only have a single enchantment:
```json
{
	"type": "enchantment",
	"modelPrefix": "item/diamond_sword/"
}
```
This additional module will only apply to items with the `fire_aspect` enchantement, and provide a CIT based on the second enchantment. (See [module priorities](Module-Configuration#priority).)
```json
{
	"type": "enchantment",
	"priority": 100,
	"modelPrefix": "item/fire_aspected_diamond_sword/",
	"parameters": {
		"requiredEnchantments": { "fire_aspect": 1 }
	}
}
```

## `instrument`
Copies the variant from the item component `instrument`, used by goat horns.

Note that all vanilla instruments have their name suffixed with `_goat_horn`; you'll need that suffix in your CITs *in addition* to your model prefix.

## `jukebox_playable`
Copies the variant from the item component `jukebox_playable`, used by music discs.

## `painting_variant`
Copies the variant from the `variant` element in the `entity_tag` component, used by paintings in the creative inventory.

The special model `invalid` will be applied to painting variants that do not exist with the current datapacks.

## `potion_type`
Derives the variant from the item component `potion_contents`, used by potions, splash potions, and lingering potions. This specifically looks at the potion's "type", and not the actual status effects.

## `stored_enchantment`
Copies the variant from a single enchantment in the item component `stored_enchantments`, used by enchanted books.

If the item has **exactly one enchantment**, the variant will be the ID of that enchantment. If the item contains multiple enchantments, it will instead use the special model `multi`.

The enchantment level can optionally be included in the variant ID.

### parameters:
- `levelSeparator`: _Optional, String._ If set, the module will add the enchantment level to the end of the variant ID, using the given separator. The separator can be an empty string. If no asset was provided for this specific level, the module will fall back to lower level CITs. Finally, if none could be found, the module will fall back to level-invariant CITs.

## `trim`
Combines the pattern and material from the `trim` component into a single identifier, in a way that imitates the vanilla format for trimmed armour models.
The resulting variant is: `<pattern_namespace>:<pattern_path>_<material_path>`.

E.g: `minecraft:sentry_diamond`

**Only the namespace of the pattern is used. The material's namespace is discarded.**


## `trim_pattern`, `trim_material`
Uses either the pattern or the material of the `trim` component as a variant.
