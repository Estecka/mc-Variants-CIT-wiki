# Building a Variant Library
This page covers some of the *theory* behind the mod. If you would like a more *practical* introduction instead, see: [Getting Started](./Getting%20Started).

### Index
- [What is a Variant IDs](#what-is-a-variant-id)
- [What is the Model ID of a given file](#what-is-the-correct-model-id-for-a-given-file)
- Binding models to a variant ID
	- [Overview](#overview)
	- [Harcoded lists](#hardcoded-list) (`modelList`)
	- [Dynamic lists](#dynamic-model-list) (`modelPrefix`)
- [Fallback and Special models](#fallback-and-special-models)

## What is a Variant ID ?
Variant IDs are an abstraction layer between items and their models.

Variant IDs were meant as a generic way to **describe items regardless of models**. Things like the type of a potion, an enchantment, or the name of a painting, are the quintessence of a Variant.

When a module looks at an item, it boils it down to a variant ID, and *then* uses the model associated to this variant ID. Every model you add to your module is bound to a variant ID; in this light, you may instead choose to see the variant IDs as an **Alias** or a **Shorthand** for a given **model ID**.

## What is the correct Model ID for a given file ?
"Model ID" is the brand of identifier used by modules refer to models and textures. It is sometimes different from the ID that vanilla Minecraft uses refers to those same files.

The golden rule that is that Model IDs mirror the vanilla component that is overriden with this ID:

Module Hook    | Vanilla equivalent of the Model ID
-----------    | ----------------------------------
`item_model`   | `item_model` component
`equippable`   | `asset_id` field of the `equippable` component
`trim_pattern` | `asset_id` field in trim patterns provided by datapacks

**Model IDs are always formatted such that they can refer to the same low-level assets that vanilla uses in those places.** When refering to other types of assets, those IDs will be interpreted according to the table below.

Hook           | Asset type        | Full path for model ID `<namespace>:<path>`
----           | ----------        | :------------------------------------------
`item_model`   | Item texture      | assets/`<namespace>`/textures/item/`<path>`.png
`item_model`   | Baked model       | assets/`<namespace>`/models/item/`<path>`.json
`item_model`   | Item state        | assets/`<namespace>`/items/`<path>`.json
`equippable`   | Equipment texture | assets/`<namespace>`/textures/entity/equipment/`<layer>`/`<path>`.png
`equippable`   | Equipment model   | assets/`<namespace>`/equipment/`<path>`.json
`trim_pattern` | Trim texture      | assets/`<namespace>`/textures/trims/entity/`<layer>`/`<path>`.png
`trim_pattern` | VCIT Trim model   | assets/`<namespace>`/variants-cit/trim_pattern/`<path>`.json

**The same Model ID can represent multiple files of different types**, and thus must never specify its type.
As a rule of thumbs, modules look for models across *all* asset types that are relevant to its hook, and use the lowest-level asset available for a given model ID.

Similarly, for items with multiple textures, the same model ID refers to every prefixed or suffixed variations. Thus those affixes must once again never be specified. Notably:
- For trims and equipments textures, the model ID must not include the `<layer>` (usually `humanoid` or `humanoid_leggings`). See also: [Equippable Armors](./Equipped-Armor).
- For items such as bows and tridents, the model ID must not include `_pulling`, `_in_hand`, etc.

The specifics of what affixes a module will look for depends on its [asset generator](./Asset-Generation#built-in-asset-generator-presets).

## Binding models to a variant ID
### Overview
Adding models to a module serves two purposes:
- Binding each models to one or several Variant IDs. This enables the module to use the model at all, and tells it when to use it.
- Telling the modules which JSON models needs to be generated based on textures you provided.  

There are two approaches to add models to your module:
- **Using `modelList`**: Manually list every models and their bindings to variant IDs.  
  This is the most straigthforward way, but may turn out awkward to use with a lot of purpose-made module types, which were designed with `modelPrefix` in mind. `modelList` is best used for modules with relatively few variants.
- **Using `modelPrefix`**: Automatically gathering everything in a folder.  
  This was the only available option for a long time, and the system most of the mod was designed around. You can add new variants to your module by simply adding files into the designated folder.
  This system is intended for large and growing collections of variants.


Both ways of binding models are compatible with each other. Any binding done in `modelList` will take priority over those found by `modelPrefix`.

### Hardcoded list
The module will collect exactly the models listed there, so you won't have to think about assetGen conflicts with that method.

```json
{ 
	"modelList": {
		"variant_id_1": "model_id_1",
		"variant_id_2": "model_id_2",
		"etc": "..."
	}
}
```

### Hardcoded List with modelPrefix
Using a `whitelist` transform with `modelPathes` acts similarly to a hardcoded list, but keep `modelPrefix`'s way of binding models to variant IDs:

```json
{
	"modelPrefix": "named_sword/",
	"modelPathes": {
		"whitelist": [
			"end_sword",
			"end_cleaver",
			"..."
		]
	}
}
```
The above is equivalent to this:
```json
{
	"modelList": {
		"end_sword":   "named_sword/end_sword",
		"end_cleaver": "named_sword/end_cleaver",
		"...":         "named_sword/...",
	}
}
```


### Dynamic model list
```json
{
	"modelPrefix": "enchanted_book/"
}
```

Using `modelPrefix` on its own, the module will attempt to collect every asset whose model ID starts with the given prefix. The variant ID associated with the model is the remainder after removing the prefix:  
`"<namespace>:<modelId>"` == `"<namespace>:<modelPrefix><variantId>"`

> [!CAUTION]
>
> The module will attempt to generate JSON models for every texture it collects in this way. Those models have a file path, like any other file in a resource pack. **Common misuses of `modelPrefix` can cause one module to go overboard, generating swathes of models it can't realistically use, and overriding the models generated by other modules.** (See: Issues labelled with [assetGen conflict](https://github.com/Estecka/mc-Variants-CIT/issues?q=is%3Aissue%20label%3A%22assetGen%20conflict%22))
>
> **As a rule of thumbs, the modelPrefix should be unique to each module.**
> Only make them overlap if you understand how this will affect asset-gneration.
> You can use the [dump](./Troubbleshooting#command-dump) command to check what models a module has collected and generated.

By design, this makes it possible for other resource-packs to contribute models to your module, by simply naming their assets after your model prefix. 

On its own, `modelPrefix` will attempt to collect **every single** model that starts with that prefix, but you can further restrict this using `modelNamespace` and `modelPathes`
```json
{
	"modelPrefix": "named_sword/",
	"modelNamespace": "my-namespace",
	"modelPathes": { "regex": ".*(_sword|_cleaver)" }
}
```

Both `modelNamespace` and `modelPathes` can be [transforms](./Transforms) used as predicates. The most relevant ones here are [regex](./Transforms#transform-regex), [whitelist](./Transforms#transform-whitelist--blacklist) and [blacklist](./Transforms#transform-whitelist--blacklist).  
`modelNamespace` can instead be a plain string, representing a single namespace that will be accepted.  
The modelPrefix is not included in the strings evaluated by `modelPathes`.

These two fields don't do anything on their own. Picking a `modelPrefix` is still required for the models to be bound to a variant ID. The only purpose of these field is to **prevent** the module from collecting certain assets that may be intended for other modules.

### Best practice: Namespacing your assets

If you have something like a name-based or lore-based modules, its variants probably aren't inherently namespaced. In this case, a good practice to avoid conflicts with other packs, would be to move your models to a namespace of your own. 

If you're using `component_data`, use [`get_identifier`](./Transforms#data-type-transforms) to apply that namespace to the variant IDs computed by your module:

```jsonc
{
	"...": "...",

	"modelPrefix": "named_sword/",
	"modelNamespace": "my-namespace",

	"type": "component_data",
	"parameters":
	{
		"property": "display_name",
		"transform": [
			{
				"function": "sanitize_path"
			},
			{
				"function": "get_identifier",
				"defaultNamespace": "my-namespace"
			}
		]
	}
}
```
Certain module types (`durability`, `enchantment_vector`) already take a `namespace` parameter. 
Those modules already filter out models that don't match their attributed namespace, so `modelNamespace` is rendundant on these. `modelNamespace` it is intentended for use with module types that can collect models from multiple namespaces at once.

## Fallback and Special models
Those models have special roles that don't neatly fit into the variant ID system. They are only useful in some very niche scenarios.

The old-school syntax for these is:
```json
{
	"fallback": "fallback_model_id",
	"special": {
		"name1": "special_model_id_1",
		"name2": "special_model_id_2"
	}
}
```
Internally, those models are bound to variant IDs in the `variants-cit:` namespace:
```json
{
	"modelList": {
		"variants-cit:fallback": "fallback_model_id",
		"variants-cit:special/name1": "special_model_id_1",
		"variants-cit:special/name2": "special_model_id_2"
	}
}
```

### Fallback
The fallback model will be used if a module is able to compute a variant ID for an item, but that variant ID has no associated model of its own.

### Special
Special models are functionalities specific module types. Only a couple of modules actually use them: `enchantment` has a `multi` special model, and `painting_variant` has an `invalid` special model.
