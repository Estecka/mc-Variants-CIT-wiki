# Tutorial: Pattern-Seeking Modules
## Prerequisites
This tutorial assumes you already have some familiarity with the mod, or have at least followed the introductory tutorial:
- [Getting Started](./Getting-Started)

## Foreword

This tutorial focuses mostly on the module type `component_data`. However its basest principles applies to almost every other module types other than `predicates`.

There is little those modules can do that `predicates` can't; so feel free to skip this tutorial if you plan on sticking to `predicates`. Using other module types is mostly a matter of optimization.

## Index
- [Typical use case](#typical-use-case)
- [Best Practices](#best-practices)
	- [AssetGen Conflicts](#assetgen-conflicts)
	- [Namespacing your assets](#namespacing-your-assets)
- [Manipulating names with `component_data`](#manipulating-names-with-component_data)
	- [Pattern-matching](#pattern-matching)
	- [Non-Latin alphabets](#names-in-non-latin-alphabets)
- [Using custom data as variants](#using-custom-data-as-variants)
	- [Locating the data](#locating-the-data)
	- [Combining multiple pieces of data](#combining-multiple-pieces-of-data)

## Typical use case
If you just came out of the Getting Started tutorial, you should have a module that looks something like this:

```jsonc
{
	"item": "diamond_sword",

	"type": "predicates",
	"parameters": {
		"predicates": [
			{
				"modelId": "custom_swords/epee_de_lend",
				"precondition": { "custom_name": "Épée de l'End" }
			},
			{
				"modelId": "custom_swords/excalibur",
				"precondition": { "custom_name": "Excalibur" }
			},
			{
				"modelId": "custom_swords/durandal",
				"precondition": { "custom_name": "Durandal" }
			}
			// etc
		]
	}
}
```

There's a clear pattern in how these variants are defined: They all use the custom name as a source of data, and the model ID mirrors the custom name.  
You can still add new variants to this module in this way. But if you have dozens or even hundreds of names, you might be interested in using different module types, which can automatically do the mapping between variant and model.

For example, the module above could be compressed to this:

```jsonc
{
	"item": "diamond_sword",
	"modelPrefix": "custom_swords/",
	"type": "custom_name"
}
```

... and that's literally it. No list of names or models at all.

This kind of modules works by computing a **Variant ID** for an item, then checking whether there exists a model whose name matches the variant ID. Here the module type `custom_name` takes the name of the item, and sanitizes it into a valid identifier:  
"`Épée de l'End`" => "`epee_de_lend`".  
This variant ID is then combined with the Model Prefix, giving the model ID: "`custom_swords/epee_de_lend`".
This works for any name. Adding new variants to the module is done by simply adding new files to the "`custom_swords/`" folder.

**Using the [`walkthrough`](./Troubbleshooting#command-walkthrough) command is particularly useful with this kind of module.** If you are not sure what you should name your files, or what Variant ID your module is outputing for a given item, this command will tell you.

While `predicates` can do pretty much anything, the other module types are much more specialized, so you may will different ones for different use cases. Some of the most notable ones are `item_count`, `durability`, `enchantment_vector` and `component_data/_format`, with the later being the most flexible.

[Full list of module types](./Module-Types).


## Best Practices
### Assetgen Conflicts
Modules that use `modelPrefix` are very suceptible to an issue called "AssetGen conflict". The typical example of which looks like this:
```jsonc
// Module 1
{
	"type": "custom_name",
	"item": "spear",

	"assetGen": "item_model/spear",
	"modelPrefix:": "weapons/"
}

// Module 2
{
	"type": "custom_name",
	"item": "diamond_sword",

	"assetGen": "item_model/handheld",
	"modelPrefix:": "weapons/"
}
```
Both modules have the same `modelPrefix`, but different `assetGen` presets.  
The two of them will compete to generate different models that have the same names, so **you will randomly end up either with swords using spear models, or spears using sword models.**

Your intent here might be to have one module use textures ending in "`_sword`", while the other would use textures ending in "`_spear`". However, the modules cannot make those assumptions; the list of textures in the folder is what defines their list of variants, so they will collect **everything** inside, even the models intended for other modules.

**When you set a folder as your model prefix, treat it as if you are reserving the entirety of the folder for this module.** The simplest and cleanest way to fix the modules above is to make them use different prefixes:
```jsonc
"modelPrefix": "weapons/spear/"
"modelPrefix": "weapons/swords/"
```

### Namespacing your assets
Modules can collect assets across multiple packs, so if your Model Prefix sounds too generic, it can still potentially cause the same kind of conflicts with someone else's pack. Using some sort of namespace (something named after you or your project) is the surest way to avoid this.

The simplest way to do this is to add your namespace at the beginning of the models pathes:
```jsonc
"modelPrefix": "my-namsespace/weapons/"
```
Although this is *technically speaking* not a namespace, it achieves the same effect and will work everywhere.


Using a *real* namespace can be more complicated and varies with the module type:

<details>
<summary>Namespaceing with <code>component_threshold</code>, <code>item_count</code>, <code>durability</code>, <code>enchantment_vector</code>... </summary>

> Those particular modules are based on non-namespaced data, so hey have a simple `namespace` parameter that both restrict what models they will accept, and sets the namespace of their variant-ID:
> ```jsonc
> "type": "item_count",
> "parameters": {
> 	"namespace": "my-namespace"
> }
> ```
</details>


<details>
<summary>Namespaceing with <code>component_data</code></summary>

> This module is originally intended to suport Variant IDs from multiple namespaces at once.
>
> To restrict it to only one, use `modelNamespace` to limit what `modelPrefix` is allowed to collect, and use `get_identifier` to change how the variant ID is computed. You can use `sanitize_path` instead of plain `sanitize` to ensure the input isn't already namespaced.
> 
> ```jsonc
> {
> 	"modelPrefix": "...",
> 	"modelNamespace": "my-namespace",
> 
> 	"type": "component_data",
> 	"parameters": {
> 		"componentType": "custom_name",
> 		"transform": [
> 			{
> 				"function": "sanitize_path"
> 			},
> 			{
> 				"function": "get_identifier",
> 				"defaultNamespace": "my-namespace"
> 			}
> 		]
> 	}
> }
> ```
</details>


<details>
<summary>Namespaceing with <code>component_format</code></summary>

> This module is originally intended to suport Variant IDs from multiple namespaces at once.
>
> To restrict it to only one, use `modelNamespace` to limit what `modelPrefix` is allowed to collect, and add your namespace directly to the `format`.
> ```jsonc
> {
> 	"modelPrefix": "...",
> 	"modelNamespace": "my-namespace",
> 
> 	"type": "component_format",
> 	"parameters": {
> 		"format": "my-namespace:${etc}",
> 		"variables": {
> 			"etc": { ... }
> 		}
> 	}
> }
> ```
</details>

## Manipulating names with `component_data`
Although the module type `custom_name` is easy to set up, it is also very limited in what it can do. I recommend switching to `component_data` in almost all cases.

The `component_data` equivalent of a `custom_name` module is this:
```jsonc
{
	"item": "diamond_sword",
	"modelPrefix": "custom_swords/",

	"type": "component_data",
	"parameters": {
		"componentType": "custom_name",
		"transform": [
			{ "function": "sanitize" }
		]
	}
}
```
First, you might want to swap out "`custom_name`" for another flavour of name:
```jsonc
// pick one
"componentType": "custom_name"
"componentType": "item_name"
"property": "display_name"
```
- `custom_name` is the name given in an anvil.
- `item_name` is the item's intrinsic name, regardless custom name.
- `display_name` is whichever of the two is currently being shown to the player.

The `{"function":"sanitize"}` is the part that turns a name into an identifier (`"Épée de l'End"` => `"epee_de_lend"`).
You can use any other [transform](./Transforms) to modify the names in other ways. You can combine multiple of them to create more comple result, and the output of the first will be fed into the next. This works similarly to loot functions, but for plain data.

### Pattern Matching
The second most important transform you may need is once again `regex`. Regex in this context can be used not only for pattern matching, but also to only keep specific parts of the string.  
For example, suppose the player can add their name at the beginning of an item's name; e.g: `"Steve's Quartz Sword"`. You of course don't want to include the player's name in the Variant ID; here is one way to exclude it:
```jsonc
"transform": [
	{
		"regex": "(.+?'s )?(?<itemname>.+)",
		"substitution": "${itemname}"
	},
	{
		"function": "sanitize"
	}
]
```
Breakdown:
- `(.+?'s )?` matches an optional player name, followed by apostrophe-s and a space
	- The encompassing `( )?` makes this whole section optional
	- \``'s `\` matches a literal apostrohe-s-space
	- `.+?` matches any non-empty string. The `?` makes it **lazy**, so it will match as few characters as possible. I.e, it will stop at the first occurence of apostrophe-s, rather than the last.
- `(?<itemname>.+)` matches the actual item name.
	- `.+` matches any non-empty string.
	- The encompassing `(?<itemname>   )` captures the value of `.+`, and saves it into a variable called `itemname`.
	- `${itemname}` means the function will return the previously captured variable, and nothing else.

[See this regex in action](https://regex101.com/substitution?regex=%28%28%3F%3CplayerName%3E.%2B%3F%29%27s+%29%3F%28%3F%3CitemName%3E.%2B%29&testString=Steve%27s+Diamond+Sword%0AAlex%27s+Hell%27s+Fury%0AMasterless+Sword&flags=gm&flavor=java&delimiter=%22&substitution=Player%3A%C2%A0%60%24%7BplayerName%7D%60%C2%A0%7C%C2%A0Item%3A%C2%A0%60%24%7BitemName%7D%60)

### Names in non-latin alphabets
Identifiers only support a specific set of characters. If you play in a language that uses a non-latin alphabet, `sanitize` will turn nearly every name into an empty string. 
To preserve meaningful characters, use [`charset_remap`](./Transforms#transform-charset_remap) to manually remap them to latin characters:
```jsonc
"transform": [
	{
		"function": "charset_remap",
		// Maps characters one-to-one
		"source": "アイウエオ",
		"destination": "aiueo",
		// Maps one character to multiple others
		"map": {
			"ラ": "ra",
			"リ": "ri",
			"ル": "ru",
			"レ": "re",
			"ロ": "ro"
		}
	},
	{
		"function": "sanitize"
	}
]
```

## Using custom data as variants
### Locating the data
Suppose, for example you want to create a module for the effect of suspicious stew:
First, you need to figure out where the variant is stored in the item. Go in-game, put a bowl of supicious stew in your main hand, and use this command:  
```
/data get entity @s SelectedItem.components
```

 This should print something like this into the chat:

![`{"minecraft:suspicious_stew_effects": [{duration:7, id:"minecraft:saturation"}]}`](./nbt_path_command.jpg)

Here we learn that: (1) The effect id is in a component called `suspicious_stew_effect`. (2) The effect is stored under a key "id", which itself is stored at the first position of an array.

----

If this command is not available to you, you can instead use the Variant-CIT's `walkthrough` command for that. Add this bit to the root of your module, and it will print the given component's NBT into the chat whenever you use that command:
```jsonc
"precondition": {
	"suspicious_stew_effects": [ "get_nbt", "log" ]
}
```

### Configuring the module to use that data
Besides setting the `componentType`, most other components need a separate ['`nbtPath`'](./Item-Properties#field-nbtpath) option :
```json
"type": "component_data",
"parameters": {
	"componentType": "suspicious_stew_effects",
	"nbtPath": "[0].id"
}
```

Use `[n]` to access arrays. Indices start at zero, so `[0]` means "take the first element".
and `.abc` to access compounds. Some keys with special characters may require quotings (`.'abc'`). Keys must always be preceded by a dot, even if they are at the very beginning of the path.

Certain components change format between versions of Minecraft. For some of these, Variants-CIT provides [properties](./Item-Properties) which are more reliable.  
E.g: using `"componentType":"dyed_color"` sometimes requires `"nbtPath":".rgb"` and sometimes not. Instead, using `"property":"dyed_color"` instead will directly give you the color regardless of the version of Minecraft.

### Combining multiple pieces of data
The module `component_data` can only work with a single piece of data, `component_format` is its equivalent for working with multiple pieces of data:

```json
"type": "component_format",
"parameters": {
	"format": "${shape}_${duration}", // E.g: star_3
	"variables": {
		"shape":  {
			"componentType": "fireworks",
			"nbtPath": ".explosions[0].shape"
		},
		"duration": {
			"componentType": "fireworks",
			"nbtPath": ".flight_duration"
		}
	}
}
```
The `format` defines how the variant ID is constructed. Each `${variable_name}` will be replaced with the values defined in the `"variables":{}` object. Each variable inside use the same format as the `component_data` module.


## Read next:
- [**Tutorial: Understanding Asset-Generation**](./Understanding-AssetGen)
- [**Troubleshooting**](./Troubbleshooting)
