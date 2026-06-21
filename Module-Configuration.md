# Modules

A module is a JSON file provided by a resource pack, located in `variants-cit/modules/`. It controls a large collection of models, and defines how to match them to an item.

> [!NOTE]
> 
> If you look at older packs, you may find that they store their modules in `item/` instead of `modules/`. This older directory is still functional, but deprecated.

Ultimately, the only thing a module does is override one of the component of an item, (either `item_model` or `equippable`), meaning everything that can be done with those components can also be done with Variants-CIT. Having a good understanding of vanilla model mechanics ([Item states](https://minecraft.wiki/w/Items_model_definition) and [Equipments](https://minecraft.wiki/w/Equipment)) will help a lot.

## Schema
The strict minimum required to form a working module is its type, its target items, and the location of its models:

```json
{
	"items": "painting",
	"type": "painting_variant",
	"modelPrefix": "painting/",
	"assetGen": "item_model/generated"
}
```

`assetGen` is technically optional, but will be useful in almost all cases.

Here's a different example with a few more fields:
```json
{
	"items": ["diamond_helmet", "diamond_boots"],
	"precondition": { "custom_data": "END_ARMOR" },

	"type": "durability",
	"parameters": {
		"scale": 100
	},

	"modelList": {
		"100": "end_armor_full",
		"50":  "end_armor_half",
	},

	"hook": "equippable",
	"assetGen": "equipment/humanoid"
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

See [Preconditions](./Precondition%20Cheat-Sheet) for the syntax to use in this field. See also [Transforms](./Transforms) for complete list of ways you can evaluate data.


> [!IMPORTANT]
> 
> **Module preconditions should not be used to differentiate between individual models or textures.**
> In the example at the top, the precondition dictates **whether** the armor can have a custom texture **at all**. The durability dictates **which** texture to use.
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

As an alternative to priority, if you have multiple modules that apply to the same item, and use similar values for `modelPrefix` and `assetGen`, you can instead bundle them into a ["`group`" module](./Module-Types#module-group). This will achieve the same effect, but yield better performances than assigning them different priorities.

## Variant Library
Fields that define what set of assets will be collected by the module, in order to create its library of variants.

The exact asset type that a module looks for varies with its `hook` and asset-generation options. This page only describes the different fields in isolation, for a more comprehensive overview of what it means for a module to collect assets, see the dedicated page: [Building a Variant Library](./Variant-Library)

> [!IMPORTANT]
> 
> You may see older modules use `item/` at the start of their model prefix or other pathes. This is a relica from before MC 1.21.4 and is only supported for the sake of backward compatibility. This may stop being supported in the future.
> 
> The leading `item/` is now implied for item models and item textures. It should no longer be specified, in order to mirror the IDs of items-states, located in `items/` (plural).

### Field: `modelList`
**Optional**, _either a list of identifiers, or a map of identifiers to identifier._

A hardcoded list of models that the module can use, and their variant IDs. This can be used in replacement of, or in complement to the `modelPrefix` option.

If a map, each key is a variant ID, and the value is its associated model. 

If an array, the variant IDs will identical to the model IDs.

Any binding done here will override variant IDs detected by the `modelPrefix`.
Models listed here are exempt from matching the predicates `modelNamespace` and `modelPathes`. However, certain module types may still refuse variant IDs listed here.

### Field: `modelPrefix`
**Optional**, _string_

The location of a of set models that the modules can use. The module will automatically collect every asset whose Model ID starts with this prefix, and use the remainder of the path as the model's Variant ID.
The exact set of of models collected in this way can be further restricted using `modelNamespace` and `modelPathes`.

The prefix **cannot** be empty.

The presence or absence of a slash '`/`' at the end of a prefix is important! It makes the difference between a prefix that consists only of directories, and a prefix that contains the beginning of a filename:
- Prefix: `enchanted_book/` -> Models: `<namespace>:enchanted_book/<path>`
- Prefix: `enchanted_book_` -> Models: `<namespace>:enchanted_book_<path>`

### Field: `modelNamespace`
**Optional**, _a string, or a chain of transforms._

A predicate for which namespaces the `modelPrefix` is allowed to collect models from. 

If absent, all namespaces will be used.  
If a plain string, this represents a single namespace to pull from.  
If a chain of transforms, it will be used as a predicate. The most relevant transform to use here is [whitelist](./Transforms#transform-whitelist--blacklist)

### Field: `modelPathes`
**Optional**, _a chain of transform_.

A predicated for which variant IDs are allowed to be collected using `modelPrefix`.
The most relevant transforms to use here is [regex](./Transforms#transform-regex) for pattern matching.

The evaluated strings do not include the model prefix.


### Field: `fallback`
**Optional**, Identifier.

If the module managed to compute a variant ID for an item, but this variant has no associated model, the fallback model will be used instead.
Modules with fallback are much less likely to hand over control to lower priority modules.

The fallback model is bound to the variant ID `variants-cit:fallback`. It can also be assigned using `modelList`.

### Field: `special`
**Optional**, maps Strings to Identifiers.

A list of models that the some module types may use in exceptional circumstances.
What models are used and when depends on the [`type`](Module-Types) of the CIT module.

Very few module types actually make use of this option; you can safely ignore it.
All special models are always optional.

Special models are bound to variant IDs starting with `variants-cit:special/`. They can instead be assigned using `modelList`


## Asset-generation

The fields `assetGen` and `modelParent` are differently formated aliases for the same setting. You can only use one at the same time.

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

Old-school alternative to the `assetGen` option; this does nothing if `assetGen` is already defined.
This is equivalent to the following, where `<modelParent>` is the value of this option:
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
