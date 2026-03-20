# Module Types

A module's type defines how its item's variant ID will be computed. See [modelPrefix](./Module-Configuration#field-modelparent) to learn how variant IDs are linked to a texture or model.

Each module type is its own separate system. The `"parameters"` of one type do not apply to the others, unless explicitely specified.

Existing types can be thought of as belonging to either of two families:  
The purpose-made types require no parameters to be functional, but are tailored for specific and common use cases. When they do have parameters, those are optional, and only there for fine-tuning.  
The generalist types are more flexible, but require a lot more configuration to achieve basic results. Some purpose-made functionalities can be difficult to replicate through generalist modules.

### Outline
- Generalist Modules
  - [`group`](#module-group)
  - [`component_data`](#module-component_data)
  - [`component_format`](#module-component_format)
  - [`component_threshold`](#module-component_threshold)
  - [`predicates`](#module-predicates)
- Purpose-made Modules
  - [`axolotl_variant`](#module-axolotl_variant)
  - [`custom_name`](#module-custom_name)
  - [`durability`](#module-durability)
  - [`enchantment`](#module-enchantment-stored_enchantment), [`stored_echantment`](#module-enchantment-stored_enchantment)
  - [`enchantment_vector`](#module-enchantment_vector-stored_enchantment_vector), [`stored_echantment_vector`](#module-enchantment_vector-stored_enchantment_vector)
  - [`instrument`](#module-instrument)
  - [`item_count`](#module-item_count)
  - [`jukebox_playable`](#module-jukebox_playable)
  - [`painting_variant`](#module-painting_variant)
  - [`potion_type`](#module-potion_type)
  - [`trim`](#module-trim)
  - [`trim_pattern`](#module-trim_pattern-trim_material), [`trim_material`](#module-trim_pattern-trim_material)


# Generalist modules

## Module: `group`
A group module is syntaxic sugar for creating multiple modules with similar configurations.  
The submodules share most of the options defined at the root. They can define an additional preconditions, and have a model prefix that is relative to the parent's.

Sub-modules are evaluated in the order they are defined. It is functionally identical to using multiple modules with different priorities, but using a group module yields better performances than creating separate modules with different priorities.

### Parameters: 
- **`submodules`**: *Mandatory array of submodules.* Each submodule can have the following fields:
	- **`type`**: Same as [`type`](./Module-Configuration#field-type) in regular modules
	- **`parameters`**:  Same as [`parameters`](./Module-Configuration#field-parameters) in regular modules
	- **`precondition`**: Same as [`precondition`](./Module-Configuration#field-precondition) in regular modules.
	- **`modelPrefix`**: Similar to a module's regular model prefix, but optional, and will be combined with the parent group's model prefix. The variant IDs computed by the submodule will be resolved against the combined model prefixes.

###	Example:
Handles an hypothetical modded enchanted book that is restricted to specific tool tiers, and uses a separate set of textures for each tier.
```jsonc
{
	"items": "enchanted_book",
	"modelPrefix": "typed_books/",

	"...": "...",

	"type": "group",
	"parameters": {
		"submodules": [
			{
				"type": "stored_enchantment_vector",
				"modelPrefix": "stone/", // => typed_books/stone/
				"precondition": { "custom_data.allowed_tool_type": "stone" }
			},
			{
				"type": "stored_enchantment_vector",
				"modelPrefix": "iron/", // => typed_books/iron/
				"precondition": { "custom_data.allowed_tool_type": "iron" }
			}
		]
	}
}
```


## Module: `component_data`

Pulls a single piece of data from a given component or property, and uses that data as the variant ID.

This module will only look for exact match. Use [`component_treshold`](#module-component_threshold), for numeric data that can fall back to a nearby value.

### Parameters:
- All fields from [Item Properties](./Item-Properties).

### Example:
```json
{
	"type": "component_data",
	"items": "diamond_sword",
	"modelPrefix": "hypixel/diamond_swords/",
	"parameters": {
		"componentType": "custom_data",
		"nbtPath": ".id",
		"transform": "lowercase"
	}
}
```

## Module: `component_format`
A step up from `component_data`, that can pull and combine multiple pieces of data from multiple properties at once. 

### Parameters:
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

## Module: `component_threshold`

Pulls a single piece of numeric data from a given component, and uses that data as the variant ID. Floating-point numbers are accepted, but are truncated to an integer.
If no exact match exists, the module will fallback to either a lower or higher value model, as configured.

Despite using a syntax that looks similar to `component_data`, this module does not support Item Properties, Expect, Transforms etc. `nbtPath` and `componentType` are the only parameters they have in common.

### Parameters:
- **`namespace`**: _Optional String, defaults to `"minecraft"`_
The namespace that will contain all the models.
- **`componentType`**: _Mandatory Identifier_. The component that the data will be pulled from.
- **`nbtPath`**: _Optional String._ The location of the data within the component. If left unspecified, the component as a whole is used.
- **`modelRange`**: _Mandatory String._ Describes the set of models that can be used, relative to the value found on the item. Possible values are: `"strictly_equal"`, `"lesser_or_equal"`, `"greater_or_equal"`
- **`scale`**, **`offset`**: _Optional floats, default to `1` and `0` respectively._ Applies a linear function to the data found on the item:  
  `variant_id = trunc((data * scale) + offset)`


## Module: `predicates`
Lets you manually configure every single variant, and the precise conditions required for each variant to apply.

This module is the closest to how Optifine CIT works, but is also the most verbose, and its performances will degrade faster as you add more variants to it.
(Most modules have a complexity of O(1) or O(Log N), but `predicates` has a complexity of O(N).)  
Its usage should only be considered when few variants are required, or as a band-aid solution to patch holes that regular modules do not cover.  
If you find yourself needing to use this module type with large amounts of variants, please [open an issue](https://github.com/Estecka/mc-Variants-CIT/issues) to describe your use case.


### Parameters
- **`predicates`**: *Mandatory array of variants.* Each variant is evaluated in teh order they are defined, and the first match returns immediately. Each variant takes the following fields:
	- **`variantId`**: *Mandatory identifier.* The variant ID that will be returned if the item matches the associated predicate.
	- **`precondition`**: *Mandatory precondition.* The condition that the item must match in order to be associated with the aforementioned variant ID.
	This follows the same syntax as a module's global precondition. See: [Preconditions](./Precondition%20Cheat-Sheet) and [Transforms](./Item-Properties#transforms).

### Example:
```jsonc
{
	"items": "apple",
	"modelPrefix": "apples_set/",

	"types": "predicates",
	"parameters": {
		"predicates":
		[
			{
				"variantId": "boatloads_of",
				"precondition": {
					"item_count": { "greater_than": 32 }
				}
			},
			{
				"variantId": "lots_of",
				"precondition": {
					"item_count": { "greater_than": 8 }
				}
			}
		]
	}
}
```


# Purpose-made modules
These modules were designed around specific use cases and item types. They can still be used on any item, so long as it provides the required component.

## Module: `axolotl_variant`
Derives the variant id from the color of an axolotl in a bucket, and optionally its age. Target components and data vary with the version of Minecraft.

Colours available in vanilla are: `lucy`, `wild`, `gold`, `cyan`, `blue`. All are in the `minecraft` namespace.

If a baby model is missing the module will fallback to the adult model of the same colour.

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

> [!TIP]
>
> This historical module type a has very limited set of options. It is easily replicated though [`component_data`](#component_data) which will give you access to Regex matching and substitutions.

### Parameters:
- **`specialNames`**: *Optional, Maps Strings to Identifiers.* Lets you hardcode some associations between names and variant ID, instead of letting the module compute them automatically like above. (This parameter is a relica from older versions, and will not be required in most cases.)


## Module: `durability`
Uses the item's remaining durability as variant. The module will use the closest model whose value is greater than or equal to the item's durability.

Durability is based on the `damage` and `max_damage` components, but only `max_damage` is required for the module to apply. Items without a `damage` component are treated as having full durability.

### Parameters:
- **`namespace`** _Optional string, defaults to `"minecraft"`._ The namespace containing all the models.
- **`scale`**: *Optional positive integer, defaults to the item's maximum durability.* The range of values to use. For example, with a scale of 100, the variant will be the percentage of durability, instead of the raw durability.


## Module: `enchantment`, `stored_enchantment`
Derives the variant ID from a single enchantment on the item, optionally including the enchantment's level. Tools should use `enchantments`, and books should use `stored_enchantment`.

> [!TIP]
> 
> **This historical module was designed with single-enchantment textures in mind,** its ability to handle multiple enchantments is limited, and requires creating multiple modules. **Prefer using `enchantment_vector` instead.**

This will prioritize enchantments that, in order: have available models, have the largest exclusive set, or have the highest level.

If the special model "`multi`"  is defined, it will be used on items with more than one enchantments. This only counts enchantments that are not defined in the `requiredEnchantment` parameter.

Although one module can only work with a single enchantment, it is still possible to create CITs based on multiple enchantments, by using multiple modules with different priorities and `requiredEnchantments` parameters.

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
This additional module will only apply to items with the `fire_aspect` enchantement, and provide a CIT based on a second enchantment. (See [module priorities](Module-Configuration#field-priority).)
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

Picks a model based on *all* of the item's enchantments and their levels.
The module will pick a model whose levels are lower or equal to those of the item. By default, models that have the most levels in total will be prioritized.

This module does not compute a variant ID for the item. Instead it takes the variant IDs of the *models*, and converts those into sets of enchantments.
You can use the [summary](./Troubbleshooting#command-summary) command to quickly check the list of unique enchantments that are presents on your models; you can easily spot subtle filename errors by looking for enchantment names you know should not exist. You can use the [dump](./Troubbleshooting#command-dump) to get a detailled list of enchantments for every model.

### Model Name Syntax
The default syntax should be able to support all vanilla enchantments without creating ambiguous names. It assumes enchantment names will never contain two successive underscores (`"__"`), never end with a number, and never have namespaces that contain two successive dots (`".."`).
If you find an enchantment that breaks those rules, you can tweak the syntax or rename enchantments in the module's parameters.

The name of a model must contain the ID of all its enchantments, separated by **TWO** underscores. The order of the enchantments does not matter:

`fire_aspect__sweeping_edge.png`

To require a specific level, just add a number after the enchantment's name. (No level implies lvl.1)

`fire_aspect2__mending.png`

For enchantments in modded namespaces, include the namespace in the enchantment name, but use two dots (`".."`) as a separator instead of a colon. (The namespace of the models themselves never changes, and can be defined in the parameters.)

`fire_aspect2__illagerplus..illager_bane__mending.png`

In order to shorten your filenames, you can define aliases in the module's parameters :
```json
"parameters": {
	"enchantAliases":{
		"mend":  "mending",
		"fire":  "fire_aspect",
		"sweep": "sweeping_edge",
		"ill":   "illagerplus:illager_bane"
	}
}
```
With this, `fire2__sweep3__ill7__mend.png` would then be short for :  
`fire_aspect2__sweeping_egde3__illagerplus..illager_bane7__mending.png`

### Parameters:
All parameters for this module are optional.
- **`namespace`**: _Optional String. Defaults to `"minecraft"`._
  The namespace that will contain all the models. (Enchantments have no effect on the namespace of models.)
- **`enchantSeparator`**: _Optional String. Defaults to `"__"`._
  The separator to use between enchantments in model names. This string cannot be empty.
- **`levelSeparator`**: _Optional String._
  The separator to use between enchantments names and their levels. This defaults to an empty string if `optionalLevel` is enabled.
- **`optionalLevel`**: _Optional Boolean. Defaults to `true`._
  Whether enchantment levels are optional. When disabled, if `levelSeparator` is set, levels will be required; if `levelSeparator` is unset, levels will be unsupported.
- **`enchantAliases`**: _Optional, maps Identifiers to Identifiers._
  If a model's name describes an enchantment present in this map's key, it will be treated as the corresponding value instead. This allows giving multiple aliases to the same enchantment, and does not prevent the enchantment's original name from being used.
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

Uses the item count as the variant. If no model matches the exact count, it will automatically fall back to lower-value model.

The namespace of the variant id is fixed, but configurable.

### Parameters:
- `namespace` _Optional string, defaults to `"minecraft"`._ The namespace to use for the variant id.

## Module: `jukebox_playable`
Copies the variant from the item component `jukebox_playable`, used by music discs.

## Module: `painting_variant`
Copies the variant ID a painting's ID, useable on paintings in the creative inventory.
Target component varies depending on the version of Minecraft.

The special model `invalid` will be applied to painting variants that do not exist with the current datapacks. This is no longer possible in MC 1.21.5, where painting items can no longer hold invalid variants.

## Module: `potion_type`
Copies the variant ID from the item component `potion_contents`, used by potions, splash potions, and lingering potions. This specifically looks at the potion's "type", and not the actual status effects.

## Module: `trim`
Combines the pattern and material from the `trim` component into a single identifier, in a way that imitates the vanilla format for trimmed armour models.
The resulting variant is: `<pattern_namespace>:<pattern_path>_<material_path>`.

E.g: `minecraft:sentry_diamond`

**Only the namespace of the pattern is used. The material's namespace is discarded.**

## Module: `trim_pattern`, `trim_material`
Uses either the pattern or the material of the `trim` component as a variant.
