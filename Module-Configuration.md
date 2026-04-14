# Modules

A module is a JSON file provided by a resource pack, located in `variants-cit/modules/`. It controls a large collection of models, and defines how to match them to an item.

> [!NOTE]
> 
> If you look at odler packs, you may find that they store their modules in `item/` instead of `modules/`. This older directory is still functional, but deprecated.

Ultimately, the only thing a module does is override one of the component of an item, (either `item_model` or `equippable`), meaning everything that can be done with those components can also be done with Variants-CIT. Having a good understanding of vanilla model mechanics ([Item states](https://minecraft.wiki/w/Items_model_definition) and [Equipments](https://minecraft.wiki/w/Equipment)) will help a lot.

## Schema
The strict minimum required to form a working module is its type, its target items, and the location of its models:

```json
{
	"type": "painting_variant",
	"items": "painting",
	"modelPrefix": "painting/",
	"assetGen": "item_model/generated"
}
```

`assetGen` is technically optional, but will be useful in almost all cases.

Here's a different example with all possible fields:
```json
{
	"priority": 0,
	"hook": "item_model",
	"items": ["diamond_sword"],
	"precondition": {
		"custom_data": "END_SWORD"
	},

	"type": "enchantment",
	"parameters": {
		"levelSeparator": "_"
	},

	"assetGen": "item_model/handheld",
	"modelPrefix": "end_sword/single_enchant/",
	"fallback": "end_sword/unknown_enchant",
	"special": {
		"multi": "end_sword/multi_enchants"
	}
}
```

## Behaviour-related fields
Fields about the way modules look at item stacks.

### Field: `type`
**Mandatory** Identifier

Defines how an item's variant ID will be computed: Are we looking at custom names? Enchantments? Are we looking for an exact match? A threshold value?

See [here](Module-Types) for a complete list of possible types, and what parameters they may accept or require.

### Field: `parameters`
**Type-dependent**

An object containing additional parameters that affect the behaviour of the module.
The list of possible parameters and their effects depends on the **type** of the module. This object may or may not be required depending on the module's type.

### Field: `items`
**Mandatory**. A single, or an array of Identifiers.

A list of item types the module will be applied to. The mod will silently ignore identifiers that do not match an existing item, so a module will remain compatible with versions of minecraft that lack some of the defined items.

### Field: `precondition`
**Optional**. A single, or an array of preconditions.

A set of conditions that the item must match for the module to apply.

See [Preconditions](./Precondition%20Cheat-Sheet) for the syntax to use in this field. See also [Transforms](./Item-Properties#transforms) for complete list of ways you can evaluate data.


> [!IMPORTANT]
> 
> **Module preconditions should not be used to differentiate between individual models or textures.**
> In the example above the precondition dictates **whether** the sword can have a custom texture **at all**. The enchantment dictates **which** texture to use.
> 
> If you wanted to set a texture based only on the value of this custom data, a ["`component_data`" module](./Module-Types#module-component_data) might be a better fit.
> 
> If you really must use preconditions to separate every single model, use a ["`predicates`" module](./Module-Types#module-predicates) instead.



### Field: `hook`
**Optional**, a single, or an array of strings. Defaults to `"item_model"`.

Formerly known as "`context`".

Defines which aspect of the item will be changed by the module.
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

As an alternative to priority, if you have multiple modules that apply to the same item, and have similar values for `modelPrefix` and `assetGen`, you can instead bundle them into a ["`group`" module](./Module-Types#module-group). This will achieve the same effect, but yield better performances than assigning them different priorities.

## Variant Library
Fields that define what set of assets will be collected by the module, in order to create its library of variants.

> [!IMPORTANT]
>
> Pre-1.21.4, before the introduction of the `items/` folder, model prefixes, fallbacks and special models were resolved from the root of `models/` and `texture/`, instead of their `item/` subdirectory.
>
> For versions of minecraft 1.21.4 and above, the leading `item/` is implied. For backward compatibility, pathes starting with "`item/`" will have that bit ignored. However if you want your pack to be compatible with versions earlier than MC 1.21.4, you must keep using "`item/`" at the start of your pathes.

### Field: `modelPrefix`
**Mandatory**, String

The location of the assets that this module will use as variants. The exact asset type varies with the module's `hook` and asset-generation options. They can be textures, json models, or a mix of everything.

The presence or absence of a slash '`/`' at the end of a prefix is important! It makes the difference between a prefix that consists only of directories, and a prefix that contains the beginning of a filename:
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

### Field: `fallback`
**Optional**, Identifier.

If the module managed to compute a variant ID for an item, but this variant has no associated model, the fallback model will be used instead.
Modules with fallback are much less likely to hand over control to lower priority modules.

### Field: `special`
**Optional**, maps Strings to Identifiers.

A list of models that the some module types may use in exceptional circumstances.
What models are used and when depends on the [`type`](Module-Types) of the CIT module.

Very few module types actually make use of this option; you can safely ignore it.
All special models are always optional.


## Asset-generation

### Field: `assetGen`
**Optional**, A single or an array of Asset Generators.

An asset generators can be either a [Custom Generator](./Asset-Generation#custom-asset-generators), or the identifier to a [Generator Preset](./Asset-Generation#built-in-asset-generator-presets).
**Those identifiers default to the `variants-cit` namespace, instead of `minecraft`.**

In most cases, the values you'll want to use are:
- `item_model/generated` for basic items.
- `item_model/handheld` for basic tools.
- `equipment/humanoid` for basic 4-pieces armors.

For items with animations, such as bows, see the complete list of presets linked above.

### Field: `modelParent`
**Optional**, Identifier.

Old-school alternative to the `assetGen` option; this does nothing if `assetGen` is already defined. It is equivalent to this, where `<modelParent>` is the value of this option:
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
