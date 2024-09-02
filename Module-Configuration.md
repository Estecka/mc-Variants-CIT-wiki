# Module Configuration

A module Configurations is a JSON file provided by a resource pack, located somewhere int `variant-cits/item`. It can control a large collection of CITs, and defines how to match them to an item.

If you're looking to create a resource pack for a vanilla item, you should have a look at the [recommended configurations](Recommended-Configurations).

## Schema
Here's an example of a hypothetical module that would be used by the mod Invariable-Paintings:

`/assets/minecraft/variant-cits/item/painting.json`
```json
{
	"type": "painting_variant",
	"priority": 0,
	"items": ["painting"],
	"modelPrefix": "item/painting/",
	"fallback": "invarpaint:item/filled_painting",
	"special": {
		"invalid": "invarpaint:item/missing_painting",
		"obfuscated": "invarpaint:item/random_painting"
	},
	"parameters": {
	}
}
```
> [!WARNING]
>
> Invariable-paintings having grand-fathered this mod, its behaviour is the most feature-complete example of how CIT modules can work. 
> However, there exists no `painting_variant` module yet, as it could conflict with the current version of Invariable-Paintings.

### `type`
**Mandatory**, Namespaced Identifier

The [type](Module-Types) of the CIT module used for this item. This affects how an item's variant is identified, and what special models are available to it.

### `modelPrefix`
**Mandatory**, String

A subdirectory of the `models/` folder, and/or a prefix. All models whose path start with this string (including subdirectories) are automatically loaded and matched to the item variant with the corresponding identifier:  
- Item variant: `<namespace>:<name>`
- Model path: `/assets/<namespace>/models/<modelPrefix><name>.json`

The name of the variant is appended *directly* to the model-prefix. The presence or absence of a trailing slash '`/`' in the model-prefix makes the difference between a directory, and a filename-prefix.

### `priority`
**Optional**, Integer, defaults to 0.

If multiple modules are applied to the same item stack, modules with higher priorities are evaluated first. If a module ends up not providing a CIT for the stack, only then control will be handed over to the next lower priority module.

If your module can be considered intrinsic to the item (see [recommended modules](Recommended-Configurations)), and would almost never fail to apply a CIT, then I recommend a value around 0.
If your module is based on `custom_data` or `custom_name`, and often leave an item unchanged, then I recommend a value around 100.

### `items`
**Optional**, List of Namespaced Identifers, defaults to the module's id.

A list of item types the module will be applied to. If unspecified, the target item is automatically derived from the path of your module: `/assets/<item_namespace>/variant-cits/item/<item_name>.json`

Implicit targeting should be reserved for modules that are intrinsic to the item type. (See [recommended modules](Recommended-Configurations)).

Regardless of how the targets are defined, the mod will ignore identifiers that do not match an existing item. A pack can will remain compatible with version of minecraft that lack these some of the defined items.

### `fallback`
**Optional**, Namespaced Identifier. Defaults to null.

The model to use for items for which a variant was identified, but no model was provided for this variant.
Modules with fallback are much less likely to hand over control to lower priority modules.

### `special`
**Optional**, maps Strings to Namespaced Identifiers.

A list of models that the CIT module may use in certain exceptional circumstances.
All models listed here are automatically loaded.
The list of models that are actually used depends on the [`type`](Module-Types) of the CIT module.

All special models are optional and default to null.


### `parameters`
**Type-dependent**, Object.

An object containing additional non-model parameters that affect the behaviour of the module.

The list of possible parameters and their effects depends on the [`type`](Module-Types) of the CIT module.
