# Modules

A module is a JSON file provided by a resource pack, located somewhere in `variants-cit/item/`. It controls a large collection of models, and defines how to match them to an item.

Ultimately, all a module does is override one of the component of an item, (either `item_model` or `equippable`), meaning everything that can be done with those components can also be done with Variants-CIT. Having a good understanding of vanilla model mechanics ([Item states](https://minecraft.wiki/w/Items_model_definition) and [Equipments](https://minecraft.wiki/w/Equipment)) will help a lot.

## Schema
The strict minimum required to form a working module is its type, its target items, and the location of its models:

```json
{
	"type": "painting_variant",
	"items": "painting",
	"modelPrefix": "item/painting/",
	"assetGen": "item_model/generated"
}
```

(`assetGen` is technically optional, but will be useful in almost all cases.)

Here's a more complete example, as used by the mod Invariable-Paintings:

```json
{
	"type": "painting_variant",
	"items": ["minecraft:painting"],
	"context": "item_model",
	"priority": 0,
	"modelPrefix": "item/painting/",
	"assetGen": "item_model/generated",
	"itemsFromModels": true,
	"fallback": "invarpaint:item/filled_painting",
	"special": {
		"invalid": "invarpaint:item/missing_painting"
	},
	"parameters": {
	}
}
```

## Behaviour-related fields
Fields about the way modules look at item stacks.

### Field: `type`
**Mandatory**, Namespaced Identifier

Defines how an item's variant ID will be computed. (E.g: `custom_name`, `durability`, etc.)
See [here](Module-Types) for a complete list of possible types, and what parameters they may accept or require.

### Field: `parameters`
**Type-dependent**

An object containing additional parameters that affect the behaviour of the module.
The list of possible parameters and their effects depends on the **type** of the CIT module. This object may or may not be required depending on the module's type.

### Field: `items`
**Mandatory**, A single, or an array of Namespaced Identifiers.

A list of item types the module will be applied to. The mod will silently ignore identifiers that do not match an existing item, so a texture pack will remain compatible with versions of minecraft that lack some of the defined items.

### Field: `context`
**Optional**, a single, or an array of strings. Defaults to `"item_model"`.

Defines which aspect of the item the module will change.
Possible values are:
- **`item_model`**: causes the module to override the `item_model` component of the item.
- **`equippable`**: causes the module to override the `equippable` component of the item, or more specifically, the '`assetId`' field inside that component.

It's technically possible for a module to pull double-duty, and change both aspects of an item at once. However, because of the discrepancy in how item assets and equipment assets are structured, it might not always be as convenient as using two separate modules.

### Field: `priority`
**Optional**, Integer, defaults to 0.

When multiple modules are applied to the same item stack, modules with higher priorities are evaluated first. If, and only if, a module does not change the stack's model, then the next lower priority module will be evaluated. This can happen, if:
- The module is unable to compute a variant ID for the item.
- The module has found a variant ID, but that variant has no associated model, and no fallback was specified.
- The module wants to use a special model, but that model was not specified.

If your modules would almost never fail to apply a CIT, then I recommend a value around 0.
If your module often leave an item unchanged, then I recommend a value around 100.

## Variant Library
Fields that define what set of assets will be collected by the module, in order to create its library of variants.

### Field: `modelPrefix`
**Mandatory**, String

The location of the assets that this module will use as variants. The exact asset type varies with the module's `context`, and its asset-generation options. They can be textures, json models, or a mix of everything.

The presence or absence of a slash '`/`' at the end of a prefix is important ! It makes the difference between a prefix that consists only of directories, and a prefix that contains the beginning of a filename:
- Prefix: `enchanted_book/` -> Variant asset: `<namespace>:enchanted_book/<path>`
- Prefix: `enchanted_book_` -> Variant asset: `<namespace>:enchanted_book_<path>`

> [!CAUTION]
>
> You should never use an empty string, or only `"item/"` as the model prefix. Doing so will cause the module to collect **every single** model in the game as potential variants. This may cause unexpected behaviours (especially with modules like `custom_name`) and reduce performances.

The fundamental asset types are [Item States](https://minecraft.wiki/w/Items_model_definition) (`items/`) and [Equipment models](https://minecraft.wiki/w/Equipment) (`equipments/`). With the proper `assetGen` option, modules will also collect orphaned textures and baked models, and automatically create the underlying asset types.

#### Asset types for `item_model` modules
Variant ID                         | `<namespace>:<path>`
---------------------------------- | :-------------------
Equivalent `item_model` component  | `<namespace>:<modelPrefix><path>`
Matching item state                | `/assets/<namespace>/items/<modelPrefix><path>.json`
Matching baked model               | `/assets/<namespace>/models/item/<modelPrefix><path>.json`
Matching item texture              | `/assets/<namespace>/textures/item/<modelPrefix><path>.png`

#### Asset types for `equippable` modules
Variant ID                         | `<namespace>:<path>`
---------------------------------- | :-------------------
Equivalent `equippable` component  | `<namespace>:<modelPrefix><path>`
Matching equipment asset           | `/assets/<namespace>/equipment/<modelPrefix><path>`
Matching equipment texture         | `/assets/<namespace>/entity/equipment/<layer>/<modelPrefix><path>`


> [!IMPORTANT]
>
> Pre-1.21.4, before the introduction of the `items/` folder, prefixes were resolved from the root of `models/` and `texture/`, instead of their `item/` subdirectory.
> For backward compatibility, prefixes starting with "`item/`" will have that bit stripped off in MC 1.21.4. If you want your pack to be compatible with both MC 1.21.4 and earlier versions, you must keep using "`item/`" at the start of your prefix.

### Field: `fallback`
**Optional**, Namespaced Identifier.

The model to use for items for which a variant was identified, but no model was provided for this variant.
Modules with fallback are much less likely to hand over control to lower priority modules.

> [!NOTE]
>
> For backward compatibility with pre-1.21.4 packs, the leading "item/" will be stripped off if present.


### Field: `special`
**Optional**, maps Strings to Namespaced Identifiers.

A list of models that the CIT module may use in certain exceptional circumstances. What models are used and when depends on the [`type`](Module-Types) of the CIT module.

All special models are optional and default to null.
All models listed here are automatically loaded.

> [!NOTE]
>
> For backward compatibility with pre-1.21.4 packs, the leading "item/" will be stripped off if present.

## Asset-generation

### Field: `assetGen`
**Optional**, A single or an array of Asset Generators.

Enabling this option will override the old-school `modelParent` and `itemsFromModel`.

An asset generators can be either a [Custom Generator](./Asset-Generation#custom-asset-generators), or the identifier to a [Generator Preset](./Asset-Generation#built-in-asset-generator-presets).
**Those identifiers default to the `variants-cit` namespace, instead of `minecraft`.**

In most cases, the values you will want to use are:
- `item_model/generated` for basic items.
- `item_model/handheld` for basic tools.
- `equipment/armor` for basic 4-pieces armors.

For items with animations such as bows, see the complete list of presets linked above.

### Field: `modelParent`
**Optional**, Namespaced Identifier.

Unless `itemsFromModels` is set to false, this will cause baked models and item states to be automatically generated from textures collected by the module.

This is the old-school alternative to `assetGen`, and will not do anything if `assetGen` is already defined. This option will remain supported for the foreseeable future, but is now backed by `assetGen` under the hood. It is equivalent to this, where `<modelParent>` is the value of this option:
```json
"assetGen": [
	{
		"pass": "models_from_textures",
		"template": "models/parent",
		"variables": {
			"modelParent": "<modelParent>"
		}
	},
	{
		"pass": "items_from_models",
		"template": "items/stateless"
	}
]
```

### Field: `itemsFromModels`
**Optional**, boolean, defaults to `true`.

See `modelParent` right above.

This setting defaults to `true` only for backward compatibility with pre-MC1.21.4 era packs. If you use neither `assetGen` nor `modelParent`, you probably want to set this to `false` as well.

In the future, this option may be removed and merged into `modelParent`.
