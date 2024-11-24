# Module Configuration

A module Configurations is a JSON file provided by a resource pack, located somewhere in `variants-cit/item`. It can control a large collection of CITs, and defines how to match them to an item.

If you're looking to create a resource pack for a vanilla item, you should have a look at the [recommended configurations](Recommended-Configurations).

## Schema
The strict minimum required to form a working module is its type, its target items, and the location of its item models:

```json
{
	"type": "painting_variant",
	"items": ["painting"],
	"modelPrefix": "item/painting/"
}
```

Here's a more complete example, as used by the mod Invariable-Paintings:

`/assets/invarpaint/variants-cit/item/painting.json`
```json
{
	"type": "painting_variant",
	"items": ["minecraft:painting"],
	"priority": 0,
	"modelPrefix": "item/painting/",
	"modelParent": "minecraft:item/generated",
	"fallback": "invarpaint:item/filled_painting",
	"special": {
		"invalid": "invarpaint:item/missing_painting",
		"obfuscated": "invarpaint:item/random_painting"
	},
	"parameters": {
	}
}
```

### `type`
**Mandatory**, Namespaced Identifier

The [type](Module-Types) of the CIT module used for this item. This affects how an item's variant is identified, and what special models are available to it.

### `items`
**Optional**, List of Namespaced Identifers, defaults to the module's id.

A list of item types the module will be applied to. If unspecified, the target item is automatically derived from the path of your module: `/assets/<item_namespace>/variant-cits/item/<item_name>.json`

Implicit targeting should be reserved for modules that are intrinsic to the item type. (See [recommended modules](Recommended-Configurations)).

The mod will silently ignore identifiers that do not match an existing item, so a texture pack will remain compatible with versions of minecraft that lacks some of the defined items.

### `priority`
**Optional**, Integer, defaults to 0.

When multiple modules are applied to the same item stack, modules with higher priorities are evaluated first. If, and only if, a module cannot provide a CIT for the item stack, then the next lower priority module will be evaluated. This can happen, if:
- The module cannot find a variant for the item.
- The module has found a variant, but that variant has no CIT, and no fallback was specified.
- The module wants to use a special model, but that model was not specified..

If your module can be considered intrinsic to the item (see [recommended modules](Recommended-Configurations)), and would almost never fail to apply a CIT, then I recommend a value around 0.
If your module is based on `custom_data` or `custom_name`, and often leave an item unchanged, then I recommend a value around 100.

### `modelPrefix`
**Mandatory**, String

A subdirectory of the `models/` folder, and/or a prefix. All models whose path start with this string (including subdirectories) are automatically loaded and matched to the item variant with the corresponding identifier:  
- Item variant: `<namespace>:<name>`
- Model path: `/assets/<namespace>/models/<modelPrefix><name>.json`

The name of the variant is appended *directly* to the model-prefix. The presence or absence of a trailing slash '`/`' in the model-prefix makes the difference between a directory, and a filename-prefix.

### `modelParent`
**Optional**, Namespaced Identifier

If set, item models will be automatically generated from textures whose name matches the `modelPrefix`, or the `fallback` and `special` models.

This value is used as the generated model's parent; typically, `item/generated` for regular items, and `item/handheld` for tools. When in doubt, try using the vanilla model as parent.

### `fallback`
**Optional**, Namespaced Identifier. Defaults to null.

The model to use for items for which a variant was identified, but no model was provided for this variant.
Modules with fallback are much less likely to hand over control to lower priority modules.

### `special`
**Optional**, maps Strings to Namespaced Identifiers.

A list of models that the CIT module may use in certain exceptional circumstances. What models are used and when depends on the [`type`](Module-Types) of the CIT module.

All special models are optional and default to null.
All models listed here are automatically loaded.


### `parameters`
**Type-dependent**, Object.

An object containing additional non-model parameters that affect the behaviour of the module.

The list of possible parameters and their effects depends on the [`type`](Module-Types) of the CIT module.
