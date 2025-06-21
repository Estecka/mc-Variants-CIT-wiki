# Module Types

This page describes the behaviour of every built-in types to use in [module configurations](Module-Configuration). See [model prefixes](Module-Configuration#modelPrefix) in particular, to understand how variants are associated with textures and models.

A module's type defines how its item's variant ID will be computed.
Each type has its own separate system, so the `"parameters"` of one type do not apply to the others, unless explicitely specified.

Existing types can be thought of as belonging to either of two families:  
The purpose-made types require no parameters to be functional, but are tailored for specific and common use cases. When they do have parameters, those are usually optional, and only there for fine-tuning.  
The generalist types are more flexible, but require a lot more configuration to achieve basic results. Those are a relatively recent addition, and there are still some purpose-made functionalities that cannot be replicated using generalist modules (e.g: Enchantment priority and numeric value thresholds).

# Generalist modules

## `component_data`

Pulls a single piece of data from a given component or property, and uses that data as the variant ID.

The module will fail if the item does not have the component, or valid data is missing from that component at that path.

### Parameters:
- `debug`: *Optional boolean, defaults to `false`.* Causes variant IDs encountered by the module to be printed in the log.
- All parameters from [Item Properties](./Item-Properties).

### Example:
```json
{
	"type": "component_data",
	"items": "minecraft:firework_rocket",
	"modelPrefix": "item/firework_rocket/flight_",
	"parameters": {
		"debug": true,
		"componentType": "lore",
		"nbtPath": "[0]",
		"expect": "rich_text",
		"transform": "sanitize_path"
	}
}
```

## `component_format`
A step up from `component_data`, that can pull and combine multiple pieces of data from multiple properties at once. 

The module will fail if at least one piece of data is missing or invalid.

### Parameters:
- `debug`: *Optional boolean, defaults to `false`.* Causes variant IDs encountered by the module to be printed in the log.
- `format`: *Mandatory, string*. The format of the variant ID. The string can contain variables formatted as `${name}`, which will be substitued with the data extracted from the components. Variable names can contain any character in `[a-zA-Z0-9_]`, but may not start with a number.
- `variables`: *Mandatory, Maps variable names to [Item Properties](./Item-Properties)*. Indicates where and how to get the data for each variable in the format.

### Example:
This behaves similarly to the [`trim`](./Module-Types#trim) module type, but can handle multiple item types at once, without mixing their respective CITs:
```json
{
	"type": "component_format",
	"items": [ "diamond_pickaxe", "iron_axe", "netherite_sword", "..."],
	"modelPrefix": "item/trimmed_",
	"parameters": {
		"debug": true,
		"format": "${patternSpace}:${item}/${patternPath}_${material}",
		"variables": {
			"patternSpace": {
				"componentType": "trim",
				"nbtPath": ".pattern",
				"transform": "discard_path"
			},
			"item": {
				"property": "item_type",
				"transform": "discard_namespace"
			},
			"patternPath": {
				"componentType": "trim",
				"nbtPath": ".pattern",
				"transform": "discard_namespace"
			},
			"material": {
				"componentType": "trim",
				"nbtPath": ".material",
				"transform": "discard_namespace"
			}
		}
	}
}
```
Example variant id: `minecraft:netherite_sword/sentry_diamond`

Corresponding texture/baked model id: `minecraft:item/trimmed_netherite_sword/sentry_diamond`


# Purpose-made modules
These modules were designed around specific use cases and item types. They can still be used on any item, so long as it provides the required component.

Some of these modules have extra functionalities that cannot be reproduced using generalist modules.

## `axolotl_variant`
Constructs a variant id from the color of the Axolotl, and optionally its age. Target components and data vary depending on the version of Minecraft.

As of writing, available colours are: `lucy`, `wild`, `gold`, `cyan`, `blue`. All are in the `minecraft` namespace.

If a baby CIT is missing the module will fallback to the adult CIT of the same colour.

Example variant ids:
- `minecraft:lucy` Default for all ages.
- `minecraft:lucy_baby` Default for babies.
- `minecraft:lucy_adult` If `adultSuffix` is set to `_adult`.

### parameters
- `adultSuffix`: *Optional, defaults to an empty string*.
- `babySuffix`: *Optional, defaults to `"_baby"`*.

## `custom_name`
Derives the variant from the `custom_name` component.

The name itself is used as the variant, with illegal characters being either removed or converted:
Uppercases are converted to lowercases, accents are stripped off, spaces '` `' are converted to underscores '`_`', and all other invalid characters are completely removed.

E.g: "Épée de l'End" -> `minecraft:epee_de_lend`

Special formatting, such as colour and boldness, are ignored.

### Parameters:
- `debug`: *Optional, defaults to false*. Prints name-to-variants conversion into the log, whenever a new one is computed.
- `specialNames`: *Optional, Maps Strings to Identifiers.* Lets you hardcode some associations between names and variant ID, instead of letting the module compute them automatically like above. (This parameter is a relica from older versions, and will not be required in most cases.)


## `durability`
Uses the item's remaining durability as variant. The module will use the closest CIT with a value greater or equal to the item's durability.

Durability is based on the `damage` and `max_damage` components, but only `max_damage` is required for the module to apply. Items without a `damage` component are treated as full durability.

### Parameters:
- `namespace` _Optional string, defaults to `"minecraft"`._ The namespace to use for the variant id.
- `scale`: *Optional positive integer, defaults to the item's maximum durability.* The range of values to use. For example, with a scale of 100, the variant will be the precentage of durability, instead of the raw durability.


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

## `item_count`

Uses the item count as the variant. If no CIT matches the exact count, it will automatically fall back to lower-value CITs.

The namespace of the variant id is fixed, but configurable.

### Parameters:
- `namespace` _Optional string, defaults to `"minecraft"`._ The namespace to use for the variant id.

## `jukebox_playable`
Copies the variant from the item component `jukebox_playable`, used by music discs.

## `painting_variant`
Uses the painting id as the variant id, used by paintings in the creative inventory.
Target component varies depending on the version of Minecraft.

The special model `invalid` will be applied to painting variants that do not exist with the current datapacks. (No longer possible in MC 1.21.5)

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
