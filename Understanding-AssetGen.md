# Understanding Asset Generation

The other page [Asset Generation](./Asset-Generation) is technical documentation. It lists all available options and functionalities in detail, but devoid of context.

This page here will try to walk you through the concepts in broader terms, and focus more on the why and the how.

### Prerequisite
This tutorial assumes you already have some familiarity with the mod, or have at least followed the introductory tutorial:
- [Getting Started](./Getting-Started)

### Index
- [What is Asset Generation](#what-is-asset-generation)
	- [What file for what model ID](#what-file-for-what-model-id)
	- [Asset-gen conflicts](#asset-gen-conflicts)
- [AssetGen with custom models](#assetgen-with-custom-models)
	- [Overriding models for specific variants](#overriding-models-for-specific-variants)
	- [Using `modelParent`](#using-modelparent)
	- [Using custom JSON templates](#using-custom-json-templates) (WIP)

## What is Asset Generation ?
Variants-CIT makes no change to the way Minecraft renders its model. All it does is override the value of the item components that controls its appearance. Minecraft receives the modified component, and the vanilla rendering pipeline picks-up it from there.

In the "Getting Started" tutorial, you were able to change the look of a sword by only providing textures; but Minecraft does not work with only textures, it needs JSON models.  
Asset Generation is the process that generates those JSON models, based on the textures you provided. The `assetGen` option in your module is what tells the mod how to generate those JSON models.

### What file for what Model ID ?
In modules, the "Model ID" is the value that is used to override the item's component;

Module Hook    | Equivalent of the Model ID
-----------    | ----------------------------------
`item_model`   | `item_model` component
`equippable`   | `asset_id` field of the `equippable` component
`trim_pattern` | `asset_id` field of the trim pattern referenced from the `trim` component

In turn, these IDs represent the low-level assets associated with these components:

Module Hook    | Asset type        | Full path for model ID `<namespace>:<path>`
----           | ----------        | :------------------------------------------
`item_model`   | Item state        | assets/`<namespace>`/items/`<path>`.json
`equippable`   | Equipment model   | assets/`<namespace>`/equipment/`<path>`.json
`trim_pattern` | VCIT Trim model*  | assets/`<namespace>`/variants-cit/trim_pattern/`<path>`.json

_*This one is not vanilla. See [Trim Pattern Assets](./Equipped-Armor#trim-pattern-assets)_

A module can work without an `assetGen` option, but then will specifically require the files above.

With asset-generation enabled, the model IDs in your modules are extended to higher-level asset types. The module will collect every asset that matches a given model ID, and generate the underlying asset types:

Module Hook    | Asset type        | Full path for model ID `<namespace>:<path>`
----           | ----------        | :------------------------------------------
`item_model`   | Item texture      | assets/`<namespace>`/textures/item/`<path>`.png
`item_model`   | Baked model       | assets/`<namespace>`/models/item/`<path>`.json
`item_model`   | Item state        | assets/`<namespace>`/items/`<path>`.json
`equippable`   | Equipment texture | assets/`<namespace>`/textures/entity/equipment/`<layer>`/`<path>`.png
`equippable`   | Equipment model   | assets/`<namespace>`/equipment/`<path>`.json
`trim_pattern` | Trim texture      | assets/`<namespace>`/textures/trims/entity/`<layer>`/`<path>`.png
`trim_pattern` | VCIT Trim model   | assets/`<namespace>`/variants-cit/trim_pattern/`<path>`.json


### Asset-Gen Conflicts
Two important things to keep in mind about generated assets: 
- **All models are generated during resource-reload, *not* at render-time.**  
  A module will preemptively generate all models that are registered to it. It is possible for a module to generate models it can never use.
- **Generated models are received by minecraft in the form of a resource pack.**
  Generated models have a file path and an identifier, just like any other file in a regular resource pack. Two models in the game cannot have the same identifers.

Conflicts occur if multiple modules attempt to create different files with the same name. In this case only one of the two models is used.

The module type `predicates` is unlikely to cause conflicts within your own pack, but could still conflict with other packs. The easiest way to avoid this issue is to put your models in a custom namespace:
```json
"parameters": {
	"predicates": [
		{
			"modelId": "my-namespace:<path>",
			"precondition": { ... }
		}
	]
}
```

Module types that use `modelPrefix` are particularly suceptible to this kind of conflicts, because by design, they will claim **all** assets that match the prefix across **all** namespaces. If you plan on using them, please read-up on the [Best Practices](./Seeking-Patterns#best-practices) for those modules.


## AssetGen with custom models
### Overriding models for a handful of variants
**Asset-Generation will never overwrite existing files.**
It only act as a default for missing models.

If you have a couple of models that should look different from the rest, your resource pack can directly provide any file from the above table, which will override the generated result for this file.

### Using `modelParent`
This option is specific to the `item_model` hook. It is roughly equivalent to using the assetGen presets `item_model/generated` or `item_model/handheld`, but lets you pick the parent to use for baked models.

For example, these two codes produce the same results:
```jsonc
{
	"assetGen": "item_model/generated"
}
```
```jsonc
{
	"modelParent": "item/generated"
}
```
The difference between the two is what values they accept and what they do with it; `modelParent` can refer to any model in the game, whereas `assetGen` is limited to the list of [existing presets](./Asset-Generation#built-in-asset-generator-presets) or to generators you write yourself.

### Using custom JSON templates
> [!NOTE]
> 
> This part of the tutorial has yet to be written.
> For now refer to the technical documentation:
> [Custom Asset Generators](./Asset-Generation#custom-asset-generators)
<!-- WIP
For the `item_model` hook, this can let you generate variants with custom item states.
For the `equippable` hook, this can let you generate equipments that are dyeables, or are made of several overlapping layers.


First, let's take the preset `"assetGen":"item_model/generated"`, and unwrap its actual code:
```json
"assetGen": [
	{
		"pass": "models_from_textures",
		"template": "models/parent",
		"variables": {
			"modelParent": "item/generated"
		}
	},
	{
		"pass": "items_from_models",
		"template": "items/stateless"
	}
]
```
-->
