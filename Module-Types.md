# Module Types

This page describes the behaviour of every built-in types to use in [module configurations](Module-Configuration). See [model prefixes](Module-Configuration#modelPrefix) in particular, to understand how variants are associated with textures and models.

A module's type defines how its item's variant ID will be computed.
Each type has its own separate system, so the `"parameters"` of one type do not apply to the others, unless explicitely specified.

Existing types can be thought of as belonging to either of two families:  
The purpose-made types require no parameters to be functional, but are tailored for specific and common use cases. When they do have parameters, those are usually optional, and only there for fine-tuning.  
The generalist types are more flexible, but require a lot more configuration to achieve basic results. Those are a relatively recent addition, and there are still some purpose-made functionalities that cannot be replicated using generalist modules (e.g: Enchantment priority and numeric value thresholds).

# Generalist modules

## Module: `component_data`

Pulls a single piece of data from a given component or property, and uses that data as the variant ID.

The module will fail if the item does not have the component, or valid data is missing from that component at that path.

### Parameters:
- **`debug`**: *Optional boolean, defaults to `false`.* Causes variant IDs encountered by the module to be printed in the log.
- All fields from [Item Properties](./Item-Properties).

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

## Module: `component_format`
A step up from `component_data`, that can pull and combine multiple pieces of data from multiple properties at once. 

The module will fail if at least one piece of data is missing or invalid.

### Parameters:
- **`debug`**: *Optional boolean, defaults to `false`.* Causes variant IDs encountered by the module to be printed in the log.
- **`format`**: *Mandatory, string*. The format of the variant ID. The string can contain variables formatted as `${name}`, which will be substitued with the data extracted from the components. Variable names can contain any character in `[a-zA-Z0-9_]`, but may not start with a number.
- **`variables`**: *Mandatory, Maps variable names to [Item Properties](./Item-Properties)*. Indicates where and how to get the data for each variable in the format.

### Example:
This behaves similarly to the [`trim`](./Module-Types#module-trim) module type, but can handle multiple item types at once, without mixing their respective CITs:
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

## Module: `axolotl_variant`
Constructs a variant id from the color of the Axolotl, and optionally its age. Target components and data vary depending on the version of Minecraft.

As of writing, available colours are: `lucy`, `wild`, `gold`, `cyan`, `blue`. All are in the `minecraft` namespace.

If a baby CIT is missing the module will fallback to the adult CIT of the same colour.

Example variant ids:
- `minecraft:lucy` Default for all ages.
- `minecraft:lucy_baby` Default for babies.
- `minecraft:lucy_adult` If `adultSuffix` is set to `_adult`.

### parameters
- **`adultSuffix`**: *Optional, defaults to an empty string*.
- **`babySuffix`**: *Optional, defaults to `"_baby"`*.

## Module: `custom_name`
Derives the variant from the `custom_name` component.

The name itself is used as the variant, with illegal characters being either removed or converted:
Uppercases are converted to lowercases, accents are stripped off, spaces '` `' are converted to underscores '`_`', and all other invalid characters are completely removed.

E.g: "Épée de l'End" -> `minecraft:epee_de_lend`

Special formatting, such as colour and boldness, are ignored.

### Parameters:
- **`debug`**: *Optional, defaults to false*. Prints name-to-variants conversion into the log, whenever a new one is computed.
- **`specialNames`**: *Optional, Maps Strings to Identifiers.* Lets you hardcode some associations between names and variant ID, instead of letting the module compute them automatically like above. (This parameter is a relica from older versions, and will not be required in most cases.)

> [!TIP]
>
> This historical module type has very limited set of options. Using [`component_data`](#component_data) instead will give you access to Regex matching and substitutions.



## Module: `durability`
Uses the item's remaining durability as variant. The module will use the closest CIT with a value greater or equal to the item's durability.

Durability is based on the `damage` and `max_damage` components, but only `max_damage` is required for the module to apply. Items without a `damage` component are treated as full durability.

### Parameters:
- **`namespace`** _Optional string, defaults to `"minecraft"`._ The namespace to use for the variant id.
- **`scale`**: *Optional positive integer, defaults to the item's maximum durability.* The range of values to use. For example, with a scale of 100, the variant will be the precentage of durability, instead of the raw durability.


## Module: `enchantment`, `stored_enchantment`
Derives the variant ID from one of the enchantment on the item, optionally including the enchantment's level. Tools should use `enchantments`, and books should use `stored_enchantment`.

> [!TIP]
> 
> **This module was designed with single-enchantment CITs in mind,** its ability to handle multiple enchantments is limited, and requires creating multiple modules. **Consider using `enchantment_vector` instead for CITs based on multiple enchantments.**

This will prioritize enchantments that, in order: have available models, have the largest exclusive set, or have the highest level.

If the special model "`multi`"  is defined, it will be used on items with more than one enchantments. This will only count enchantments that are not defined in the `requiredEnchantment` parameter.

While one module can only work with a single enchantment, it is still possible to create CITs based on multiple enchantments, by using multiple modules with different priorities and `requiredEnchantments` parameters.

### Parameters:
- **`levelSeparator`**: _Optional, String._ If set, the module will add the enchantment level at the end of the variant ID, using the given separator. The module will fall back to lower-level CITs, if it can't find one that matches the item's exact level. Finally, if none could be found, the module will fall back to level-invariant CITs.
- **`requiredEnchantments`**: *Optional, maps Identifiers to Integers.* The module will only be applied to items that have enough levels in the specified enchantments. Enchantments listed here will never be selected as the variant ID.

### Example:
This module will manage CITs that only have a single enchantment:
```json
{
	"type": "enchantment",
	"modelPrefix": "item/diamond_sword/"
}
```
This additional module will only apply to items with the `fire_aspect` enchantement, and provide a CIT based on a second enchantment. (See [module priorities](Module-Configuration#priority).)
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

## Module: `enchantment_vector`, `stored_enchantment_vector`

> ⚠ This module is exclusive to VCIT v3/v4 (MC 1.21.4+)

Picks a model based on *all* of the item's enchantments and their levels.
The module will pick a model whose levels are lower or equal to those of the item. By default, models that have the most levels in total will be prioritized.

This module does not compute a variant ID for the item. Instead it takes the variant IDs of the *models*, and converts those into sets of enchantments.  For development, you probably want to enable `bakingDebug` in the module's parameters, which will print extra infos in the log during resource reload.

### Model Name Syntax
The default syntax should be able to support all vanilla enchantments without creating ambiguous names. It assumes enchantment names will never contain two successive underscores (`"__"`), never end with a number, and never have namespaces that contain two successive dots (`".."`).
If you find an enchantment that breaks those rules, you can tweak the syntax or rename enchantments in the module's parameters.

The name of a model must contain the ID of all its enchantments, separated by **TWO** underscores. The order of the enchantments does not matter:

`fire_aspect__sweeping_edge.png`

To require a specific level, just add a number after the enchantment's name. (No level implies lvl.1)

`fire_aspect2__mending.png`

For enchantments in modded namespaces, include the namespace before the name as usual, but use two dots (`".."`) as a separator instead of a colon. The namespace of the models never changes, and can be defined in the parameters.

`fire_aspect2__illagerplus..illager_bane__mending.png`

In order to shorten your filenames, you can define aliases in the module's parameters :
```json
"parameters": {
	"enchantAliases":{
		"sharp": "sharpness",
		"fire":  "fire_aspect",
		"sweep": "sweeping_edge",
		"ill":   "illagerplus:illager_bane"
	}
}
```
`fire2__sweep3__ill7__mending.png` would then be short for :  
`fire_aspect2__sweeping_egde3__illagerplus..illager_bane7__mending.png`


### Parameters:
- **`bakingDebug`**: _Optional Boolean. Defaults to `false`._
  During resource reload, the module will print into the log the list of unique enchantments it detected in your CITs, and thinks are valid. You can easily spot subtle filename errors by looking for enchantment names you know should not exist.
- **`runtimeDebug`**: _Optional Boolean. Defaults to `false`._
  Whenever the module recomputes an item's model, it will print that model's name into the log.
- **`namespace`**: _Optional String. Defaults to `"minecraft"`._
  The namespace that will contain all the models. (Enchantments have no effect on the namespace of models.)
- **`enchantSeparator`**: _Optional String. Defaults to `"__"`._
  The separator to use between enchantments in model names. This string cannot be empty.
- **`levelSeparator`**: _Optional String._
  The separator to use between enchantments names and their levels. This defaults to an empty string if `optionalLevel` is enabled.
- **`optionalLevel`**: _Optional Boolean. Defaults to `true`._
  Whether enchantment levels are optional. When disabled, if `levelSeparator` is set, levels will be required; if `levelSeparator` is unset, levels will be unsupported.
- **`enchantAliases`**: _Optional, maps Identifiers to Identifiers._
  If a model's name describes an enchantment present in this map's key, it will use the corresponding value instead. This allows giving multiple aliases to the same enchantment, and does not prevent other models from using the enchantment's original name.
- **`ordering`**: _Optional array of Strings. Defaults to `["taxicab", "dimension"]`._
  Defines what models are prioritized when no perfect match exists. The first comparator matters the most, the others will be used as tie-breakers.
	- `"taxicab"`: Prioritizes models with the most levels in total.
	- `"dimension"`: Prioritizes models with the most enchantments, regardless of levels.
	- `"maximum"`:   Prioritizes models with the highest single level.
	- `"euclidian"`: Prioritizes models with the highest sum of squared levels.


## Module: `instrument`
Copies the variant from the item component `instrument`, used by goat horns.

Note that all vanilla instruments have their names suffixed with `_goat_horn`. You will need to include that in the filenames of your models.

## Module: `item_count`

Uses the item count as the variant. If no CIT matches the exact count, it will automatically fall back to lower-value CITs.

The namespace of the variant id is fixed, but configurable.

### Parameters:
- `namespace` _Optional string, defaults to `"minecraft"`._ The namespace to use for the variant id.

## Module: `jukebox_playable`
Copies the variant from the item component `jukebox_playable`, used by music discs.

## Module: `painting_variant`
Uses the painting id as the variant id, used by paintings in the creative inventory.
Target component varies depending on the version of Minecraft.

The special model `invalid` will be applied to painting variants that do not exist with the current datapacks. (No longer possible in MC 1.21.5)

## Module: `potion_type`
Derives the variant from the item component `potion_contents`, used by potions, splash potions, and lingering potions. This specifically looks at the potion's "type", and not the actual status effects.

## Module: `trim`
Combines the pattern and material from the `trim` component into a single identifier, in a way that imitates the vanilla format for trimmed armour models.
The resulting variant is: `<pattern_namespace>:<pattern_path>_<material_path>`.

E.g: `minecraft:sentry_diamond`

**Only the namespace of the pattern is used. The material's namespace is discarded.**


## Module: `trim_pattern`, `trim_material`
Uses either the pattern or the material of the `trim` component as a variant.
