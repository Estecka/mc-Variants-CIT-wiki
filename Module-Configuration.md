# Module Configuration

A module Configurations is a JSON file provided by a resource pack, located somewhere in `variants-cit/item`. It controls a large collection of CITs, and defines how to match them to an item.

## Schema
The strict minimum required to form a working module is its type, its target items, and the location of its models:

```json
{
	"type": "painting_variant",
	"items": ["painting"],
	"modelPrefix": "item/painting/"
}
```

Here's a more complete example, as used by the mod Invariable-Paintings:

```json
{
	"type": "painting_variant",
	"items": ["minecraft:painting"],
	"priority": 0,
	"modelPrefix": "item/painting/",
	"modelParent": "minecraft:item/generated",
	"itemsFromModels": true,
	"fallback": "invarpaint:item/filled_painting",
	"special": {
		"invalid": "invarpaint:item/missing_painting"
	},
	"parameters": {
	}
}
```

### `type`
**Mandatory**, Namespaced Identifier

The [type](Module-Types) of the CIT module used for this item. This affects how an item's variant is identified, and what special models are available to it.

### `items`
**Required**, List of Namespaced Identifers, defaults to the module's id.

A list of item types the module will be applied to. The mod will silently ignore identifiers that do not match an existing item, so a texture pack will remain compatible with versions of minecraft that lacks some of the defined items.

### `priority`
**Optional**, Integer, defaults to 0.

When multiple modules are applied to the same item stack, modules with higher priorities are evaluated first. If, and only if, a module cannot provide a CIT for the item stack, then the next lower priority module will be evaluated. This can happen, if:
- The module cannot find a variant for the item.
- The module has found a variant, but that variant has no CIT, and no fallback was specified.
- The module wants to use a special model, but that model was not specified.

If your modules would almost never fail to apply a CIT, then I recommend a value around 0.
If your module often leave an item unchanged, then I recommend a value around 100.

### `modelPrefix`
**Mandatory**, String

A subdirectory of the `items/` folder, and/or a prefix. All assets whose path start with this string (including subdirectories) are matched to the item variant with the corresponding identifier:  

The name of the variant is appended *directly* to the model-prefix. The presence or absence of a trailing slash '`/`' makes the difference between a prefix that consists only of directories, and a prefix that contains a filename.

Depending on which asset generation options are used, the module may also look for models and/or textures with matching names:

Item variant     | `<namespace>:<path>`
---------------- | :-------------------
Matching item    | `/assets/<namespace>/items/<modelPrefix><path>.json`
Matching model   | `/assets/<namespace>/models/item/<modelPrefix><path>.json`
Matching texture | `/assets/<namespace>/textures/item/<modelPrefix><path>.json`


> [!INFO]
>
> Pre-1.21.4, models were resolved from the root of `models/` and `texture/`, instead of their `item/` subdirectory.
> For backward compatibility, values with a leading "item/" will have that prefix stripped off.


### `modelParent`
**Optional**, Namespaced Identifier.

If set, missing `models/` will be automatically generated from existing `textures/`, whose name matches the `modelPrefix`, or the `fallback` and `special` models.

The give value is used as the generated model's parent; typically, `item/generated` for regular items, and `item/handheld` for tools. When in doubt, try using the vanilla model as parent.

Resulting models:
```json
{
	"parent": "<modelParent>",
	"textures":{
		"layer0":"<namespace>:item/<modelPrefix><path>"
	}
}
```

### `itemsFromModels`
**Optional**, boolean, defaults to true.

If set, missing `items/` will be automatically generated based on existing or previously generated `models/`, whose name matches the `modelPrefix` or the `fallback` and `special` models.

Resulting item state:
```json
{
	"model": {
		"type": "model",
		"model": "<namespace>:item/<modelPrefix><path>"
	}
}
```

### `fallback`
**Optional**, Namespaced Identifier.

The model to use for items for which a variant was identified, but no model was provided for this variant.
Modules with fallback are much less likely to hand over control to lower priority modules.

> [!INFO]
>
> For backward compatibility with pre-1.21.4 packs, the leading "item/" will be stripped off if present.


### `special`
**Optional**, maps Strings to Namespaced Identifiers.

A list of models that the CIT module may use in certain exceptional circumstances. What models are used and when depends on the [`type`](Module-Types) of the CIT module.

All special models are optional and default to null.
All models listed here are automatically loaded.

> [!INFO]
>
> For backward compatibility with pre-1.21.4 packs, the leading "item/" will be stripped off if present.


### `parameters`
**Type-dependent**, Object.

An object containing additional non-model parameters that affect the behaviour of the module.

The list of possible parameters and their effects depends on the [`type`](Module-Types) of the CIT module.
