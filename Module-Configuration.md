# Module Configuration

A module Configurations is a JSON file provided by a resource pack, which describes the CIT logic for exactly one item type. The type of the target item is automatically derived from the path of the file:  
`/assets/<item_namespace>/variants-cit/<item_name>.json`

Only one CIT module can exist per item. If you're looking to create a resource pack for a vanilla item, you should have a look at the [recommended configurations](Recommended-Configurations) in order to maximise compatibility with other packs.

## Parameters
Here's an example of a hypothetical module that would be used by the mod Invariable-Paintings:

`/assets/minecraft/variants-cit/painting.json`
```json
{
	"type": "painting_variant",
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

### `fallback`
**Optional**, Namespaced Identifier. Defaults to the vanilla model.

The model to use for items for which a variant was identified, but no model was provided for this variant.

### `special`
**Optional**, maps Strings to Namespaced Identifiers.

A list of models that the CIT module may use in certain exceptional circumstances.
All models listed here are automatically loaded.
The list of models that are actually used depends on the [`type`](Module-Types) of the CIT module.

All special models are optional and default to the vanilla model.


### `parameters`
**Type-dependent**, Object.

An object containing additional non-model parameters that affect the behaviour of the module.

The list of possible parameters and their effects depends on the [`type`](Module-Types) of the CIT module.
